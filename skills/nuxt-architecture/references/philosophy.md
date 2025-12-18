# Architecture Philosophy

## Core Principles

### 1. Separation of Concerns

Clear boundaries between layers:

| Layer | Responsibility | Example |
|-------|---------------|---------|
| **Models** | Data structure, relations, casting | `Lead.ts` |
| **Repositories** | API access, CRUD operations | `LeadRepository.ts` |
| **Queries** | Reactive data fetching with filters | `get-leads-query.ts` |
| **Mutations** | Pure data operations | `create-lead-mutation.ts` |
| **Actions** | Business logic + UI feedback | `create-lead-action.ts` |
| **Components** | Presentation, user interaction | `LeadsTable.vue` |
| **Pages** | Route handling, layout composition | `leads/index.vue` |

### 2. Type Safety

Full TypeScript coverage:

```typescript
// Explicit types on all public interfaces
interface GetLeadsFilters extends Pick<Filters, 'page' | 'size' | 'search'> {
  status?: string
  testFlag?: boolean
}

// Models define typed properties
export default class Lead extends Model {
  ulid: string
  status: LeadStatus        // Enum, auto-cast from API
  contact: Contact          // Relation, auto-hydrated
  createdAt: DateValue      // Value object, auto-cast
}
```

### 3. Composable-First

Reusable logic via composables:

```typescript
// Encapsulate feature logic
export default function useUser() {
  const user = ref<User>()

  const setUser = (data: BaseEntity) => {
    user.value = User.hydrate(data)
  }

  return { user, setUser }
}
```

### 4. Domain-Driven Organization

Features grouped by domain, not technical concern:

```
features/
├── leads/
│   ├── queries/get-leads-query.ts
│   ├── mutations/create-lead-mutation.ts
│   └── actions/create-lead-action.ts
├── contacts/
│   └── ...
└── secure-links/
    └── ...
```

## Pattern Selection Guide

### When to Create Each Pattern

| Pattern | Create When |
|---------|-------------|
| **Model** | Entity from API with relations or computed properties |
| **Repository** | New API resource endpoint |
| **Query** | Need reactive data fetching with filters |
| **Mutation** | API call that modifies data |
| **Action** | Mutation + user feedback or orchestration |
| **Composable** | Reusable stateful logic |
| **Enum** | Fixed set of values with behavior (colors, labels) |
| **Value Object** | Typed wrapper with formatting (dates, money) |

### Decision Tree: Action vs Mutation

```
Need to modify data via API?
├── Yes → Does it need UI feedback (toast, modal, etc.)?
│   ├── Yes → Create Action (calls Mutation internally)
│   └── No → Create Mutation only
└── No → Use Query for fetching
```

### Decision Tree: Model vs Plain Interface

```
Data from API?
├── Yes → Does it have:
│   ├── Relations to other models? → Use Model
│   ├── Properties needing casting (dates, enums)? → Use Model
│   ├── Computed properties or methods? → Use Model
│   └── None of above → Plain interface is fine
└── No → Plain interface or type
```

## Anti-Patterns to Avoid

### 1. Fat Components

```typescript
// BAD: Business logic in component
const onSubmit = async () => {
  const { data } = await useRepository('leads').create(formData)
  flash.success('Lead created!')
  if (data.hasSecureLinks()) {
    // Show toast with link...
  }
}

// GOOD: Delegate to action
const createLeadAction = createLeadActionFactory()
const onSubmit = async () => {
  await createLeadAction(formData)
}
```

### 2. Skipping Layers

```typescript
// BAD: Component directly calls repository for mutations
const deleteButton = async () => {
  await useRepository('leads').delete(lead.ulid)
}

// GOOD: Use action for mutations with feedback
const deleteLeadAction = deleteLeadActionFactory()
const deleteButton = async () => {
  await deleteLeadAction(lead)
}
```

### 3. Untyped API Responses

```typescript
// BAD: Raw API data
const { data } = await leadApi.get(id)
console.log(data.status)  // string, no type safety

// GOOD: Hydrated model
const { data } = await leadApi.get(id)  // hydration enabled
console.log(data.status.color())  // LeadStatus enum with methods
```

## Layer Dependencies

```
Pages
  ↓
Components
  ↓
Actions ← Composables
  ↓
Mutations
  ↓
Queries ← Repositories
  ↓
Models ← Enums, Values
```

Each layer only imports from layers below it. Never import upward.

## File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Models | PascalCase | `Lead.ts`, `SecureLink.ts` |
| Repositories | PascalCase + "Repository" | `LeadRepository.ts` |
| Composables | camelCase, "use" prefix | `useUser.ts` |
| Queries | kebab-case + `-query` | `get-leads-query.ts` |
| Mutations | kebab-case + `-mutation` | `create-lead-mutation.ts` |
| Actions | kebab-case + `-action` | `create-lead-action.ts` |
| Components | PascalCase | `CreateLeadSlideover.vue` |
| Enums | PascalCase | `LeadStatus.ts` |
| Pages | kebab-case | `leads/[ulid].vue` |

## Related Skills

- **[nuxt-models](../../nuxt-models/SKILL.md)** - Model implementation details
- **[nuxt-repositories](../../nuxt-repositories/SKILL.md)** - Repository pattern
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Query/mutation/action pattern
- **[nuxt-components](../../nuxt-components/SKILL.md)** - Component organization
