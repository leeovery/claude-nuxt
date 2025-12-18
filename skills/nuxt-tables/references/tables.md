# Table Patterns

## Column Builder Utility

Create reusable column configurations:

```typescript
// app/utils/createColumnBuilder.ts
import type { TableColumn } from '@tanstack/vue-table'

export interface ColumnBuilder<TModel> {
  all(): TableColumn<TModel>[]
  build(columns: string[]): TableColumn<TModel>[]
  except(columnsToExclude: string[]): TableColumn<TModel>[]
}

export function createColumnBuilder<TModel>(
  columns: Record<string, TableColumn<TModel>>
): ColumnBuilder<TModel> {
  return {
    all: () => Object.values(columns),
    build: (columnKeys: string[]) =>
      columnKeys.map(key => columns[key]).filter(Boolean),
    except: (columnsToExclude: string[]) =>
      Object.keys(columns)
        .filter(key => !columnsToExclude.includes(key))
        .map(key => columns[key]),
  }
}
```

---

## Complete Table Configuration

```typescript
// app/tables/leads.ts
import { h } from 'vue'
import type { TableColumn } from '@tanstack/vue-table'
import type Lead from '~/models/Lead'
import Copyable from '~/components/Common/Copyable.vue'
import { truncateMiddle } from '#layers/base/app/utils'

// ULID column with copy functionality
const ulidColumn: TableColumn<Lead> = {
  id: 'ulid',
  accessorKey: 'ulid',
  header: 'ULID',
  cell: ({ row }) => h(Copyable, { content: row.original.ulid }, () =>
    truncateMiddle(row.original.ulid, 8)
  ),
}

// Status column with badge
const statusColumn: TableColumn<Lead> = {
  id: 'status',
  accessorKey: 'status',
  header: 'Status',
  cell: ({ row }) => h(UBadge, {
    color: row.getValue('status').color(),
  }, () => row.getValue('status').text),
}

// Contact column with link
const contactColumn: TableColumn<Lead> = {
  id: 'contact',
  accessorKey: 'contact.name',
  header: 'Contact',
  cell: ({ row }) => h(NuxtLink, {
    to: `/contacts/${row.original.contact.ulid}`,
    class: 'text-primary-500 hover:underline',
  }, () => row.original.contact.name),
}

// Email column
const emailColumn: TableColumn<Lead> = {
  id: 'email',
  accessorKey: 'contact.email',
  header: 'Email',
  cell: ({ row }) => row.original.contact.email,
}

// Demand column (truncated)
const demandColumn: TableColumn<Lead> = {
  id: 'demand',
  accessorKey: 'demand',
  header: 'Demand',
  cell: ({ row }) => h('span', {
    class: 'truncate max-w-xs',
    title: row.original.demand,
  }, row.original.demand),
}

// Dates column
const datesColumn: TableColumn<Lead> = {
  id: 'dates',
  header: 'Created',
  cell: ({ row }) => row.original.createdAt.format('DD MMM YYYY'),
}

// Secure links count column
const secureLinksColumn: TableColumn<Lead> = {
  id: 'secureLinks',
  header: 'Links',
  cell: ({ row }) => row.original.secureLinksCount ?? 0,
}

// Test flag column
const testFlagColumn: TableColumn<Lead> = {
  id: 'testFlag',
  header: 'Test',
  cell: ({ row }) => row.original.testFlag
    ? h(UBadge, { color: 'warning' }, () => 'Test')
    : null,
}

// Export builder
export const leadsColumnBuilder = createColumnBuilder<Lead>({
  ulid: ulidColumn,
  contact: contactColumn,
  email: emailColumn,
  status: statusColumn,
  demand: demandColumn,
  secureLinks: secureLinksColumn,
  testFlag: testFlagColumn,
  dates: datesColumn,
})
```

---

## Using Column Builder

```typescript
// All columns
const columns = leadsColumnBuilder.all()

// Specific columns
const columns = leadsColumnBuilder.build(['ulid', 'contact', 'status'])

// Exclude columns
const columns = leadsColumnBuilder.except(['ulid', 'dates'])
```

---

## XTable Component

```vue
<XTable
  :data="leads"
  :columns="columns"
  :loading="isLoading"
  :fetching="isFetching"
  :row-actions="rowActions"
  row-id="ulid"
  @row-click="handleRowClick"
  @row-select="handleRowSelect"
/>
```

### Props

| Prop | Type | Description |
|------|------|-------------|
| `data` | `T[]` | Table data array |
| `columns` | `TableColumn<T>[]` | Column definitions |
| `loading` | `boolean` | Initial loading state |
| `fetching` | `boolean` | Refetch loading state |
| `rowActions` | `(row) => Action[]` | Row action menu items |
| `rowId` | `string` | Unique identifier key |
| `selectable` | `boolean` | Enable row selection |

### Events

| Event | Payload | Description |
|-------|---------|-------------|
| `@row-click` | `{ row, event }` | Row clicked |
| `@row-select` | `{ selected: T[] }` | Selection changed |

---

## Row Actions

```typescript
import type { Row } from '@tanstack/vue-table'

const rowActions = computed(() => (row: Row<Lead>) => [
  // Navigation action
  {
    label: 'View lead',
    to: `/leads/${row.original.ulid}`,
  },
  // Navigation action
  {
    label: 'View contact',
    to: `/contacts/${row.original.contact.ulid}`,
  },
  // Function action
  {
    label: 'Edit lead',
    onSelect: () => openUpdateSlideover({ lead: row.original }),
  },
  // Destructive action
  {
    label: 'Delete lead',
    onSelect: () => handleDelete([row.original]),
  },
])
```

### Conditional Actions

```typescript
const rowActions = computed(() => (row: Row<Lead>) => {
  const actions = [
    { label: 'View', to: `/leads/${row.original.ulid}` },
  ]

  // Conditional edit
  if (can(UpdateLead)) {
    actions.push({
      label: 'Edit',
      onSelect: () => openEdit(row.original),
    })
  }

  // Conditional delete
  if (can(DeleteLead) && !row.original.status.isTerminal()) {
    actions.push({
      label: 'Delete',
      onSelect: () => handleDelete(row.original),
    })
  }

  return actions
})
```

---

## Table Component Pattern

```vue
<!-- app/components/Tables/LeadsTable.vue -->
<script lang="ts" setup>
import { leadsColumnBuilder } from '~/tables/leads'
import type { Row } from '@tanstack/vue-table'

const props = defineProps<{
  leads: Lead[]
  loading?: boolean
  fetching?: boolean
}>()

const emits = defineEmits<{
  edit: [lead: Lead]
  delete: [leads: Lead[]]
  select: [leads: Lead[]]
}>()

// Get all columns
const columns = leadsColumnBuilder.all()

// Or specific columns for this context
// const columns = leadsColumnBuilder.build(['contact', 'status', 'dates'])

// Row actions
const rowActions = computed(() => (row: Row<Lead>) => [
  { label: 'View lead', to: `/leads/${row.original.ulid}` },
  { label: 'View contact', to: `/contacts/${row.original.contact.ulid}` },
  { label: 'Edit lead', onSelect: () => emits('edit', row.original) },
  { label: 'Delete lead', onSelect: () => emits('delete', [row.original]) },
])

// Row click handler
const handleRowClick = ({ row }: { row: Row<Lead> }) => {
  navigateTo(`/leads/${row.original.ulid}`)
}
</script>

<template>
  <XTable
    :data="leads"
    :columns="columns"
    :loading="loading"
    :fetching="fetching"
    :row-actions="rowActions"
    row-id="ulid"
    @row-click="handleRowClick"
  />
</template>
```

---

## Custom Cell Renderers

### Using h() function

```typescript
import { h } from 'vue'

const statusColumn: TableColumn<Lead> = {
  id: 'status',
  header: 'Status',
  cell: ({ row }) => h(UBadge, {
    color: row.original.status.color(),
  }, () => row.original.status.text),
}
```

### Complex Cell

```typescript
const actionsColumn: TableColumn<Lead> = {
  id: 'actions',
  header: '',
  cell: ({ row }) => h('div', { class: 'flex gap-2' }, [
    h(UButton, {
      size: 'xs',
      variant: 'ghost',
      icon: 'i-heroicons-pencil',
      onClick: () => emits('edit', row.original),
    }),
    h(UButton, {
      size: 'xs',
      variant: 'ghost',
      color: 'error',
      icon: 'i-heroicons-trash',
      onClick: () => emits('delete', row.original),
    }),
  ]),
}
```

### Conditional Rendering

```typescript
const testFlagColumn: TableColumn<Lead> = {
  id: 'testFlag',
  header: 'Test',
  cell: ({ row }) => row.original.testFlag
    ? h(UBadge, { color: 'warning' }, () => 'Test')
    : null,
}
```

---

## Directory Structure

```
app/
└── tables/
    ├── leads.ts
    ├── contacts.ts
    └── evaluations.ts
```

---

## Naming Conventions

| Type | Convention |
|------|------------|
| File | kebab-case: `leads.ts` |
| Builder export | camelCase + "ColumnBuilder": `leadsColumnBuilder` |
| Column variables | camelCase + "Column": `statusColumn` |

---

## Related Skills

- **[nuxt-components](../../nuxt-components/SKILL.md)** - Table component patterns
- **[nuxt-pages](../../nuxt-pages/SKILL.md)** - Using tables in pages
