---
name: nuxt-features
description: Feature module pattern organizing domain logic into queries, mutations, and actions. Use when implementing data fetching with filters, API mutations with loading states, business logic with UI feedback, or organizing domain-specific code.
---

# Nuxt Features

Domain-based feature modules with three-layer pattern: queries, mutations, actions.

## Core Concepts

**[queries.md](references/queries.md)** - Reactive data fetching with filters
**[mutations.md](references/mutations.md)** - Pure API operations with loading states
**[actions.md](references/actions.md)** - Business logic with UI feedback

## Pattern Flow

```
Component → Action → Mutation → Repository → API
              ↓
         Query (reactive data fetching)
```

## Directory Structure

```
features/{domain}/
├── queries/
│   ├── get-leads-query.ts      # List with filters
│   └── get-lead-query.ts       # Single item
├── mutations/
│   ├── create-lead-mutation.ts
│   ├── update-lead-mutation.ts
│   └── delete-lead-mutation.ts
└── actions/
    ├── create-lead-action.ts
    ├── update-lead-action.ts
    └── delete-lead-action.ts
```

## Quick Examples

**Query** - Reactive data fetching:
```typescript
export default function getLeadsQueryFactory() {
  const leadApi = useRepository('leads')
  return (filters: MaybeRef<GetLeadsFilters>) => {
    return useFilterQuery('leads', () => leadApi.list(params), filters)
  }
}
```

**Mutation** - Pure API call:
```typescript
export default function createLeadMutationFactory() {
  const leadApi = useRepository('leads')
  const { start, stop, waitingFor } = useWait()
  return async (data: CreateLeadData) => {
    start(waitingFor.leads.creating)
    try { return (await leadApi.create(data)).data }
    finally { stop(waitingFor.leads.creating) }
  }
}
```

**Action** - Business logic + UI:
```typescript
export default function createLeadActionFactory() {
  const createLead = createLeadMutationFactory()
  const flash = useFlash()
  return async (data: CreateLeadData) => {
    const lead = await createLead(data)
    flash.success('Lead created!')
    return lead
  }
}
```
