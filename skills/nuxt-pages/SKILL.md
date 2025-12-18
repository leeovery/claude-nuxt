---
name: nuxt-pages
description: File-based routing with page patterns for lists, details, and navigation. Use when creating pages, defining page meta (permissions, layouts), implementing list/detail patterns, or setting up breadcrumbs and headers.
---

# Nuxt Pages

File-based routing with common page patterns and navigation.

## Core Concepts

**[pages.md](references/pages.md)** - Page patterns, meta, layouts, navigation

## Directory Structure

```
pages/
├── index.vue              # Dashboard/redirect
├── profile.vue            # User profile
├── auth/
│   └── login.vue          # Login page
├── leads/
│   ├── index.vue          # List view
│   └── [ulid].vue         # Detail view
└── contacts/
    ├── index.vue
    └── [ulid].vue
```

## List Page Pattern

```vue
<script lang="ts" setup>
import getLeadsQueryFactory, { type GetLeadsFilters } from '~/features/leads/queries/get-leads-query'
import { ListLeads, CreateLead } from '~/constants/permissions'

definePageMeta({ permissions: ListLeads })

const { setAppHeader } = useAppHeader()
setAppHeader({ title: 'Leads', icon: 'lucide:briefcase' })

const { filters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  page: 1,
  size: 25,
})

const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, isLoading, pagination } = getLeadsQuery(filters)
</script>

<template>
  <div>
    <UInput v-model="filters.search" placeholder="Search..." />
    <LeadsTable :leads="leads?.data || []" :loading="isLoading" />
    <XPagination v-if="pagination" v-model:page="filters.page" :pagination="pagination" />
  </div>
</template>
```

## Detail Page Pattern

```vue
<script lang="ts" setup>
import getLeadQueryFactory from '~/features/leads/queries/get-lead-query'

definePageMeta({ permissions: 'leads.show' })

const route = useRoute()
const ulid = computed(() => route.params.ulid as string)

const getLeadQuery = getLeadQueryFactory()
const { data: lead, isLoading } = getLeadQuery(ulid)
</script>

<template>
  <UTabs v-if="!isLoading && lead" :items="tabs">
    <template #details><LeadDetail :lead="lead.data" /></template>
  </UTabs>
</template>
```
