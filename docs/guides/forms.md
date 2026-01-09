# Form Building UX Agent Specification (v1)

## Table of Contents

1. [Core Principles](#1-core-principles) (Line 9)
2. [Form Patterns](#2-form-patterns) (Line 35)
   - 2.1 [React Hook Form + SWR Pattern](#21-react-hook-form--swr-pattern) (Line 37)
   - 2.2 [Optimistic Updates](#22-optimistic-updates) (Line 95)
   - 2.3 [Auto-Save with Debouncing](#23-auto-save-with-debouncing) (Line 120)
   - 2.4 [Error Summary](#24-error-summary) (Line 145)
   - 2.5 [Draft Persistence](#25-draft-persistence) (Line 180)
   - 2.6 [Multi-Step Forms](#26-multi-step-forms) (Line 205)
   - 2.7 [Modal Forms](#27-modal-forms) (Line 230)
   - 2.8 [Real-Time Updates](#28-real-time-updates) (Line 255)
3. [Complete Form Template](#3-complete-form-template) (Line 280)
4. [Best Practices Checklist](#4-best-practices-checklist) (Line 350)
5. [Common Patterns from Codebase](#5-common-patterns-from-codebase) (Line 380)
6. [Accessibility & Performance](#6-accessibility--performance) (Line 400)
7. [Version History](#7-version-history) (Line 450)

---

## Overview

The Form Building UX Agent implements forms following Flexbike's best practices: React Hook Form + SWR, optimistic updates, auto-save, validation, and error summaries. All patterns are consolidated from the booking modal, product editor, and other form components.

---

## 1. CORE PRINCIPLES

### 1.1 User Experience First

- Forms should feel instant and responsive
- Users should never lose their work
- Clear feedback on all actions (saving, errors, success)
- Progressive enhancement: works without JavaScript, enhanced with it

### 1.2 Optimistic Updates

- Update UI immediately before server confirmation
- Revert gracefully on failure
- Show loading states during async operations
- Never block user input during saves

### 1.3 Data Persistence

- Auto-save with debouncing (1 second default)
- Draft persistence for multi-step forms (localStorage)
- Restore drafts on page reload
- Clear drafts after successful submission

### 1.4 Validation & Error Handling

- Client-side validation for immediate feedback
- Server-side validation as final authority
- Clear, actionable error messages
- Field-level and form-level error display
- Error summary near submit buttons
- Prevent invalid submissions

---

## 2. FORM PATTERNS

### 2.1 React Hook Form + SWR Pattern

**Always Use React Hook Form with SWR:**

- React Hook Form for form state management (uncontrolled components, minimal re-renders)
- SWR with custom hooks for data fetching (never embed `fetch` directly)
- Zod schemas for type-safe validation
- Optimistic updates via SWR `mutate`

**Installation:**

```bash
pnpm add react-hook-form @hookform/resolvers zod swr
```

**Custom Hook Pattern:**

```typescript
// hooks/useProduct.ts
import useSWR, { mutate } from "swr";
import { useRouter } from "next/navigation";

const fetcher = async (url: string) => {
  const response = await fetch(url, { credentials: "include" });
  if (!response.ok) {
    if (response.status === 401) throw new Error("UNAUTHORIZED");
    if (response.status === 403) throw new Error("ACCESS_DENIED");
    if (response.status === 404) throw new Error("NOT_FOUND");
    const errorData = await response.json().catch(() => ({}));
    throw new Error(errorData.error || "Failed to fetch");
  }
  const data = await response.json();
  return data.product;
};

export function useProduct(productId: string | null) {
  const router = useRouter();
  const {
    data,
    error,
    isLoading,
    mutate: revalidate,
  } = useSWR(productId ? `/api/admin/products/${productId}` : null, fetcher, {
    revalidateOnFocus: true,
    revalidateOnReconnect: true,
    dedupingInterval: 5000,
    onError: (err) => {
      if (err.message === "UNAUTHORIZED") {
        router.push(`/admin/login?redirect=${window.location.pathname}`);
      }
    },
  });

  const updateProduct = async (updates: Partial<Product>) => {
    if (!productId) return;
    // Optimistic update
    await mutate(
      `/api/admin/products/${productId}`,
      { ...data, ...updates },
      false
    );
    try {
      const response = await fetch(`/api/admin/products/${productId}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        credentials: "include",
        body: JSON.stringify(updates),
      });
      if (!response.ok) {
        await revalidate(); // Revert on error
        throw new Error("Failed to update");
      }
      await revalidate(); // Revalidate with server data
    } catch (err) {
      await revalidate(); // Revert on error
      throw err;
    }
  };

  return {
    product: data ?? null,
    loading: isLoading,
    error: error?.message || null,
    refetch: () => revalidate(),
    updateProduct,
  };
}
```

**Form Component with React Hook Form:**

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { useProduct } from "@/hooks/useProduct";
import { debounce } from "lodash";

const schema = z.object({
  title: z.string().min(1, "Title is required"),
  pricePerDay: z.number().min(0, "Price must be positive"),
});

type FormData = z.infer<typeof schema>;

export default function ProductForm({ productId }: Props) {
  const { product, loading, error, updateProduct } = useProduct(productId);
  const {
    register,
    handleSubmit,
    formState: { errors, isDirty, isSubmitting },
    watch,
    reset,
    setFocus,
  } = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: product || { title: "", pricePerDay: 0 },
    mode: "onBlur",
  });

  // Reset form when data loads
  useEffect(() => {
    if (product) reset(product);
  }, [product, reset]);

  // Auto-save with debounce
  const watchedValues = watch();
  const debouncedSave = useMemo(
    () =>
      debounce(async (data: FormData) => {
        if (!isDirty || !productId) return;
        try {
          await mutate(
            `/api/admin/products/${productId}`,
            { ...product, ...data },
            false
          );
          await updateProduct(data);
        } catch (err) {
          console.error("Auto-save failed:", err);
        }
      }, 1000),
    [productId, isDirty, product, updateProduct]
  );

  useEffect(() => {
    if (isDirty && productId) debouncedSave(watchedValues);
  }, [watchedValues, isDirty, productId, debouncedSave]);

  const onSubmit = async (data: FormData) => {
    try {
      await updateProduct(data);
      reset(data, { keepDirty: false });
      toast("Saved successfully", { variant: "success" });
    } catch (err) {
      toast(err instanceof Error ? err.message : "Failed to save", {
        variant: "error",
      });
    }
  };

  // Error summary (see section 2.4)
  const allErrors = useMemo(() => {
    const errorList: Array<{ field: string; message: string }> = [];
    Object.entries(errors).forEach(([field, error]) => {
      if (error?.message) errorList.push({ field, message: error.message });
    });
    if (error) errorList.push({ field: "server", message: error });
    return errorList;
  }, [errors, error]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!product) return <NotFound />;

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Error summary (section 2.4) */}
      {/* Form fields */}
      <input {...register("title")} />
      {errors.title && <p className="text-red-600">{errors.title.message}</p>}
      {/* Save status & submit button */}
    </form>
  );
}
```

**Key RHF Best Practices:**

- Use `mode: "onBlur"` for better UX
- Use `reset(updated, { keepDirty: false })` after successful save
- Use `setFocus()` to programmatically focus error fields
- Use `isDirty` to track unsaved changes
- Use `isSubmitting` for loading states

---

### 2.2 Optimistic Updates

**Global Pending Updates Store:**

```typescript
declare global {
  interface Window {
    __csPendingUpdate?: Record<string, any>;
  }
}

const upsertPending = useCallback(
  (patch: Record<string, any>) => {
    if (!window.__csPendingUpdate) window.__csPendingUpdate = {};
    const prev = window.__csPendingUpdate[entityId] || {};
    window.__csPendingUpdate[entityId] = { ...prev, ...patch };
    window.dispatchEvent(new Event("cs-pending-update"));
  },
  [entityId]
);
```

**With SWR:**

```typescript
// Optimistic update before save
await mutate(`/api/entity/${id}`, { ...data, ...updates }, false);

try {
  const response = await fetch(`/api/entity/${id}`, {
    method: "PATCH",
    body: JSON.stringify(updates),
  });
  if (!response.ok) {
    await mutate(`/api/entity/${id}`); // Revert on error
    throw new Error("Save failed");
  }
  await mutate(`/api/entity/${id}`); // Revalidate with server data
} catch (err) {
  await mutate(`/api/entity/${id}`); // Revert on error
  throw err;
}
```

**Events:**

- `cs-pending-update`: Notifies components of pending changes
- `cs-saving-start`: Save operation started
- `cs-saving-complete`: Save succeeded
- `cs-saving-error`: Save failed
- `cs-booking-updated`: Entity updated (refresh data)

---

### 2.3 Auto-Save with Debouncing

**Implementation:**

```typescript
const debouncedSave = useMemo(
  () =>
    debounce(async (data: FormData) => {
      if (!isDirty || !entityId) return;
      try {
        await mutate(`/api/entity/${entityId}`, { ...data }, false);
        const response = await fetch(`/api/entity/${entityId}`, {
          method: "PATCH",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(data),
        });
        if (!response.ok) {
          await mutate(`/api/entity/${entityId}`);
          throw new Error("Save failed");
        }
        const updated = await response.json();
        await mutate(`/api/entity/${entityId}`, updated, false);
        reset(updated, { keepDirty: false });
      } catch (err) {
        await mutate(`/api/entity/${entityId}`);
        console.error("Auto-save failed:", err);
      }
    }, 1000),
  [entityId, isDirty, mutate, reset]
);

useEffect(() => {
  if (isDirty && entityId) debouncedSave(watchedValues);
}, [watchedValues, isDirty, entityId, debouncedSave]);
```

**Save Status Indicators:**

- `"saved"`: Green indicator (✓ Saved)
- `"saving"`: Yellow spinner (Saving...)
- `"unsaved"`: Orange dot (Unsaved changes)
- `"error"`: Red indicator (auto-reset after 3s)

**Prevent Concurrent Saves:**

```typescript
const activeSavesRef = useRef<Set<string>>(new Set());
if (activeSavesRef.current.has(field)) return; // Skip if already saving
activeSavesRef.current.add(field);
// ... perform save ...
activeSavesRef.current.delete(field);
```

---

### 2.4 Error Summary

**Display all errors in a summary above submit buttons:**

```typescript
const allErrors = useMemo(() => {
  const errorList: Array<{ field: string; message: string }> = [];
  Object.entries(formState.errors).forEach(([field, error]) => {
    if (error?.message) errorList.push({ field, message: error.message });
  });
  if (serverError) errorList.push({ field: "server", message: serverError });
  return errorList;
}, [formState.errors, serverError]);

{
  allErrors.length > 0 && (
    <div
      className="mb-4 p-4 bg-red-50 border-l-4 border-red-500 rounded-md"
      role="alert"
    >
      <h3 className="text-sm font-semibold text-red-800 mb-2">
        Please fix the following {allErrors.length} error
        {allErrors.length > 1 ? "s" : ""}:
      </h3>
      <ul className="list-disc list-inside space-y-1">
        {allErrors.map((err, index) => (
          <li
            key={index}
            className="text-sm text-red-700 cursor-pointer"
            onClick={() => {
              if (err.field !== "server") setFocus(err.field as keyof FormData);
            }}
          >
            {err.field !== "server" && (
              <span className="font-medium capitalize">{err.field}: </span>
            )}
            {err.message}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Features:**

- Show error count in header
- Clickable errors that focus the field
- Disable submit button when errors exist
- Use `role="alert"` and `aria-live="polite"` for accessibility

---

### 2.5 Draft Persistence

**Multi-Step Forms:**

```typescript
type Draft = { bikeId: string; startDate?: string; endDate?: string /* ... */ };

function getDraftKey(bikeId: string) {
  return `bookingDraft:${bikeId}`;
}

// Restore on mount
useEffect(() => {
  try {
    const raw = localStorage.getItem(getDraftKey(bikeId));
    if (raw) {
      const parsed = JSON.parse(raw) as Draft;
      // Restore form state from parsed
    }
  } catch (_) {
    setStorageAvailable(false);
  }
}, [bikeId]);

// Save after each step
const saveDraft = useCallback(
  (update: Partial<Draft>) => {
    try {
      const existing = JSON.parse(
        localStorage.getItem(getDraftKey(bikeId)) || "{}"
      );
      const next = { ...existing, ...update };
      localStorage.setItem(getDraftKey(bikeId), JSON.stringify(next));
    } catch (_) {
      setStorageAvailable(false);
    }
  },
  [bikeId]
);

// Clear on successful submit
const onSubmit = async (data: FormData) => {
  await saveToServer(data);
  localStorage.removeItem(getDraftKey(bikeId));
};
```

**With React Hook Form:**

```typescript
const form = useForm({ defaultValues: loadDraft() || initialValues });
useEffect(() => {
  const subscription = watch((value) => saveDraft(value));
  return () => subscription.unsubscribe();
}, [watch]);
```

---

### 2.6 Multi-Step Forms

**Step Management:**

```typescript
const [step, setStep] = useState<1 | 2 | 3 | 4>(1);

const handleStep = () => {
  if (!validateStep()) {
    setError("Please complete required fields");
    return;
  }
  setError(null);
  saveDraft(getCurrentStepData(), `step-${step}`);
  setStep((prev) => (prev + 1) as any);
};

// Progress indicator
<div>
  <h2>Form Title</h2>
  <p className="text-sm text-gray-600">Step {step} of 4</p>
</div>

// Navigation
<div className="flex justify-between">
  {step > 1 && <Button onClick={() => setStep((prev) => (prev - 1) as any)}>Back</Button>}
  <Button onClick={handleStep}>Continue</Button>
</div>
```

---

### 2.7 Modal Forms

**Modal Structure:**

```typescript
<div className="fixed inset-0 z-[100] flex items-center justify-center bg-black bg-opacity-50">
  <div className="relative w-full max-w-7xl max-h-[90vh] bg-white rounded-lg shadow-xl flex flex-col">
    {/* Sticky header */}
    <div className="flex items-center justify-between p-4 border-b bg-gray-50 sticky top-0 z-10">
      <h2>Form Title</h2>
      <button onClick={onClose}>×</button>
    </div>
    {/* Scrollable content */}
    <div className="flex-1 overflow-y-auto min-h-0">{/* Form */}</div>
  </div>
</div>
```

**Backdrop Handling (prevent close on drag):**

```typescript
const [backdropMouseDownTarget, setBackdropMouseDownTarget] =
  useState<EventTarget | null>(null);

const handleBackdropMouseDown = (e: React.MouseEvent) => {
  if (e.target === e.currentTarget) setBackdropMouseDownTarget(e.target);
  else setBackdropMouseDownTarget(null);
};

const handleBackdropMouseUp = (e: React.MouseEvent) => {
  if (
    e.target === e.currentTarget &&
    backdropMouseDownTarget === e.currentTarget
  ) {
    onClose();
  }
  setBackdropMouseDownTarget(null);
};
```

**Keyboard Handling:**

```typescript
useEffect(() => {
  const handleEscape = (e: KeyboardEvent) => {
    if (e.key === "Escape") onClose();
  };
  document.addEventListener("keydown", handleEscape);
  return () => document.removeEventListener("keydown", handleEscape);
}, [onClose]);

// Prevent body scroll
useEffect(() => {
  if (isOpen) {
    document.body.style.overflow = "hidden";
    return () => {
      document.body.style.overflow = "unset";
    };
  }
}, [isOpen]);
```

---

### 2.8 Real-Time Updates

**WebSocket Integration:**

```typescript
useEffect(() => {
  const sb = getSupabaseBrowser();
  if (!sb) return;
  const channel = sb
    .channel(`bookings:${booking.id}`)
    .on("broadcast", { event: "note" }, (payload) => {
      if (payload?.payload?.actor === "customer") {
        setCustomerMessage(payload.payload.note);
        setIsMessageModalOpen(true);
      }
    })
    .subscribe();
  return () => sb.removeChannel(channel);
}, [booking.id]);
```

**Connection Status:**

```typescript
const [connectionStatus, setConnectionStatus] = useState<
  "connecting" | "connected" | "disconnected"
>("connecting");

<div className="flex items-center gap-1.5">
  <div
    className={`w-2 h-2 rounded-full ${
      connectionStatus === "connected"
        ? "bg-green-500"
        : connectionStatus === "connecting"
        ? "bg-amber-500 animate-pulse"
        : "bg-red-500"
    }`}
  />
  <span className="text-xs capitalize">
    {connectionStatus === "connected" ? "Live" : connectionStatus}
  </span>
</div>;
```

---

## 3. COMPLETE FORM TEMPLATE

**Full implementation with all patterns:**

```typescript
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { useProduct } from "@/hooks/useProduct";
import { debounce } from "lodash";
import { AlertCircle } from "lucide-react";

const schema = z.object({
  title: z.string().min(1, "Title is required"),
  pricePerDay: z.number().min(0, "Price must be positive"),
});

type FormData = z.infer<typeof schema>;

export default function ProductForm({ productId }: Props) {
  const {
    product,
    loading,
    error: fetchError,
    updateProduct,
  } = useProduct(productId);
  const {
    register,
    handleSubmit,
    formState: { errors, isDirty, isSubmitting },
    watch,
    reset,
    setFocus,
  } = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: product || { title: "", pricePerDay: 0 },
    mode: "onBlur",
  });

  useEffect(() => {
    if (product) reset(product);
  }, [product, reset]);

  // Auto-save
  const watchedValues = watch();
  const debouncedSave = useMemo(
    () =>
      debounce(async (data: FormData) => {
        if (!isDirty || !productId) return;
        try {
          await mutate(
            `/api/admin/products/${productId}`,
            { ...product, ...data },
            false
          );
          await updateProduct(data);
        } catch (err) {
          console.error("Auto-save failed:", err);
        }
      }, 1000),
    [productId, isDirty, product, updateProduct]
  );

  useEffect(() => {
    if (isDirty && productId) debouncedSave(watchedValues);
  }, [watchedValues, isDirty, productId, debouncedSave]);

  const onSubmit = async (data: FormData) => {
    try {
      await updateProduct(data);
      reset(data, { keepDirty: false });
      toast("Saved successfully", { variant: "success" });
    } catch (err) {
      toast(err instanceof Error ? err.message : "Failed to save", {
        variant: "error",
      });
    }
  };

  // Error summary
  const allErrors = useMemo(() => {
    const errorList: Array<{ field: string; message: string }> = [];
    Object.entries(errors).forEach(([field, error]) => {
      if (error?.message) errorList.push({ field, message: error.message });
    });
    if (fetchError) errorList.push({ field: "server", message: fetchError });
    return errorList;
  }, [errors, fetchError]);

  if (loading) return <LoadingSpinner />;
  if (fetchError) return <ErrorMessage error={fetchError} />;
  if (!product) return <NotFound />;

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Error summary */}
      {allErrors.length > 0 && (
        <div
          className="mb-4 p-4 bg-red-50 border-l-4 border-red-500 rounded-md"
          role="alert"
        >
          <div className="flex items-start">
            <AlertCircle className="w-5 h-5 text-red-600 mt-0.5 mr-3 flex-shrink-0" />
            <div className="flex-1">
              <h3 className="text-sm font-semibold text-red-800 mb-2">
                Please fix the following {allErrors.length} error
                {allErrors.length > 1 ? "s" : ""}:
              </h3>
              <ul className="list-disc list-inside space-y-1">
                {allErrors.map((err, index) => (
                  <li
                    key={index}
                    className="text-sm text-red-700 cursor-pointer"
                    onClick={() => {
                      if (err.field !== "server")
                        setFocus(err.field as keyof FormData);
                    }}
                  >
                    {err.field !== "server" && (
                      <span className="font-medium capitalize">
                        {err.field}:{" "}
                      </span>
                    )}
                    {err.message}
                  </li>
                ))}
              </ul>
            </div>
          </div>
        </div>
      )}

      {/* Form fields */}
      <div>
        <label htmlFor="title">Title</label>
        <input
          id="title"
          {...register("title")}
          className={errors.title ? "border-red-500" : ""}
        />
        {errors.title && (
          <p className="text-sm text-red-600 mt-1">{errors.title.message}</p>
        )}
      </div>

      {/* Save status */}
      {isDirty && (
        <div className="text-sm text-orange-600">Unsaved changes</div>
      )}
      {isSubmitting && <div className="text-sm text-blue-600">Saving...</div>}

      {/* Submit button */}
      <div className="flex justify-end gap-3 mt-6">
        <Button type="submit" disabled={isSubmitting || allErrors.length > 0}>
          {isSubmitting ? "Saving..." : "Save"}
        </Button>
      </div>
    </form>
  );
}
```

---

## 4. BEST PRACTICES CHECKLIST

### Before Building

- [ ] Use React Hook Form + SWR (never embed `fetch` directly)
- [ ] Create custom hook for data fetching
- [ ] Define Zod schema for validation
- [ ] Plan auto-save strategy (1 second debounce)
- [ ] Plan draft persistence (for multi-step forms)
- [ ] Plan error summary display

### During Implementation

- [ ] Implement optimistic updates with SWR `mutate`
- [ ] Add debounced auto-save (1 second default)
- [ ] Track save status (saved/saving/error)
- [ ] Add error summary near submit buttons
- [ ] Use `mode: "onBlur"` for validation
- [ ] Use `reset(updated, { keepDirty: false })` after save
- [ ] Prevent concurrent saves
- [ ] Handle localStorage unavailability

### After Implementation

- [ ] Test with slow network (throttle in DevTools)
- [ ] Test error scenarios (network failure, validation errors)
- [ ] Test draft restoration
- [ ] Verify optimistic updates revert on failure
- [ ] Check accessibility (keyboard navigation, screen readers)
- [ ] Test on mobile devices

---

## 5. COMMON PATTERNS FROM CODEBASE

### Booking Modal (EnhancedBookingFlow)

- Multi-step form with progress indicator
- Draft persistence in localStorage
- Step-by-step validation
- Quick preset options (today, tomorrow, week, month)

### Product Editor (AdminProductDetailPage)

- Auto-save with 1 second debounce
- Save status indicators (saved/saving/error)
- Change detection via JSON comparison
- Drag-and-drop file uploads

### Booking Confirmation (BookingConfirmationClient)

- Optimistic updates via `window.__csPendingUpdate`
- Field-level auto-save with debounce
- Real-time WebSocket updates
- Change tracking and visual indicators

### Booking Preview Modal (BookingPreviewModal)

- Modal with backdrop click handling
- Escape key to close
- Body scroll prevention
- Connection status indicator

---

## 6. ACCESSIBILITY & PERFORMANCE

### Accessibility

- **Keyboard Navigation**: Tab order follows visual order, Enter submits, Escape closes modals
- **Screen Readers**: Label all fields, associate errors with fields using `aria-describedby`, use `role="alert"` for error summary
- **Focus Management**: Focus first field on mount, focus error field on validation failure, return focus after modal close

### Performance

- **Debouncing**: Use 1 second debounce for auto-save, clear timeouts on unmount
- **Memoization**: Memoize expensive computations, use `useMemo` for derived state, use `useCallback` for event handlers
- **State Updates**: Batch state updates when possible, avoid unnecessary re-renders, use refs for values that don't need to trigger renders

---

## 7. VERSION HISTORY

- **v1** (2025-01-XX): Initial form building UX agent specification
  - Consolidated patterns from booking modal and product editor
  - Always use React Hook Form + SWR with custom hooks
  - Optimistic updates pattern
  - Auto-save with debouncing
  - Error summary near submit buttons
  - Draft persistence
  - All patterns condensed to ~800 lines

---

## END OF SPECIFICATION
