---
name: nuxt-tables
description: Table components with column builder pattern and XTable. Use when creating data tables, defining columns with custom cells, implementing row actions, or building reusable table configurations.
---

# Nuxt Tables

Data tables with column builder pattern and XTable component.

## Core Concepts

**[tables.md](references/tables.md)** - Column builder, XTable, row actions

## Column Builder Pattern

```typescript
// app/tables/leads.ts
import { h } from 'vue'
import type { TableColumn } from '@tanstack/vue-table'

const statusColumn: TableColumn<Lead> = {
  id: 'status',
  accessorKey: 'status',
  header: 'Status',
  cell: ({ row }) => h(UBadge, {
    color: row.getValue('status').color()
  }, () => row.getValue('status').text),
}

export const leadsColumnBuilder = createColumnBuilder<Lead>({
  ulid: ulidColumn,
  contact: contactColumn,
  status: statusColumn,
  dates: datesColumn,
})

// Usage
const columns = leadsColumnBuilder.all()
const columns = leadsColumnBuilder.build(['ulid', 'status'])
const columns = leadsColumnBuilder.except(['dates'])
```

## XTable Usage

```vue
<XTable
  :data="leads"
  :columns="columns"
  :loading="isLoading"
  :fetching="isFetching"
  :row-actions="rowActions"
  row-id="ulid"
  @row-click="handleRowClick"
/>
```

## Row Actions

```typescript
const rowActions = computed(() => (row: Row<Lead>) => [
  { label: 'View', to: `/leads/${row.original.ulid}` },
  { label: 'Edit', onSelect: () => openEdit(row.original) },
  { label: 'Delete', onSelect: () => handleDelete(row.original) },
])
```
