---
name: nuxt-composables
description: Vue composables for reusable stateful logic. Use when understanding core composables (useWait, useFlash, useReactiveFilters, usePermissions), creating custom composables, or managing shared state.
---

# Nuxt Composables

Reusable stateful logic via Vue Composition API.

## Core Concepts

**[composables.md](references/composables.md)** - Core composables, patterns, custom composables

## Core Composables (from layers)

| Composable | Purpose |
|------------|---------|
| `useRepository(name)` | Get typed repository instance |
| `useWait()` | Global loading state management |
| `useFlash()` | Toast notification system |
| `usePermissions()` | Permission checking |
| `useReactiveFilters(defaults)` | URL-synced reactive filters |
| `useRealtime()` | WebSocket subscriptions |
| `useSlideover(name)` | Slideover control |
| `useModal(name)` | Modal control |
| `useConfirmationToast()` | Confirm/cancel toasts |

## Quick Examples

```typescript
// Loading states
const { start, stop, is, waitingFor } = useWait()
start(waitingFor.leads.creating)
// ... work
stop(waitingFor.leads.creating)

// Flash messages
const flash = useFlash()
flash.success('Lead created!')
flash.error('Failed to create lead')

// Permissions
const { can, cannot } = usePermissions()
if (can('leads.create')) { /* ... */ }

// Reactive filters
const { filters, hasFilters, resetFilters } = useReactiveFilters({
  status: undefined,
  page: 1,
}, { syncWithUrl: true })
```

## Custom Composable Pattern

```typescript
// app/composables/useUser.ts
let user = ref<User>()

export default function useUser() {
  const setUser = (data: BaseEntity) => {
    user.value = User.hydrate(data)
  }
  const clearUser = () => { user.value = undefined }

  return { user, setUser, clearUser }
}
```
