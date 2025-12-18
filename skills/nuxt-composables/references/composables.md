# Composable Patterns

## Core Composables Reference

### useWait

Global loading state management:

```typescript
const { start, stop, is, waitingFor } = useWait()

// Global operations (no ID)
waitingFor.leads.creating       // Creating any lead
waitingFor.leads.listing        // Listing leads

// Per-item operations (with ID)
waitingFor.lead.loading(ulid)   // Loading specific lead
waitingFor.lead.updating(ulid)  // Updating specific lead
waitingFor.lead.deleting(ulid)  // Deleting specific lead

// Start/stop
start(waitingFor.leads.creating)
// ... do work
stop(waitingFor.leads.creating)

// Check state
const isCreating = is(waitingFor.leads.creating)

// In templates
<UButton :loading="is(waitingFor.leads.creating)">Create</UButton>
```

### useFlash

Toast notification system:

```typescript
const flash = useFlash()

// Basic messages
flash.success('Lead created successfully.')
flash.error('Failed to create lead.')
flash.warning('This action cannot be undone.')
flash.info('New updates available.')

// With description
flash.success('Lead created', 'A secure link has been sent.')
flash.error('Failed to create lead', 'Please check your input.')

// Duration (ms)
flash.success('Quick message', undefined, { duration: 2000 })
```

### usePermissions

Permission checking:

```typescript
const { can, cannot, before, registerPermissions } = usePermissions()

// Check single permission
if (can('leads.create')) { /* ... */ }
if (cannot('leads.delete')) { /* ... */ }

// Check multiple (any of)
if (can(['leads.create', 'leads.update'])) { /* ... */ }

// Register permissions (usually in init plugin)
registerPermissions(['leads.list', 'leads.create', 'leads.update'])

// Admin bypass
before(() => {
  if (user.isAdmin) return true
  return null  // Check normally
})
```

### useReactiveFilters

URL-synced reactive filters:

```typescript
const { filters, hasFilters, resetFilters, resetFilter } = useReactiveFilters<GetLeadsFilters>({
  // Default values
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
}, {
  syncWithUrl: true,              // Sync to URL query params
  neverResetOn: ['page'],         // Don't reset page on filter changes
  debounceUrlUpdate: 300,         // Debounce URL updates
  provideAs: 'LeadFilters',       // Provide to children
})

// Filters are reactive
filters.status = 'new lead'      // Triggers query refetch
filters.page = 2                  // Triggers query refetch

// Check if any filters active
if (hasFilters.value) {
  // Show "Clear filters" button
}

// Reset single filter
resetFilter('status')

// Reset all filters
resetFilters()
```

### useSlideover

Slideover panel control:

```typescript
const { open, close, isOpen, props } = useSlideover('create-lead')

// Open slideover
open()

// Open with props
open({ contact: selectedContact })

// Close
close()

// Check state
if (isOpen.value) { /* ... */ }

// Access props in slideover component
const slideover = useSlideover('create-lead')
const contact = computed(() => slideover.props.value?.contact)
```

### useModal

Modal dialog control:

```typescript
const { open, close, isOpen, props } = useModal('delete-lead')

// Open modal
open()

// Open with props
open({ lead: selectedLead })

// Close
close()
```

### useConfirmationToast

Confirmation with confirm/cancel actions:

```typescript
const { trigger } = useConfirmationToast()

trigger({
  title: 'Delete Lead?',
  description: 'This action cannot be undone.',
  confirmLabel: 'Delete',
  cancelLabel: 'Cancel',
  onConfirm: async () => {
    await deleteLeadAction(lead)
  },
  onCancel: () => {
    // Optional cancel handler
  },
})
```

### useRealtime

WebSocket channel subscriptions:

```typescript
const { privateChannel, presenceChannel, leaveChannel } = useRealtime()

// Subscribe to private channel
const channel = privateChannel('leads.{id}', leadId)

// Listen for events
channel.on('LeadUpdated', (event) => {
  refresh()
})

// Multiple events
channel.on(['LeadUpdated', 'LeadDeleted'], (event) => {
  refresh()
})

// Cleanup
onUnmounted(() => {
  leaveChannel('leads.{id}', leadId)
})
```

### useAppHeader

App header state:

```typescript
const { setAppHeader, appHeader } = useAppHeader()

setAppHeader({
  title: 'Leads',
  subtitle: 'Manage your leads',
  icon: 'lucide:briefcase',
})

// Access in layout
<h1>{{ appHeader.title }}</h1>
```

### useBreadcrumbs

Breadcrumb navigation:

```typescript
const { setBreadcrumbs, breadcrumbs } = useBreadcrumbs()

setBreadcrumbs([
  { label: 'Lead Management' },
  { label: 'Leads', to: '/leads' },
  { label: 'John Doe' },
])

// Access in layout
<UBreadcrumb :items="breadcrumbs" />
```

---

## Custom Composable Patterns

### Singleton State Pattern

State persists across components:

```typescript
// app/composables/useUser.ts
let user = ref<User>()  // Singleton - defined outside function

export default function useUser() {
  const getUser = () => user

  const setUser = (data: BaseEntity) => {
    user.value = User.hydrate(data)
  }

  const clearUser = () => {
    user.value = undefined
  }

  return { user, getUser, setUser, clearUser }
}
```

### Factory Pattern

Create new instance each call:

```typescript
// app/composables/useForm.ts
export default function useForm<T>(url: string, method: string, initialData: T) {
  const data = ref(initialData)
  const processing = ref(false)
  const errors = ref<Record<string, string[]>>({})

  const submit = async () => {
    processing.value = true
    try {
      // ... submit logic
    } finally {
      processing.value = false
    }
  }

  return { data, processing, errors, submit }
}
```

### Domain-Specific Composable

```typescript
// app/composables/useFeedbackCategories.ts
export function useFeedbackCategories() {
  const builtTree = useState('feedback-categories')

  const getCategoryTree = (
    categories: MaybeRef<FeedbackCategory[] | undefined>,
    type?: string
  ): ComputedRef<Category | CategoryTree | undefined> => {
    return computed(() => {
      // Build hierarchical tree from flat array
      const cats = toValue(categories)
      if (!cats) return undefined

      // ... tree building logic

      return type ? tree[type] : tree
    })
  }

  return { getCategoryTree, builtTree }
}
```

### Error Handler Composable

```typescript
// app/composables/useHandleActionError.ts
export function useHandleActionError() {
  const flash = useFlash()

  const handleActionError = (
    error: unknown,
    context: { entity: string; operation: string }
  ) => {
    const message = `Failed to ${context.operation} ${context.entity}.`

    if (error instanceof ValidationError) {
      flash.error(message, error.message)
    } else {
      flash.error(message, 'An unexpected error occurred.')
    }

    return error
  }

  return { handleActionError }
}
```

### Job Count Composable

```typescript
// app/composables/useRemainingJobCount.ts
export default function useRemainingJobCount() {
  const remainingJobCount = useState<number>('remaining-job-count', () => 0)
  const jobApi = useRepository('jobs')
  const { privateChannel } = useRealtime()

  const fetchJobCount = async () => {
    const { data } = await jobApi.list({ filter: { status: 'pending' } })
    remainingJobCount.value = data.length
  }

  const init = () => {
    fetchJobCount()

    privateChannel(MatchJobs).on(
      [MatchJobCreated, MatchJobComplete],
      fetchJobCount
    )
  }

  return { remainingJobCount, init }
}
```

---

## Composable Organization

```
app/
└── composables/
    ├── useUser.ts                 # User state management
    ├── useFeedbackCategories.ts   # Domain-specific logic
    ├── useHandleActionError.ts    # Error handling utility
    └── useRemainingJobCount.ts    # Real-time job tracking
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | camelCase with "use" prefix: `useUser.ts` |
| Export | Default function: `export default function useUser()` |
| Return | Object with reactive refs and methods |

---

## Best Practices

1. **Single Responsibility** - Each composable does one thing
2. **Return Object** - Always return object, not array (more extensible)
3. **Reactive by Default** - Use `ref` or `computed` for state
4. **Cleanup** - Use `onUnmounted` for subscriptions
5. **Type Everything** - Full TypeScript coverage

---

## Related Skills

- **[nuxt-layers](../../nuxt-layers/SKILL.md)** - Layer composables
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Using composables in features
- **[nuxt-realtime](../../nuxt-realtime/SKILL.md)** - Real-time composables
