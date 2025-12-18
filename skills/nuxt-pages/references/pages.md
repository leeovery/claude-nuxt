# Page Patterns

## File-Based Routing

Nuxt generates routes from the `pages/` directory structure:

| File | Route |
|------|-------|
| `pages/index.vue` | `/` |
| `pages/profile.vue` | `/profile` |
| `pages/leads/index.vue` | `/leads` |
| `pages/leads/[ulid].vue` | `/leads/:ulid` |
| `pages/auth/login.vue` | `/auth/login` |

### Dynamic Routes

```
pages/
├── leads/
│   ├── index.vue          # /leads
│   └── [ulid].vue         # /leads/:ulid
└── contacts/
    ├── index.vue          # /contacts
    └── [ulid]/
        ├── index.vue      # /contacts/:ulid
        └── edit.vue       # /contacts/:ulid/edit
```

---

## Page Meta

### Permissions

```typescript
// Single permission
definePageMeta({
  permissions: 'leads.list',
})

// Multiple permissions (any of)
definePageMeta({
  permissions: ['leads.list', 'contacts.list'],
})
```

### Layouts

```typescript
// Use specific layout
definePageMeta({
  layout: 'auth',
})

// Disable layout
definePageMeta({
  layout: false,
})
```

### Middleware

```typescript
// Apply middleware
definePageMeta({
  middleware: ['auth', 'verified'],
})
```

---

## Complete List Page

```vue
<!-- app/pages/leads/index.vue -->
<script lang="ts" setup>
import getLeadsQueryFactory, { type GetLeadsFilters } from '~/features/leads/queries/get-leads-query'
import deleteLeadActionFactory from '~/features/leads/actions/delete-lead-action'
import { ListLeads, CreateLead } from '~/constants/permissions'
import type { Row } from '@tanstack/vue-table'

// Page meta
definePageMeta({
  permissions: ListLeads,
})

// Page header
const { setAppHeader } = useAppHeader()
setAppHeader({
  title: 'Leads',
  icon: 'lucide:briefcase',
})

// Breadcrumbs
const { setBreadcrumbs } = useBreadcrumbs()
setBreadcrumbs([
  { label: 'Lead Management' },
  { label: 'Leads' },
])

// Reactive filters
const { filters, hasFilters, resetFilters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
}, {
  syncWithUrl: true,
})

// Query
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh, isLoading, isFetching, pagination } = getLeadsQuery(filters)

// Actions
const deleteLeadAction = deleteLeadActionFactory()

// Slideovers
const { open: openCreateSlideover } = useSlideover('create-lead')
const { open: openUpdateSlideover } = useSlideover('update-lead')

// Delete handling
const deleteLeads = async (items: Lead[]) => {
  for (const lead of items) {
    await deleteLeadAction(lead)
  }
  refresh()
}

// Row actions
const tableRowActions = computed(() => (row: Row<Lead>) => [
  { label: 'View lead', to: `/leads/${row.original.ulid}` },
  { label: 'View contact', to: `/contacts/${row.original.contact.ulid}` },
  { label: 'Edit lead', onSelect: () => openUpdateSlideover({ lead: row.original }) },
  { label: 'Delete lead', onSelect: () => deleteLeads([row.original]) },
])
</script>

<template>
  <div>
    <!-- Toolbar -->
    <div class="flex items-center gap-4 mb-4">
      <UInput
        v-model="filters.search"
        placeholder="Search..."
        icon="i-heroicons-magnifying-glass"
      />

      <USelect
        v-model="filters.status"
        :options="LeadStatus.values()"
        placeholder="Status"
        option-attribute="text"
      />

      <UButton
        v-if="hasFilters"
        variant="ghost"
        @click="resetFilters"
      >
        Clear filters
      </UButton>

      <div class="flex-1" />

      <UButton
        v-if="can(CreateLead)"
        icon="i-heroicons-plus"
        label="Create Lead"
        @click="openCreateSlideover()"
      />
    </div>

    <!-- Loading indicator -->
    <LoadingLine v-if="isFetching" />

    <!-- Table -->
    <LeadsTable
      :leads="leads?.data || []"
      :loading="isLoading"
      :fetching="isFetching"
      :row-actions="tableRowActions"
      @delete="deleteLeads"
    />

    <!-- Pagination -->
    <XPagination
      v-if="pagination"
      v-model:page="filters.page"
      :pagination="pagination"
      class="mt-4"
    />

    <!-- Slideovers -->
    <CreateLeadSlideover @close="refresh" />
    <UpdateLeadSlideover @close="refresh" />
  </div>
</template>
```

---

## Complete Detail Page

```vue
<!-- app/pages/leads/[ulid].vue -->
<script lang="ts" setup>
import getLeadQueryFactory from '~/features/leads/queries/get-lead-query'
import deleteLeadActionFactory from '~/features/leads/actions/delete-lead-action'
import { ShowLead, UpdateLead, DeleteLead } from '~/constants/permissions'

// Page meta
definePageMeta({
  permissions: ShowLead,
})

// Route params
const route = useRoute()
const router = useRouter()
const ulid = computed(() => route.params.ulid as string)

// Query
const getLeadQuery = getLeadQueryFactory()
const { data: lead, isLoading, refresh } = getLeadQuery(ulid)

// Actions
const deleteLeadAction = deleteLeadActionFactory()

// Page header (reactive)
const { setAppHeader } = useAppHeader()
watch(lead, (l) => {
  if (l) {
    setAppHeader({
      title: l.data.contact.name,
      subtitle: l.data.demand,
      icon: 'lucide:briefcase',
    })
  }
}, { immediate: true })

// Breadcrumbs (reactive)
const { setBreadcrumbs } = useBreadcrumbs()
watch(lead, (l) => {
  if (l) {
    setBreadcrumbs([
      { label: 'Lead Management' },
      { label: 'Leads', to: '/leads' },
      { label: l.data.contact.name },
    ])
  }
}, { immediate: true })

// Tabs
const tabs = computed(() => [
  { label: 'Details', slot: 'details' },
  {
    label: 'Secure Links',
    slot: 'secure-links',
    badge: lead.value?.data.secureLinksCount,
  },
  {
    label: 'Insights',
    slot: 'insights',
    badge: lead.value?.data.insightsCount,
  },
  {
    label: 'Conversations',
    slot: 'conversations',
    badge: lead.value?.data.conversationsCount,
  },
])

// Slideover
const { open: openEditSlideover } = useSlideover('edit-lead')

// Delete
const handleDelete = async () => {
  if (!lead.value) return
  await deleteLeadAction(lead.value.data)
  router.push('/leads')
}

// Real-time updates
const { privateChannel } = useRealtime()
privateChannel(Lead, ulid.value).on(LeadUpdated, refresh)
</script>

<template>
  <div>
    <!-- Loading -->
    <USkeleton v-if="isLoading" class="h-96" />

    <!-- Content -->
    <div v-else-if="lead">
      <!-- Actions -->
      <div class="flex gap-2 mb-4">
        <UButton
          v-if="can(UpdateLead)"
          @click="openEditSlideover({ lead: lead.data })"
        >
          Edit
        </UButton>
        <UButton
          v-if="can(DeleteLead)"
          color="error"
          @click="handleDelete"
        >
          Delete
        </UButton>
      </div>

      <!-- Tabs -->
      <UTabs :items="tabs">
        <template #details>
          <LeadDetail :lead="lead.data" />
        </template>

        <template #secure-links>
          <LeadSecureLinksTab :lead="lead.data" />
        </template>

        <template #insights>
          <LeadInsightsTab :lead="lead.data" />
        </template>

        <template #conversations>
          <ConversationTab :conversations="lead.data.conversations" />
        </template>
      </UTabs>
    </div>

    <!-- Slideover -->
    <UpdateLeadSlideover @close="refresh" />
  </div>
</template>
```

---

## Layouts

### Default Layout

```vue
<!-- app/layouts/default.vue -->
<script lang="ts" setup>
const { appHeader } = useAppHeader()
const { breadcrumbs } = useBreadcrumbs()
</script>

<template>
  <div class="flex min-h-screen">
    <!-- Sidebar -->
    <aside class="w-64 border-r">
      <Sidebar />
    </aside>

    <!-- Main content -->
    <main class="flex-1">
      <!-- Header -->
      <header class="border-b p-4">
        <div class="flex items-center gap-2">
          <UIcon v-if="appHeader.icon" :name="appHeader.icon" />
          <h1 class="text-xl font-semibold">{{ appHeader.title }}</h1>
        </div>
        <p v-if="appHeader.subtitle" class="text-sm text-gray-500">
          {{ appHeader.subtitle }}
        </p>
        <UBreadcrumb :items="breadcrumbs" class="mt-2" />
      </header>

      <!-- Page content -->
      <div class="p-4">
        <slot />
      </div>
    </main>
  </div>
</template>
```

### Auth Layout

```vue
<!-- app/layouts/auth.vue -->
<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="w-full max-w-md">
      <slot />
    </div>
  </div>
</template>
```

---

## Navigation Helpers

### useAppHeader

```typescript
const { setAppHeader, appHeader } = useAppHeader()

setAppHeader({
  title: 'Leads',
  subtitle: 'Manage your leads',
  icon: 'lucide:briefcase',
})
```

### useBreadcrumbs

```typescript
const { setBreadcrumbs, breadcrumbs } = useBreadcrumbs()

setBreadcrumbs([
  { label: 'Lead Management' },
  { label: 'Leads', to: '/leads' },
  { label: 'John Doe' },
])
```

---

## Redirect Page

```vue
<!-- app/pages/index.vue -->
<script lang="ts" setup>
// Redirect to leads on root
definePageMeta({
  redirect: '/leads',
})
</script>
```

---

## Login Page

```vue
<!-- app/pages/auth/login.vue -->
<script lang="ts" setup>
definePageMeta({
  layout: 'auth',
  middleware: 'guest',  // Only for unauthenticated
})

const { login } = useSanctumAuth()
const router = useRouter()

const credentials = ref({
  email: '',
  password: '',
})

const onSubmit = async () => {
  await login(credentials.value)
  router.push('/')
}
</script>

<template>
  <XCard title="Login">
    <form @submit.prevent="onSubmit">
      <UFormField label="Email" name="email">
        <UInput v-model="credentials.email" type="email" />
      </UFormField>

      <UFormField label="Password" name="password">
        <UInput v-model="credentials.password" type="password" />
      </UFormField>

      <UButton type="submit" class="w-full mt-4">
        Login
      </UButton>
    </form>
  </XCard>
</template>
```

---

## Naming Conventions

| File | Convention |
|------|------------|
| List pages | `index.vue` in resource folder |
| Detail pages | `[param].vue` with param name |
| Nested routes | Folder structure matches URL |
| Layouts | kebab-case in `layouts/` |

---

## Related Skills

- **[nuxt-auth](../../nuxt-auth/SKILL.md)** - Page authorization
- **[nuxt-components](../../nuxt-components/SKILL.md)** - Component patterns
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Queries for pages
