# Composable Patterns

## Core Composables Reference

### useWait

Global loading state management:

```typescript
const { start, stop, is, waitingFor } = useWait()

// Global operations (no ID)
waitingFor.posts.creating       // Creating any post
waitingFor.posts.listing        // Listing posts

// Per-item operations (with ID)
waitingFor.post.loading(ulid)   // Loading specific post
waitingFor.post.updating(ulid)  // Updating specific post
waitingFor.post.deleting(ulid)  // Deleting specific post

// Start/stop
start(waitingFor.posts.creating)
// ... do work
stop(waitingFor.posts.creating)

// Check state
const isCreating = is(waitingFor.posts.creating)

// In templates
<UButton :loading="is(waitingFor.posts.creating)">Create</UButton>
```

### useFlash

Toast notification system:

```typescript
const flash = useFlash()

// Basic messages
flash.success('Post created successfully.')
flash.error('Failed to create post.')
flash.warning('This action cannot be undone.')
flash.info('New updates available.')

// With description
flash.success('Post created', 'It will be published shortly.')
flash.error('Failed to create post', 'Please check your input.')

// Duration (ms)
flash.success('Quick message', undefined, { duration: 2000 })
```

### usePermissions

Permission checking:

```typescript
const { can, cannot, before, registerPermissions } = usePermissions()

// Check single permission
if (can('posts.create')) { /* ... */ }
if (cannot('posts.delete')) { /* ... */ }

// Check multiple (any of)
if (can(['posts.create', 'posts.update'])) { /* ... */ }

// Register permissions (usually in init plugin)
registerPermissions(['posts.list', 'posts.create', 'posts.update'])

// Admin bypass
before(() => {
  if (user.isAdmin) return true
  return null  // Check normally
})
```

### useReactiveFilters

URL-synced reactive filters:

```typescript
const { filters, hasFilters, resetFilters, resetFilter } = useReactiveFilters<GetPostsFilters>({
  // Default values
  status: undefined,
  isDraft: undefined,
  page: 1,
  size: 25,
}, {
  syncWithUrl: true,              // Sync to URL query params
  neverResetOn: ['page'],         // Don't reset page on filter changes
  debounceUrlUpdate: 300,         // Debounce URL updates
  provideAs: 'PostFilters',       // Provide to children
})

// Filters are reactive
filters.status = 'published'      // Triggers query refetch
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
const { open, close, isOpen, props } = useSlideover('create-post')

// Open slideover
open()

// Open with props
open({ author: selectedAuthor })

// Close
close()

// Check state
if (isOpen.value) { /* ... */ }

// Access props in slideover component
const slideover = useSlideover('create-post')
const author = computed(() => slideover.props.value?.author)
```

### useModal

Modal dialog control:

```typescript
const { open, close, isOpen, props } = useModal('delete-post')

// Open modal
open()

// Open with props
open({ post: selectedPost })

// Close
close()
```

### useConfirmationToast

Confirmation with confirm/cancel actions:

```typescript
const { trigger } = useConfirmationToast()

trigger({
  title: 'Delete Post?',
  description: 'This action cannot be undone.',
  confirmLabel: 'Delete',
  cancelLabel: 'Cancel',
  onConfirm: async () => {
    await deletePostAction(post)
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
const channel = privateChannel('posts.{id}', postId)

// Listen for events
channel.on('PostUpdated', (event) => {
  refresh()
})

// Multiple events
channel.on(['PostUpdated', 'PostDeleted'], (event) => {
  refresh()
})

// Cleanup
onUnmounted(() => {
  leaveChannel('posts.{id}', postId)
})
```

### useAppHeader

App header state:

```typescript
const { setAppHeader, appHeader } = useAppHeader()

setAppHeader({
  title: 'Posts',
  subtitle: 'Manage your posts',
  icon: 'lucide:file-text',
})

// Access in layout
<h1>{{ appHeader.title }}</h1>
```

### useBreadcrumbs

Breadcrumb navigation:

```typescript
const { setBreadcrumbs, breadcrumbs } = useBreadcrumbs()

setBreadcrumbs([
  { label: 'Content' },
  { label: 'Posts', to: '/posts' },
  { label: 'My Post' },
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
// app/composables/useCategories.ts
export function useCategories() {
  const builtTree = useState('categories')

  const getCategoryTree = (
    categories: MaybeRef<Category[] | undefined>,
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

### Task Count Composable

```typescript
// app/composables/useRemainingTaskCount.ts
export default function useRemainingTaskCount() {
  const remainingTaskCount = useState<number>('remaining-task-count', () => 0)
  const taskApi = useRepository('tasks')
  const { privateChannel } = useRealtime()

  const fetchTaskCount = async () => {
    const { data } = await taskApi.list({ filter: { status: 'pending' } })
    remainingTaskCount.value = data.length
  }

  const init = () => {
    fetchTaskCount()

    privateChannel(Tasks).on(
      [TaskCreated, TaskCompleted],
      fetchTaskCount
    )
  }

  return { remainingTaskCount, init }
}
```

---

## Composable Organization

```
app/
└── composables/
    ├── useUser.ts                 # User state management
    ├── useCategories.ts           # Domain-specific logic
    ├── useHandleActionError.ts    # Error handling utility
    └── useRemainingTaskCount.ts   # Real-time task tracking
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
