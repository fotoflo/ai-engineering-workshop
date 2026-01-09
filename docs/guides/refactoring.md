# Refactoring Agent Specification (v1)

## Overview

The Refactoring Agent handles safe, systematic code refactoring tasks for the Flexbike codebase. It extracts functions, breaks down large files into smaller modules, improves code readability, eliminates duplication, and modernizes legacy code while maintaining functionality and following project conventions.

---

## 1. Capabilities

The Refactoring Agent must:

1. Analyze code for refactoring opportunities (duplication, complexity, readability)
2. Extract reusable functions and components
3. Break down large files into smaller, focused modules
4. Move files to their correct locations based on Flexbike conventions
5. Update all imports and references when moving or splitting files
6. Apply clear variable naming conventions
7. Maintain TypeScript types and interfaces
8. Preserve existing functionality while improving structure
9. Generate tests for refactored code
10. Follow Flexbike coding conventions and patterns
11. Provide before/after comparison of changes
12. Handle rollback options for failed refactoring

---

## 2. Interaction Flow

1. **Timestamp start**
2. **Analyze target code** - Read files and identify refactoring opportunities
3. **Present refactoring options**:
   - **A** – Extract function from duplicated code
   - **B** – Extract reusable component
   - **C** – Break down large file into smaller modules
   - **D** – Move files to correct locations
   - **E** – Rename variables/functions for clarity
   - **F** – Simplify complex conditional logic
   - **G** – Convert to TypeScript (if applicable)
   - **H** – Optimize performance (memoization, etc.)
   - **I** – Manual refactoring task
   - **J** – Cancel
4. **Show proposed changes** with before/after preview
   - For file moves: Show current location → proposed location
   - For file splits: Show original file → proposed new files with their purposes
   - List all files that will be affected by import updates
   - Show file size reduction (lines of code before/after)
5. **Ask for approval**: _Y / N / Edit_
6. **If approved**:
   - Apply refactoring changes
   - Break down large files into smaller modules (if selected)
   - Move files to correct locations (if selected)
   - Update all imports and references across the codebase
   - Run tests to verify functionality preserved
7. **Generate test coverage** for refactored code
8. **Offer commit options** (delegate to Git agent)
9. **Timestamp end**

---

## 3. Guardrails

- Never refactor without preserving functionality
- Never remove code without understanding its purpose
- Never commit breaking changes
- Always run tests after refactoring
- Always provide rollback options
- Never refactor production-critical code without approval
- Follow Flexbike naming conventions (clear, descriptive names)
- Maintain existing TypeScript types and interfaces
- Preserve React component APIs and props
- Never auto-generate complex logic without human review
- **When moving files**: Always update all imports and references before completing the move
- **When moving files**: Verify file location matches Flexbike directory topology rules
- **When moving files**: Check for circular dependencies before moving
- **When moving files**: Ensure moved files follow correct naming conventions for their new location
- **When breaking down files**: Identify logical boundaries (components, hooks, utilities, types)
- **When breaking down files**: Maintain single responsibility principle for each new file
- **When breaking down files**: Keep related code together (don't split unnecessarily)
- **When breaking down files**: Update all imports in files that reference the original
- **When breaking down files**: Ensure each new file has a clear, descriptive purpose
- **When breaking down files**: Consider file size targets (components: <500 lines, utilities: <300 lines, hooks: <200 lines)

---

## 4. File Naming and Topology

### Current Flexbike Codebase Structure

#### Directory Topology

```
src/
├── app/                          # Next.js App Router
│   ├── [...slug]/               # Dynamic catch-all routes
│   ├── admin/                   # Admin interface (65 files)
│   │   ├── calendar/            # Calendar views
│   │   ├── companies/           # Company management
│   │   ├── components/          # Admin-specific components
│   │   ├── firebase-sync/       # Data synchronization UI
│   │   └── hooks/               # Admin-specific hooks
│   ├── api/                     # API routes (132 files)
│   │   ├── admin/               # Admin-only endpoints
│   │   ├── auth/                # Authentication
│   │   ├── bookings/            # Booking operations
│   │   └── companies/           # Company data
│   ├── components/              # Reusable UI components (87 files)
│   ├── hooks/                   # React hooks (31 files)
│   ├── utils/                   # Page-specific utilities
│   └── [feature]/               # Feature-specific pages
├── lib/                         # Shared business logic
│   ├── transforms/              # Data transformation logic (9 files)
│   ├── data/                    # Static data and constants
│   └── [utility].ts             # Utility modules
├── server/                      # Server-side utilities
│   ├── db/                      # Database operations
│   ├── services/                # Business services
│   └── geo/                     # Geolocation services
├── utils/                       # Global utilities
└── __tests__/                   # Test suites (mirrors src/)
```

#### Naming Conventions

**Components (`src/app/components/`)**

- **File naming**: `PascalCase.tsx` (e.g., `BikeCard.tsx`, `CompanyHeader.tsx`)
- **Component naming**: PascalCase matching filename
- **Client components**: Add `Client` suffix (e.g., `BookingContactClient.tsx`)
- **Special patterns**:
  - Form components: `BookingCreateForm.tsx`
  - Modal components: `BookingPreviewModal.tsx`
  - Page components: `BikeDetailPage.tsx` (in `pages/` subfolder)

**Hooks (`src/app/hooks/`)**

- **File naming**: `use*.ts` or `get*.ts`
- **Hook naming**: camelCase with `use` prefix (e.g., `useCompanyBikes.ts`)
- **Data fetching**: `useGet*ById.ts` (e.g., `useGetBikeById.ts`)
- **State management**: `useBookingState.ts`, `useBookingDraft.ts`
- **Utility hooks**: `useLocationSuggestions.ts`, `useGoogleMaps.ts`

**API Routes (`src/app/api/`)**

- **File naming**: `route.ts` (standard Next.js API routes)
- **Directory structure**: RESTful with nested folders
- **Examples**:
  - `/api/bookings/route.ts` - Main bookings endpoint
  - `/api/admin/companies/[companyId]/route.ts` - Company-specific admin endpoint
  - `/api/companies/[id]/bookings/route.ts` - Company bookings

**Utilities and Libraries (`src/lib/`, `src/utils/`)**

- **File naming**: `kebab-case.ts` or `camelCase.ts`
- **Examples**:
  - `booking-utils.ts`
  - `dataTransformers.ts`
  - `stripe-utils.ts`
- **Subfolders**: Logical grouping (e.g., `transforms/`, `data/`)

**Tests (`src/__tests__/`)**

- **Mirrors source structure**: `hooks/`, `components/`, `utils/`
- **File naming**: `[originalName].test.js` or `[originalName].test.ts`
- **Integration tests**: `[feature].integration.test.ts`
- **E2E tests**: `e2e/` folder

### Import Patterns

**Relative Imports**

- Same directory: `./ComponentName`
- Parent directory: `../utils/helpers`
- Sibling directories: `../../components/BikeCard`

**Absolute Imports (from project root)**

- Components: `src/app/components/BikeCard`
- Hooks: `src/app/hooks/useCompanyBikes`
- Utils: `src/lib/booking-utils`
- Types: `src/lib/types` (when applicable)

**Barrel Exports**

- `src/app/hooks/index.ts` - Exports all hooks
- `src/lib/transforms/index.ts` - Exports all transformers

### Refactoring Topology Rules

**When extracting components:**

1. Place in `src/app/components/` if reusable across pages
2. Place in `src/app/[feature]/components/` if feature-specific
3. Place in `src/app/admin/components/` for admin-specific components
4. Use `Client` suffix for components using browser APIs

**When extracting hooks:**

1. Place in `src/app/hooks/` for general hooks
2. Place in `src/app/admin/hooks/` for admin-specific hooks
3. Place in `src/hooks/` for global hooks (rare)

**When extracting utilities:**

1. Place in `src/lib/` for business logic utilities
2. Place in `src/utils/` for generic utilities
3. Place in `src/app/utils/` for page-specific utilities
4. Place in `src/server/` for server-only utilities

**When creating new features:**

1. Create `src/app/[feature]/` directory
2. Add `page.tsx` for the main page
3. Add `components/` subfolder for feature components
4. Add `hooks/` subfolder for feature hooks
5. Add API routes in `src/app/api/[feature]/`

### File Movement Rules

**When moving files to correct locations:**

1. **Identify correct location** based on file type and usage:

   - Components → `src/app/components/` (reusable) or `src/app/[feature]/components/` (feature-specific)
   - Hooks → `src/app/hooks/` (general) or `src/app/admin/hooks/` (admin-specific)
   - Utilities → `src/lib/` (business logic) or `src/utils/` (generic)
   - API routes → `src/app/api/[category]/`
   - Server utilities → `src/server/`

2. **Check file naming** matches conventions for target location:

   - Components: PascalCase (e.g., `BikeCard.tsx`)
   - Hooks: camelCase with `use` prefix (e.g., `useCompanyBikes.ts`)
   - Utilities: kebab-case or camelCase (e.g., `booking-utils.ts`)

3. **Find all imports** that reference the file:

   - Search codebase for import statements
   - Check for relative imports (`./`, `../`)
   - Check for absolute imports (`src/app/components/...`)
   - Check for barrel exports (`index.ts` files)

4. **Update imports systematically**:

   - Update relative imports to reflect new path
   - Update absolute imports if path changed
   - Update barrel exports if file moved to/from barrel export directory
   - Verify no circular dependencies introduced

5. **Verify file location** matches usage pattern:
   - If used across multiple features → move to shared location
   - If used only in one feature → move to feature-specific location
   - If admin-only → move to `admin/` subdirectory

**Common file movement scenarios:**

- **Component in wrong location**: Move from `src/app/[feature]/` to `src/app/components/` if reusable
- **Hook in component file**: Extract hook to `src/app/hooks/` or `src/app/admin/hooks/`
- **Utility in wrong directory**: Move from `src/utils/` to `src/lib/` if business logic, or vice versa
- **API route misorganized**: Move to correct category folder in `src/app/api/`
- **Server code in client directory**: Move to `src/server/` if server-only

---

## 5. Project Structure Context (Flexbike-Specific)

### Refactoring Targets

1. **Admin Components** (`src/app/admin/components/`)

   - Extract shared table logic
   - Componentize repeated UI patterns
   - Simplify complex booking management logic

2. **API Routes** (`src/app/api/`)

   - Extract common validation logic
   - Standardize error handling patterns
   - Simplify complex business logic

3. **Data Transformers** (`src/lib/transforms/`)

   - Extract shared transformation utilities
   - Simplify nested object mappings
   - Standardize data validation

4. **React Hooks** (`src/app/hooks/`)

   - Extract common hook logic
   - Simplify complex state management
   - Standardize error handling

5. **Utility Functions** (`src/utils/`)
   - Consolidate duplicate utility functions
   - Extract reusable calculation logic
   - Improve function naming and documentation

### Code Quality Standards

- **Variable naming**: Clear, descriptive names (not `data`, `result`, `temp`)
- **Function length**: Max 50 lines, extract if longer
- **Component complexity**: Max 3 levels of nesting
- **Import organization**: Group by external/internal, sort alphabetically
- **Error handling**: Consistent patterns across similar functions
- **Type safety**: Full TypeScript coverage for new/refactored code

---

## 5. Refactoring Patterns

### Function Extraction

**Pattern**: Extract duplicated logic into reusable functions

**Before:**

```typescript
// In multiple components
const handleBookingSubmit = async (data) => {
  const validated = validateBookingData(data);
  if (!validated.isValid) {
    setErrors(validated.errors);
    return;
  }
  const result = await submitBooking(validated.data);
  if (result.success) {
    navigate("/confirmation");
  }
};
```

**After:**

```typescript
// Extracted to utils/bookingUtils.ts
export const processBookingSubmission = async (
  data: BookingData,
  onSuccess: () => void,
  onError: (errors: ValidationError[]) => void
) => {
  const validated = validateBookingData(data);
  if (!validated.isValid) {
    onError(validated.errors);
    return;
  }
  const result = await submitBooking(validated.data);
  if (result.success) {
    onSuccess();
  }
};

// In components
const handleBookingSubmit = async (data) => {
  await processBookingSubmission(
    data,
    () => navigate("/confirmation"),
    setErrors
  );
};
```

### Component Extraction

**Pattern**: Extract repeated UI patterns into reusable components

**Before:**

```tsx
// Repeated in multiple admin tables
<div className="flex items-center space-x-2">
  <button
    onClick={() => onEdit(item)}
    className="text-blue-600 hover:text-blue-800"
  >
    <PencilIcon className="h-4 w-4" />
  </button>
  <button
    onClick={() => onDelete(item)}
    className="text-red-600 hover:text-red-800"
  >
    <TrashIcon className="h-4 w-4" />
  </button>
</div>
```

**After:**

```tsx
// New component: components/TableActions.tsx
interface TableActionsProps {
  item: any;
  onEdit: (item: any) => void;
  onDelete: (item: any) => void;
}

export const TableActions = ({ item, onEdit, onDelete }: TableActionsProps) => (
  <div className="flex items-center space-x-2">
    <button
      onClick={() => onEdit(item)}
      className="text-blue-600 hover:text-blue-800"
      aria-label="Edit item"
    >
      <PencilIcon className="h-4 w-4" />
    </button>
    <button
      onClick={() => onDelete(item)}
      className="text-red-600 hover:text-red-800"
      aria-label="Delete item"
    >
      <TrashIcon className="h-4 w-4" />
    </button>
  </div>
);

// Usage in tables
<TableActions item={item} onEdit={onEdit} onDelete={onDelete} />;
```

### Variable Naming Improvements

**Pattern**: Replace unclear names with descriptive ones

**Before:**

```typescript
const d = getCompanyData();
const r = processResults(d);
const f = formatOutput(r);
```

**After:**

```typescript
const companyData = getCompanyData();
const processedResults = processResults(companyData);
const formattedOutput = formatOutput(processedResults);
```

### Breaking Down Large Files

**Pattern**: Split large files into smaller, focused modules based on logical boundaries

**Before:**

```typescript
// LargeComponent.tsx (800+ lines)
// Contains: Main component, sub-components, hooks, utilities, types, constants

import { useState, useEffect } from 'react';
import { formatDate, calculatePrice } from './utils';

// Types (50 lines)
interface Booking { ... }
interface Company { ... }
interface User { ... }

// Constants (30 lines)
const STATUSES = { ... };
const PRICING_RULES = { ... };

// Utility functions (100 lines)
function formatBookingData() { ... }
function validateBooking() { ... }
function calculateTotal() { ... }

// Custom hooks (150 lines)
function useBookingState() { ... }
function useCompanyData() { ... }

// Sub-components (200 lines)
function BookingForm() { ... }
function BookingSummary() { ... }
function BookingActions() { ... }

// Main component (270 lines)
export function LargeComponent() {
  // Complex component logic
}
```

**After:**

```typescript
// types.ts (50 lines)
export interface Booking { ... }
export interface Company { ... }
export interface User { ... }

// constants.ts (30 lines)
export const STATUSES = { ... };
export const PRICING_RULES = { ... };

// bookingUtils.ts (100 lines)
export function formatBookingData() { ... }
export function validateBooking() { ... }
export function calculateTotal() { ... }

// hooks/useBookingState.ts (75 lines)
export function useBookingState() { ... }

// hooks/useCompanyData.ts (75 lines)
export function useCompanyData() { ... }

// components/BookingForm.tsx (70 lines)
export function BookingForm() { ... }

// components/BookingSummary.tsx (65 lines)
export function BookingSummary() { ... }

// components/BookingActions.tsx (65 lines)
export function BookingActions() { ... }

// LargeComponent.tsx (150 lines - main component only)
import { BookingForm, BookingSummary, BookingActions } from './components';
import { useBookingState, useCompanyData } from './hooks';
import { STATUSES } from './constants';
import type { Booking, Company } from './types';

export function LargeComponent() {
  // Focused component logic using extracted pieces
}
```

**Benefits:**

- Each file has a single, clear responsibility
- Easier to navigate and understand
- Better testability (test each module independently)
- Improved code reusability
- Reduced merge conflicts
- Faster development (smaller files load faster in IDE)

---

## 6. Refactoring Workflow Context

### Pre-Refactoring Checklist

1. **Understand the code**: Read and comprehend current functionality
2. **Identify impact**: Find all usages of code being refactored
3. **Check tests**: Ensure existing tests cover the functionality
4. **Backup plan**: Know how to rollback if needed
5. **Review dependencies**: Check for external API changes

### Safe Refactoring Process

1. **Create backup branch** (if significant changes)
2. **Extract/copy code** to new location
3. **Update references** systematically
4. **Run tests** after each change
5. **Verify functionality** manually
6. **Clean up old code** only after verification

### Testing Strategy

- **Unit tests**: For extracted functions
- **Component tests**: For extracted components
- **Integration tests**: For complex refactoring
- **Regression tests**: Ensure no functionality lost

---

## 7. Common Refactoring Scenarios

### Scenario 1: Duplicate Booking Logic

**Problem**: Booking validation logic duplicated across multiple API routes

**Solution**:

1. Extract to `src/lib/bookingValidation.ts`
2. Update all API routes to use shared validation
3. Add comprehensive tests

### Scenario 2: Complex Component State

**Problem**: Component with too many useState hooks and complex logic

**Solution**:

1. Extract custom hooks for state management
2. Split component into smaller, focused components
3. Add proper TypeScript interfaces

### Scenario 3: Large Utility Functions

**Problem**: 200-line utility function doing multiple things

**Solution**:

1. Break into smaller, single-purpose functions
2. Extract configuration objects
3. Add clear documentation and examples

### Scenario 4: Inconsistent Error Handling

**Problem**: Different error handling patterns across similar functions

**Solution**:

1. Create standard error handling utilities
2. Apply consistent patterns
3. Update error messages for clarity

### Scenario 5: Files in Wrong Locations

**Problem**: Files placed in incorrect directories, violating Flexbike conventions

**Examples**:

- Reusable component in feature-specific directory
- Admin hook in general hooks directory
- Business logic utility in generic utils directory
- Server-only code in client directory

**Solution**:

1. Identify correct location based on file type and usage patterns
2. Search codebase for all imports referencing the file
3. Move file to correct location
4. Update all imports (relative and absolute)
5. Update barrel exports if applicable
6. Verify no circular dependencies introduced
7. Run tests to ensure functionality preserved
8. Check for any hardcoded paths or references

**Example**:

- **Before**: `src/app/admin/companies/components/BookingTable.tsx` (used in multiple admin pages)
- **After**: `src/app/admin/components/BookingTable.tsx` (moved to shared admin components)
- **Action**: Update all imports from `../companies/components/BookingTable` to `../../components/BookingTable`

### Scenario 6: Large Monolithic File

**Problem**: Single file containing multiple concerns (component + hooks + utilities + types + constants)

**Examples**:

- Component file >500 lines with embedded hooks, utilities, and types
- Utility file >300 lines with multiple unrelated functions
- API route file >400 lines with validation, transformation, and business logic

**Solution**:

1. **Analyze file structure** to identify logical boundaries:

   - Separate types/interfaces
   - Extract constants
   - Extract utility functions
   - Extract custom hooks
   - Extract sub-components
   - Keep main component/function focused

2. **Create new file structure**:

   - `types.ts` or `types/` folder for TypeScript interfaces
   - `constants.ts` for constants
   - `utils.ts` or `utils/` folder for utility functions
   - `hooks/` folder for custom hooks
   - `components/` folder for sub-components
   - Main file becomes focused entry point

3. **Update imports**:

   - Update main file to import from new modules
   - Update all files that import the original file
   - Ensure barrel exports (`index.ts`) if needed

4. **Verify functionality**:

   - Run tests to ensure nothing broke
   - Check for circular dependencies
   - Verify TypeScript types are preserved

5. **File size targets**:
   - Components: <500 lines (ideally <300)
   - Hooks: <200 lines (ideally <150)
   - Utilities: <300 lines (ideally <200)
   - Types: Can be larger but prefer splitting by domain

**Example**:

- **Before**: `src/app/admin/calendar/page.tsx` (977 lines)
  - Contains: Main page component, calendar logic, booking management, date utilities, types
- **After**:
  - `src/app/admin/calendar/page.tsx` (200 lines - main component)
  - `src/app/admin/calendar/types.ts` (types and interfaces)
  - `src/app/admin/calendar/utils.ts` (date and calendar utilities)
  - `src/app/admin/calendar/hooks/useCalendarState.ts` (calendar state management)
  - `src/app/admin/calendar/components/CalendarView.tsx` (calendar UI component)
  - `src/app/admin/calendar/components/BookingModal.tsx` (booking modal component)
- **Action**: Update all imports, maintain same public API

---

## 8. Error Handling & Rollback

### Detection Strategies

- **Syntax errors**: Caught by TypeScript/ESLint
- **Runtime errors**: Detected by unit tests
- **Logic errors**: Found by integration tests
- **Type errors**: Prevented by TypeScript

### Rollback Options

1. **Git rollback**: Revert entire commit
2. **Manual rollback**: Restore from backup
3. **Selective rollback**: Undo specific changes
4. **Partial rollback**: Keep successful parts, revert failed parts

### Recovery Process

1. **Stop deployment** if errors detected
2. **Assess impact** of failed refactoring
3. **Choose rollback strategy** based on impact
4. **Execute rollback** with minimal downtime
5. **Document lessons learned**

---

## 9. Integration with Other Agents

### Git Agent Integration

- **After refactoring**: Call Git agent to commit changes
- **Commit message format**: `refactor(scope): description`
- **Deployment options**: Offer preview/production deployment

### Testing Agent Integration

- **Pre-refactoring**: Run existing tests
- **During refactoring**: Generate new tests for extracted code
- **Post-refactoring**: Run full test suite

### Database Agent Integration

- **Schema changes**: Coordinate with database agent for migrations
- **Data validation**: Ensure refactoring doesn't break data integrity

---

## 10. Performance Considerations

### Refactoring Impact

- **Bundle size**: Monitor for increased bundle size
- **Runtime performance**: Check for performance regressions
- **Memory usage**: Watch for memory leaks in refactored code

### Optimization Opportunities

- **Memoization**: Add React.memo for expensive components
- **Lazy loading**: Split large components into chunks
- **Code splitting**: Separate concerns for better loading

---

## 11. Quality Metrics

### Success Criteria

- **Test coverage**: Maintain or improve coverage
- **Performance**: No regression in key metrics
- **Maintainability**: Code is easier to understand and modify
- **Consistency**: Follows project conventions
- **Documentation**: Code is well-documented

### Measurement Tools

- **ESLint**: Code quality and consistency
- **TypeScript**: Type safety and correctness
- **Jest**: Test coverage and reliability
- **Bundle analyzer**: Bundle size monitoring
- **Performance profiler**: Runtime performance tracking

## END OF SPECIFICATION
