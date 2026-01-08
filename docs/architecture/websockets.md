# WebSocket Implementation Guide

## Overview

Flexbike implements real-time functionality using **Supabase Real-Time Channels**, which provides WebSocket capabilities under the hood. This enables live updates across the application for booking status changes, admin notifications, and cross-tab synchronization.

## Architecture

### Supabase Real-Time Channels

Flexbike uses Supabase's built-in real-time infrastructure instead of traditional WebSockets:

```typescript
// Supabase channel creation
const channel = supabase.channel("channel-name");

// Subscribe to events
channel.on("broadcast", { event: "event-name" }, callback).subscribe();
```

This abstracts away WebSocket complexity while providing the same real-time benefits.

## Current Implementation

### 1. Global Booking Status Listener

**File**: `src/app/components/GlobalBookingStatusListener.tsx`

**Purpose**: Monitors booking status changes and shows real-time notifications to users.

**Key Features**:

- Cross-tab synchronization using BroadcastChannel
- Real-time booking status updates
- Payment status notifications
- Audio notifications for status changes

**Implementation**:

```typescript
// Setup WebSocket subscription
const channel = sb.channel(`bookings:${bookingId}`);

channel
  .on("broadcast", { event: "updated" }, async (msg: any) => {
    // Fetch latest booking data and show notification
    const booking = await fetch(`/api/bookings/${bookingId}`).then((r) =>
      r.json()
    );
    showStatusModal(booking);
    playChime();
  })
  .on("broadcast", { event: "note" }, async (payload: any) => {
    // Handle note/admin updates
    const booking = await fetch(`/api/bookings/${bookingId}`).then((r) =>
      r.json()
    );
    showNoteModal(booking, payload.payload.note);
  })
  .subscribe();
```

### 2. Booking Preview Modal (Admin)

**File**: `src/app/components/BookingPreviewModal.tsx`

**Purpose**: Admin interface for viewing booking details with real-time connection status.

**Key Features**:

- Connection status indicator
- WebSocket health monitoring
- Test channel for connection validation

**Implementation**:

```typescript
// Test connection setup
const testChannel = sb.channel("admin-test-connection");

testChannel
  .on("broadcast", { event: "test" }, (payload) => {
    console.log("WebSocket test received:", payload);
  })
  .subscribe((status: string) => {
    if (status === "SUBSCRIBED") {
      setConnectionStatus("connected");
    }
  });
```

### 3. Company Day View (Admin Calendar)

**File**: `src/app/admin/components/CompanyDayView.tsx`

**Purpose**: Admin calendar view with real-time booking updates.

**Key Features**:

- Live booking updates in calendar
- Real-time occupancy changes
- Admin notification system

**Implementation**:

```typescript
const channel = supabase.channel("admin-dayview-bookings-realtime");

channel
  .on(
    "postgres_changes",
    {
      event: "*",
      schema: "public",
      table: "bookings",
    },
    (payload) => {
      console.log("DayView WebSocket: Received booking update:", payload);
      // Update calendar UI
      refreshCalendarData();
    }
  )
  .subscribe((status) => {
    console.log("DayView WebSocket status:", status);
  });
```

### 4. useBooking Hook

**File**: `src/hooks/useBooking.ts`

**Purpose**: React hook providing real-time booking data with SWR caching.

**Key Features**:

- WebSocket-driven cache invalidation
- Fallback polling (30-second intervals)
- Real-time payment status updates

**Implementation**:

```typescript
// WebSocket listener for real-time updates
const channel = sb.channel(`bookings:${bookingId}`);

channel
  .on("broadcast", { event: "updated" }, (payload) => {
    // Invalidate SWR cache to trigger fresh fetch
    mutate();
  })
  .on("broadcast", { event: "note" }, (payload) => {
    // Handle note changes
    mutate();
  })
  .subscribe((status) => {
    if (status === "SUBSCRIBED") {
      console.log("WebSocket subscribed for booking:", bookingId);
    }
  });
```

## Event System

### Broadcast Events

#### Booking Updates (`event: 'updated'`)

- Triggered when booking status changes
- Payload: `{ bookingId, status, paymentStatus, ... }`
- Handled by: GlobalBookingStatusListener, useBooking hook

#### Note Changes (`event: 'note'`)

- Triggered when admin adds notes or defers bookings
- Payload: `{ note: 'DEFERRAL::reason\nadditional notes' }`
- Handled by: GlobalBookingStatusListener, useBooking hook

#### Test Events (`event: 'test'`)

- Used for connection health checks
- Payload: `{ message: 'test message', timestamp }`
- Handled by: BookingPreviewModal

### Database Change Events

#### PostgreSQL Changes

- Listens to direct database changes
- Used in admin calendar for live updates
- Event: `postgres_changes`
- Schema: `public`
- Tables: `bookings`

## Broadcasting Messages

### Server-Side Broadcasting

Messages are broadcast from API routes using Supabase:

```typescript
// In API route handler
const supabase = getSupabaseServer();

await supabase.channel(`bookings:${bookingId}`).send({
  type: "broadcast",
  event: "updated",
  payload: { bookingId, status: "confirmed" },
});
```

### Current Broadcasting Points

1. **Booking Status Updates** (`/api/bookings/[id]/status`)
2. **Payment Processing** (`/api/stripe/webhook`)
3. **Admin Actions** (`/api/admin/bookings/[id]/*`)
4. **Note Updates** (`/api/bookings/[id]/notes`)

## Cross-Tab Synchronization

### BroadcastChannel API

For communication between browser tabs:

```typescript
// Setup cross-tab communication
const broadcastChannel = new BroadcastChannel(`booking-status-${bookingId}`);

broadcastChannel.onmessage = (event) => {
  const { type, statusKey, booking } = event.data;
  if (type === "STATUS_CHANGE") {
    // Handle status change from another tab
    showModal(booking);
  }
};

// Broadcast to other tabs
broadcastChannel.postMessage({
  type: "STATUS_CHANGE",
  statusKey: "confirmed|paid",
  booking: bookingData,
});
```

## Connection Management

### Connection States

- **connecting**: Establishing WebSocket connection
- **connected**: Successfully subscribed to channel
- **disconnected**: Connection lost or failed

### Error Handling

```typescript
channel.subscribe((status) => {
  if (status === "SUBSCRIBED") {
    console.log("WebSocket connected");
    setConnectionStatus("connected");
  } else if (status === "CLOSED" || status === "CHANNEL_ERROR") {
    console.warn("WebSocket connection issue:", status);
    setConnectionStatus("disconnected");
    // Implement reconnection logic
  }
});
```

### Cleanup

Proper cleanup is crucial to prevent memory leaks:

```typescript
useEffect(() => {
  // Setup WebSocket
  const channel = setupWebSocket();

  return () => {
    // Cleanup
    supabase.removeChannel(channel);
  };
}, [dependencies]);
```

## Security Considerations

### Authentication

- WebSocket connections inherit Supabase auth
- Row Level Security (RLS) applies to real-time subscriptions
- API key authentication for server-side broadcasting

### Data Privacy

- Users only receive updates for their own bookings
- Admin channels are protected by role-based access
- Sensitive data is filtered before broadcasting

## Performance Optimization

### Channel Naming

- Use specific channel names: `bookings:${bookingId}`
- Avoid wildcard channels that broadcast to all users
- Clean up unused channels promptly

### Message Batching

- Batch related updates when possible
- Use efficient payload structures
- Avoid sending unnecessary data

### Connection Limits

- Monitor active connections per user
- Implement connection timeouts
- Handle high-traffic scenarios

## Testing WebSocket Functionality

### Unit Tests

```typescript
describe("WebSocket Connection", () => {
  it("should subscribe to booking channel", () => {
    const channel = supabase.channel("bookings:test-id");
    expect(channel).toBeDefined();
  });

  it("should handle connection status changes", () => {
    // Test subscription callbacks
  });
});
```

### Integration Tests

```typescript
describe("Real-time Updates", () => {
  it("should receive booking status updates", async () => {
    // Simulate status change
    // Verify UI updates
    // Check cross-tab synchronization
  });
});
```

### Manual Testing Checklist

- [ ] WebSocket connection establishes successfully
- [ ] Real-time booking updates appear
- [ ] Cross-tab synchronization works
- [ ] Admin calendar updates live
- [ ] Connection recovers from network issues
- [ ] Notifications play correctly

## Monitoring & Debugging

### Client-Side Logging

All WebSocket events are logged with timestamps:

```
[2025-01-07T10:30:15.123Z] GlobalBookingStatusListener: received updated event
[2025-01-07T10:30:15.124Z] useBooking: WebSocket subscribed for booking: abc-123
```

### Server-Side Monitoring

- Supabase dashboard shows active connections
- API logs show broadcast events
- Error rates and latency metrics

### Common Issues

1. **Connection failures**: Check Supabase service status
2. **Missing updates**: Verify channel names match
3. **Cross-tab issues**: Check BroadcastChannel support
4. **Performance**: Monitor connection counts

## Extending WebSocket Functionality

### Adding New Events

1. **Define the event** in broadcasting code:

```typescript
await supabase.channel(`bookings:${bookingId}`).send({
  type: "broadcast",
  event: "new-event-name",
  payload: { data: "value" },
});
```

2. **Add listener** in client components:

```typescript
channel.on("broadcast", { event: "new-event-name" }, (payload) => {
  // Handle new event
});
```

3. **Update tests** and documentation

### Adding New Channels

1. **Choose channel naming** convention
2. **Implement authentication** and authorization
3. **Add cleanup** logic
4. **Document** the new channel usage

## Summary

Flexbike's WebSocket implementation provides robust real-time functionality using Supabase's infrastructure:

- ✅ **Real-time booking updates** across all user interfaces
- ✅ **Cross-tab synchronization** for seamless UX
- ✅ **Admin live updates** for calendar management
- ✅ **Status notifications** with audio feedback
- ✅ **Payment processing** updates
- ✅ **Comprehensive error handling** and connection management
- ✅ **Security** through Supabase auth and RLS
- ✅ **Performance optimized** with proper cleanup and monitoring

The implementation successfully provides WebSocket-like real-time capabilities while leveraging Supabase's managed infrastructure for reliability and scalability.
