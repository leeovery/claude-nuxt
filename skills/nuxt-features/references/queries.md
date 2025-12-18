# Query Patterns

Queries provide reactive data fetching with automatic refetching when filters change.

## Core Composables

```typescript
// Simple cached query
useQuery(key, fetcher, options)

// Query with reactive filters (auto-refetch)
useFilterQuery(key, fetcher, filters)
```

---

## Basic Query (No Filters)

```typescript
// app/features/leads/queries/get-lead-query.ts
import type { MaybeRef } from 'vue'

export default function getLeadQueryFactory() {
  const leadApi = useRepository('leads')

  return (ulid: MaybeRef<string>) => {
    return useQuery(`lead-${toValue(ulid)}`, () => {
      const params = useJsonSpec()
        .include('contact', 'secure-links', 'insights', 'conversations')
        .build()

      return leadApi.get(toValue(ulid), params)
    })
  }
}
```

### Usage

```typescript
const ulid = computed(() => route.params.ulid as string)
const getLeadQuery = getLeadQueryFactory()
const { data: lead, isLoading, refresh } = getLeadQuery(ulid)

// Access data
lead.value?.data.contact.name
```

---

## Filter Query (Auto-Refetch)

```typescript
// app/features/leads/queries/get-leads-query.ts
import type { Filters } from '#layers/base/app/types'
import type { MaybeRef } from 'vue'
import { KebabCase } from '#layers/base/app/utils'

// Define filter interface
export interface GetLeadsFilters extends Pick<Filters, 'page' | 'size' | 'search'> {
  status?: string
  testFlag?: boolean
}

export default function getLeadsQueryFactory() {
  const leadApi = useRepository('leads')

  return (filters: MaybeRef<GetLeadsFilters>) => {
    return useFilterQuery('leads', () => {
      // Build JSON:API spec params
      const params = useJsonSpec()
        .filters(filters, KebabCase)           // Apply filters
        .include('secure-links', 'contact')     // Include relations
        .build()

      return leadApi.list(params)
    }, filters)
  }
}
```

### Usage

```typescript
// Set up reactive filters
const { filters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
})

// Create query
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh, isLoading, isFetching, pagination } = getLeadsQuery(filters)

// Data auto-refetches when filters change
filters.status = 'new lead'  // Triggers refetch
filters.page = 2             // Triggers refetch
```

---

## useJsonSpec Builder

Build JSON:API query parameters:

```typescript
const params = useJsonSpec()
  // Apply filters from reactive object
  .filters(filters, KebabCase)

  // Include relations
  .include('contact', 'secure-links')

  // Or include as array
  .include(['contact', 'secure-links'])

  // Sort
  .sort('-created_at', 'name')

  // Sparse fieldsets
  .fields('leads', ['ulid', 'status', 'created_at'])

  // Build final params object
  .build()
```

### Filter Transformation

```typescript
// KebabCase transforms filter keys: testFlag → test-flag
.filters(filters, KebabCase)

// No transformation
.filters(filters)

// Custom transformation
.filters(filters, (key) => key.toUpperCase())
```

---

## Query Return Values

```typescript
const {
  data,        // Reactive data (Ref<T>)
  isLoading,   // Initial load in progress
  isFetching,  // Any fetch in progress (including refetch)
  error,       // Error if failed
  refresh,     // Manual refresh function
  pagination,  // Pagination metadata (if paginated)
} = getLeadsQuery(filters)
```

### Pagination Object

```typescript
pagination: {
  currentPage: number
  lastPage: number
  perPage: number
  total: number
  from: number
  to: number
}
```

---

## Complete Query Examples

### Leads by Contact

```typescript
// app/features/leads/queries/get-leads-by-contact-query.ts
export interface GetLeadsByContactFilters extends Pick<Filters, 'page' | 'size'> {
  status?: string
}

export default function getLeadsByContactQueryFactory() {
  const leadApi = useRepository('leads')

  return (contactUlid: MaybeRef<string>, filters: MaybeRef<GetLeadsByContactFilters>) => {
    return useFilterQuery(`leads-by-contact-${toValue(contactUlid)}`, () => {
      const params = useJsonSpec()
        .filters(filters, KebabCase)
        .include('secure-links')
        .build()

      return leadApi.listByContact(toValue(contactUlid), params)
    }, filters)
  }
}
```

### Contacts with Search

```typescript
// app/features/contacts/queries/get-contacts-query.ts
export interface GetContactsFilters extends Pick<Filters, 'page' | 'size' | 'search'> {}

export default function getContactsQueryFactory() {
  const contactApi = useRepository('contacts')

  return (filters: MaybeRef<GetContactsFilters>) => {
    return useFilterQuery('contacts', () => {
      const params = useJsonSpec()
        .filters(filters, KebabCase)
        .include('leads')
        .build()

      return contactApi.list(params)
    }, filters)
  }
}
```

### Evaluations by Status

```typescript
// app/features/evaluations/queries/get-evaluations-query.ts
export interface GetEvaluationsFilters extends Pick<Filters, 'page' | 'size'> {
  status?: string
  reviewStatus?: string
}

export default function getEvaluationsQueryFactory() {
  const evalApi = useRepository('evaluations')

  return (filters: MaybeRef<GetEvaluationsFilters>) => {
    return useFilterQuery('evaluations', () => {
      const params = useJsonSpec()
        .filters(filters, KebabCase)
        .sort('-created_at')
        .build()

      return evalApi.list(params)
    }, filters)
  }
}
```

---

## Using Queries in Pages

```vue
<script lang="ts" setup>
import getLeadsQueryFactory, { type GetLeadsFilters } from '~/features/leads/queries/get-leads-query'

// Set up filters
const { filters, hasFilters, resetFilters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
}, {
  syncWithUrl: true,
})

// Create query
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh, isLoading, isFetching, pagination } = getLeadsQuery(filters)
</script>

<template>
  <div>
    <!-- Loading state -->
    <LoadingLine v-if="isFetching" />

    <!-- Filters -->
    <USelect v-model="filters.status" :options="statusOptions" />

    <!-- Data -->
    <LeadsTable :leads="leads?.data || []" :loading="isLoading" />

    <!-- Pagination -->
    <XPagination v-if="pagination" v-model:page="filters.page" :pagination="pagination" />
  </div>
</template>
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | `get-{resource}-query.ts`, `get-{resource}s-query.ts` |
| Factory function | `get{Resource}QueryFactory`, `get{Resource}sQueryFactory` |
| Filter interface | `Get{Resource}Filters`, `Get{Resource}sFilters` |

---

## Related Skills

- **[nuxt-repositories](../../nuxt-repositories/SKILL.md)** - Repositories used in queries
- **[nuxt-composables](../../nuxt-composables/SKILL.md)** - useReactiveFilters details
