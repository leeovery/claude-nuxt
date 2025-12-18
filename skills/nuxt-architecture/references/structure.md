# Project Structure

## Complete Directory Layout

```
project-root/
├── app/                              # Main application code
│   ├── assets/
│   │   ├── css/main.css             # Global styles (Tailwind)
│   │   └── images/
│   ├── components/                   # Vue components by type
│   │   ├── Common/                  # Shared/generic (TeriLogo.vue)
│   │   ├── Detail/                  # Detail views (LeadDetail.vue)
│   │   ├── Form/                    # Form inputs (ContactEmailInput.vue)
│   │   ├── Modals/                  # Modal dialogs (DeleteLeadModal.vue)
│   │   ├── Nav/                     # Navigation (UserMenu.vue)
│   │   ├── Slideovers/              # Slideout panels (CreateLeadSlideover.vue)
│   │   ├── TabSections/             # Tab content (LeadInsightsTab.vue)
│   │   └── Tables/                  # Table components (LeadsTable.vue)
│   ├── composables/                 # Custom Vue composables
│   │   ├── useUser.ts
│   │   ├── useFeedbackCategories.ts
│   │   └── useHandleActionError.ts
│   ├── constants/                   # App-wide constants
│   │   ├── channels.ts              # WebSocket channel names
│   │   ├── events.ts                # Event names
│   │   ├── permissions.ts           # Permission strings
│   │   └── symbols.ts               # Vue injection symbols
│   ├── enums/                       # TypeScript enums with behavior
│   │   ├── LeadStatus.ts
│   │   ├── EvaluationStatus.ts
│   │   └── SecureLinkStatus.ts
│   ├── errors/                      # Custom error classes
│   │   └── (optional app-specific)
│   ├── features/                    # Domain-based feature modules
│   │   ├── leads/
│   │   │   ├── queries/
│   │   │   │   ├── get-leads-query.ts
│   │   │   │   └── get-lead-query.ts
│   │   │   ├── mutations/
│   │   │   │   ├── create-lead-mutation.ts
│   │   │   │   ├── update-lead-mutation.ts
│   │   │   │   └── delete-lead-mutation.ts
│   │   │   └── actions/
│   │   │       ├── create-lead-action.ts
│   │   │       ├── update-lead-action.ts
│   │   │       └── delete-lead-action.ts
│   │   ├── contacts/
│   │   ├── secure-links/
│   │   └── insights/
│   ├── interceptors/                # HTTP interceptors
│   │   ├── request/                 # Request interceptors
│   │   │   └── append-source.ts
│   │   └── response/                # Response interceptors
│   │       └── error-handler.ts
│   ├── layouts/                     # Page layouts
│   │   ├── auth.vue                 # Auth layout (login pages)
│   │   └── default.vue              # Main app layout
│   ├── models/                      # Domain models
│   │   ├── Lead.ts
│   │   ├── Contact.ts
│   │   ├── SecureLink.ts
│   │   ├── Insight.ts
│   │   └── User.ts
│   ├── pages/                       # File-based routing
│   │   ├── index.vue                # Dashboard/redirect
│   │   ├── profile.vue
│   │   ├── auth/
│   │   │   └── login.vue
│   │   ├── leads/
│   │   │   ├── index.vue            # List view
│   │   │   └── [ulid].vue           # Detail view
│   │   └── contacts/
│   │       ├── index.vue
│   │       └── [ulid].vue
│   ├── plugins/                     # Nuxt plugins
│   │   ├── fetch.ts                 # Register fetch provider
│   │   ├── init.ts                  # Initialize user/app state
│   │   └── session.ts               # Session expiry handling
│   ├── repositories/                # Data access layer
│   │   ├── LeadRepository.ts
│   │   ├── ContactRepository.ts
│   │   └── SecureLinkRepository.ts
│   ├── tables/                      # Table column configurations
│   │   ├── leads.ts
│   │   └── contacts.ts
│   ├── types/                       # TypeScript definitions
│   │   └── index.ts
│   ├── utils/                       # Utility functions
│   │   └── createColumnBuilder.ts
│   ├── values/                      # Value objects
│   │   └── DateValue.ts
│   └── app.vue                      # Root component
├── public/                          # Static assets
├── .env                             # Environment variables
├── nuxt.config.ts                   # Nuxt configuration
├── app.config.ts                    # App configuration
├── package.json
└── tsconfig.json
```

## Directory Purposes

### `/app/components/`

Organized by **UI pattern**, not domain:

| Folder | Purpose | Example |
|--------|---------|---------|
| `Common/` | Shared utilities | `Copyable.vue`, `LoadingLine.vue` |
| `Detail/` | Entity detail views | `LeadDetail.vue`, `ContactDetail.vue` |
| `Form/` | Reusable form inputs | `ContactEmailInput.vue` |
| `Modals/` | Confirmation/action modals | `DeleteLeadModal.vue` |
| `Nav/` | Navigation elements | `UserMenu.vue`, `Sidebar.vue` |
| `Slideovers/` | Slideout panels | `CreateLeadSlideover.vue` |
| `TabSections/` | Tab content sections | `LeadInsightsTab.vue` |
| `Tables/` | Data tables | `LeadsTable.vue` |

### `/app/features/`

Organized by **domain**:

```
features/{domain}/
├── queries/      # Read operations (useFilterQuery)
├── mutations/    # Write operations (API calls)
└── actions/      # Business logic + UI
```

Each domain maps to a model/resource. Feature modules are the primary pattern for new Nuxt apps.

### `/app/constants/`

Static values shared across the app:

```typescript
// channels.ts - WebSocket channels
export const Leads = 'leads'
export const Lead = 'lead.{lead}'

// events.ts - Event names
export const LeadCreated = 'LeadCreated'
export const LeadUpdated = 'LeadUpdated'

// permissions.ts - Permission strings
export const ListLeads = 'leads.list'
export const CreateLead = 'leads.create'

// symbols.ts - Vue injection keys
export const SlideoverKey = Symbol('slideover')
```

## Configuration Files

### `nuxt.config.ts`

```typescript
export default defineNuxtConfig({
  ssr: false,  // SPA mode

  extends: [
    '../../../nuxt-layers/base',
    '../../../nuxt-layers/nuxt-ui',
    '../../../nuxt-layers/x-ui',
  ],

  modules: [
    'nuxt-auth-sanctum',
    '@nuxt/ui',
  ],

  components: [{ path: 'components', pathPrefix: false }],
})
```

### `app.config.ts`

```typescript
export default defineAppConfig({
  repositories: {
    leads: LeadRepository,
    contacts: ContactRepository,
  },

  interceptors: {
    request: [appendSource],
    response: [errorHandler],
  },

  errorHandlers: {
    401: async ({ flash }) => { /* ... */ },
    422: async ({ response }) => new ValidationError(response),
  },
})
```

## Layer Imports

Use layer aliases for base layer imports:

```typescript
// Import from layers
import Model from '#layers/base/app/models/Model'
import type { Castable } from '#layers/base/app/types'
import { BaseRepository } from '#layers/base/app/repositories/base-repository'

// Import from app (use ~ alias)
import Lead from '~/models/Lead'
import { ListLeads } from '~/constants/permissions'
```

## New Feature Checklist

When adding a new domain feature:

1. Create model in `models/`
2. Create enum(s) in `enums/` if needed
3. Create repository in `repositories/`
4. Register repository in `app.config.ts`
5. Create feature module in `features/{domain}/`
   - `queries/get-{domain}s-query.ts`
   - `mutations/create-{domain}-mutation.ts`
   - `actions/create-{domain}-action.ts`
6. Create table config in `tables/`
7. Create components in `components/`
8. Create pages in `pages/`
