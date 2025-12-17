# Lee Overy's Nuxt Application Architecture Guide

> A comprehensive guide to building Nuxt applications following the patterns and conventions established across the Fabric ecosystem.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Directory Structure](#2-directory-structure)
3. [Configuration](#3-configuration)
4. [Models & Data Structures](#4-models--data-structures)
5. [Repository Pattern](#5-repository-pattern)
6. [Feature Module Pattern (Queries/Mutations/Actions)](#6-feature-module-pattern)
7. [Enums & Constants](#7-enums--constants)
8. [Composables](#8-composables)
9. [Components](#9-components)
10. [Pages & Routing](#10-pages--routing)
11. [Plugins & Initialization](#11-plugins--initialization)
12. [Interceptors](#12-interceptors)
13. [Error Handling](#13-error-handling)
14. [Forms](#14-forms)
15. [Tables](#15-tables)
16. [Authentication & Authorization](#16-authentication--authorization)
17. [Real-time Features](#17-real-time-features)
18. [Code Style & Conventions](#18-code-style--conventions)
19. [TypeScript Patterns](#19-typescript-patterns)
20. [Nuxt Layers](#20-nuxt-layers)

---

## 1. Overview

### Philosophy

These applications follow a **domain-driven**, **type-safe**, **composable-first** architecture that prioritizes:

- **Separation of Concerns**: Clear boundaries between data access, business logic, and presentation
- **Type Safety**: Full TypeScript coverage with explicit types everywhere
- **Reusability**: Composables and base classes for DRY code
- **Scalability**: Feature-based organization for large applications
- **Developer Experience**: Auto-imports, clear conventions, predictable patterns

### Technology Stack

| Layer | Technology |
|-------|------------|
| Framework | Nuxt 4 (with SSR disabled - SPA mode) |
| UI Framework | Vue 3 with Composition API |
| Component Library | Nuxt UI (v4) with Tailwind CSS 4 |
| State Management | Composables with `ref`/`useState` |
| HTTP Client | Custom fetch via Sanctum/ofetch |
| Authentication | Laravel Sanctum (`nuxt-auth-sanctum`) |
| Real-time | Laravel Echo (`nuxt-laravel-echo`) |
| Icons | Heroicons + Lucide via Iconify |

### Project References

| Project | Path | Version | Purpose |
|---------|------|---------|---------|
| Teri for Ops | `/fabric/teri/teri/javascript/apps/teri-for-ops` | Nuxt 4 | Lead management, chat ops |
| FlowX App | `/fabric/flowx/flowx-app` | Nuxt 3→4 | Pension tracing system |
| Nuxt Layers | `/fabric/nuxt-layers` | v4 (main) | Shared foundation |

---

## 2. Directory Structure

### Standard Project Layout

```
project-root/
├── app/                              # Main application code
│   ├── actions/                      # Business logic with UI feedback (older pattern)
│   ├── assets/
│   │   ├── css/main.css             # Global styles
│   │   └── images/
│   ├── components/                   # Vue components (organized by type)
│   │   ├── Common/                  # Shared/generic components
│   │   ├── Detail/                  # Detail view components
│   │   ├── Form/                    # Form input components
│   │   ├── Modals/                  # Modal dialogs
│   │   ├── Nav/                     # Navigation components
│   │   ├── Slideovers/              # Slideout panels
│   │   ├── TabSections/             # Tab content components
│   │   └── Tables/                  # Table components
│   ├── composables/                 # Custom Vue composables
│   ├── constants/                   # App-wide constants
│   │   ├── channels.ts              # WebSocket channel names
│   │   ├── events.ts                # Event names
│   │   ├── permissions.ts           # Permission strings
│   │   └── symbols.ts               # Vue injection symbols
│   ├── enums/                       # TypeScript enums with behavior
│   ├── errors/                      # Custom error classes
│   ├── features/                    # Domain-based feature modules (newer pattern)
│   │   └── {domain}/
│   │       ├── queries/             # Data fetching
│   │       ├── mutations/           # Pure data operations
│   │       └── actions/             # Business logic + UI
│   ├── interceptors/                # HTTP interceptors
│   │   ├── request/                 # Request interceptors
│   │   └── response/                # Response interceptors
│   ├── layouts/                     # Page layouts
│   │   ├── auth.vue                 # Auth layout
│   │   └── default.vue              # Main layout
│   ├── models/                      # Domain models
│   ├── pages/                       # File-based routing
│   ├── plugins/                     # Nuxt plugins
│   ├── repositories/                # Data access layer
│   ├── tables/                      # Table column configurations
│   ├── types/                       # TypeScript definitions
│   ├── utils/                       # Utility functions
│   ├── values/                      # Value objects
│   └── app.vue                      # Root component
├── public/                          # Static assets
├── .env                             # Environment variables
├── .env-live                        # Production environment
├── .env-local                       # Local overrides
├── nuxt.config.ts                   # Nuxt configuration
├── package.json                     # Dependencies
├── tsconfig.json                    # TypeScript config
└── eslint.config.mjs                # Linting rules
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Models | PascalCase | `Lead.ts`, `SecureLink.ts` |
| Repositories | PascalCase + "Repository" | `LeadRepository.ts` |
| Composables | camelCase, "use" prefix | `useUser.ts`, `useFeedbackCategories.ts` |
| Actions/Mutations/Queries | kebab-case + suffix | `create-lead-action.ts` |
| Components | PascalCase | `CreateLeadSlideover.vue` |
| Enums | PascalCase | `LeadStatus.ts` |
| Constants | kebab-case files, camelCase exports | `permissions.ts` |
| Pages | kebab-case | `eval-feedback/index.vue` |

---

## 3. Configuration

### nuxt.config.ts

```typescript
// Example: teri-for-ops/nuxt.config.ts
export default defineNuxtConfig({
  // SPA mode - SSR disabled
  ssr: false,

  // Extend shared layers
  extends: [
    '../../../nuxt-layers/base',
    '../../../nuxt-layers/nuxt-ui',
    '../../../nuxt-layers/x-ui',
  ],

  // Compatibility
  compatibilityDate: '2025-07-15',

  // Modules
  modules: [
    'nuxt-auth-sanctum',
    'reka-ui/nuxt',
    '@nuxt/ui',
  ],

  // Dev tools
  devtools: { enabled: true },

  // Component auto-import without path prefix
  components: [{ path: 'components', pathPrefix: false }],

  // Sanctum authentication
  sanctum: {
    baseUrl: process.env.NUXT_PUBLIC_API_URL,
    endpoints: {
      login: '/chat/login',
      user: '/chat/profile',
      csrf: '/chat/csrf-cookie',
      logout: '/chat/logout',
    },
    csrf: {
      cookie: 'CHAT-XSRF-TOKEN',
      header: 'X-XSRF-TOKEN',
    },
    redirect: {
      onAuthOnly: '/auth/login',
      onGuestOnly: '/',
      onLogin: '/',
      onLogout: '/auth/login',
    },
  },

  // Icon configuration
  icon: {
    serverBundle: 'remote',
    clientBundle: {
      scan: true,
      includeCustomCollections: true,
    },
    customCollections: [
      {
        prefix: 'custom',
        dir: './app/assets/icons',
      },
    ],
  },

  // Runtime configuration
  runtimeConfig: {
    public: {
      // API URLs
      apiUrl: undefined,

      // Repository-specific configurations
      repositories: {
        leads: { fetchOptions: { baseURL: undefined } },
        contacts: { fetchOptions: { baseURL: undefined } },
        // ... more repositories
      },

      // External service URLs
      chatService: { urls: { api: undefined } },
      objectiveAgent: { urls: { app: undefined } },
    },
  },

  // UI configuration
  ui: {
    colorMode: false,
    theme: {
      colors: ['primary', 'secondary', 'success', 'info', 'warning', 'error', 'neutral'],
    },
  },

  // Tailwind CSS
  css: ['~/assets/css/main.css'],
  vite: {
    plugins: [tailwindcss()],
  },
});
```

### Environment Variables

```bash
# API Configuration
NUXT_PUBLIC_API_URL=https://api.example.com

# Per-repository base URLs (override global)
NUXT_PUBLIC_REPOSITORIES_LEADS_FETCH_OPTIONS_BASE_URL=https://leads-api.example.com
NUXT_PUBLIC_REPOSITORIES_CONTACTS_FETCH_OPTIONS_BASE_URL=https://contacts-api.example.com

# External services
NUXT_PUBLIC_CHAT_SERVICE_URLS_API=https://chat.example.com
NUXT_PUBLIC_OBJECTIVE_AGENT_URLS_APP=https://agent.example.com

# WebSocket (Laravel Echo)
NUXT_PUBLIC_ECHO_KEY=your-pusher-key
NUXT_PUBLIC_ECHO_HOST=echo.example.com
NUXT_PUBLIC_ECHO_SCHEME=https
NUXT_PUBLIC_ECHO_PORT=443
```

### tsconfig.json

```json
{
  "extends": "./.nuxt/tsconfig.json",
  "compilerOptions": {
    "strictPropertyInitialization": false
  }
}
```

### eslint.config.mjs

```javascript
import withNuxt from './.nuxt/eslint.config.mjs'

export default withNuxt({
  rules: {
    'vue/multi-word-component-names': 'off',
    'vue/no-v-html': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'import/order': 'off', // Prevent script block reordering
  },
})
```

---

## 4. Models & Data Structures

### Base Model Class

All models extend the base `Model` class from the nuxt-layers/base layer:

```typescript
// #layers/base/app/models/Model.ts
export default class Model<T = any> {
  constructor() { /* NOOP */ }

  // Static factory methods
  public static instance<T, M extends typeof Model>(data: T): InstanceType<M>
  public static hydrate<T, M extends typeof Model>(data: T): InstanceType<M>
  public static collect<T, M extends typeof Model>(collection: T[]): InstanceType<M>[]

  // Override in subclasses
  public transform<D>(data: D): D { return data }
  public relations(): Record<string, Model> | {} { return {} }
  public casts(): Record<string, Castable> | {} { return {} }
  public strictProperties(): boolean { return true }
  public primaryKey(): string { return 'id' }

  // Lifecycle hooks
  public booting(): void {}
  public booted(): void {}

  // Instance methods
  public is(model: Model): boolean
  public clone(): this
}
```

### Model Lifecycle

```
API Response Data
       ↓
booting() hook
       ↓
transform() - Transform raw data (e.g., camelCase conversion)
       ↓
Property Assignment (strict or loose based on strictProperties())
       ↓
applyRelations() - Hydrate nested models
       ↓
applyCasts() - Cast properties to types (enums, dates, etc.)
       ↓
booted() hook
       ↓
Model Instance Ready
```

### Creating Models

```typescript
// app/models/Lead.ts
import Model from '#layers/base/app/models/Model'
import type { Castable } from '#layers/base/app/types'
import LeadStatus from '~/enums/LeadStatus'
import DateValue from '~/values/DateValue'
import Contact from '~/models/Contact'
import SecureLink from '~/models/SecureLink'
import Insight from '~/models/Insight'
import Conversation from '~/models/Conversation'

export default class Lead extends Model {
  // Properties with types
  ulid: string
  status: LeadStatus
  demand: string
  callScheduledAt: DateValue
  testFlag: boolean

  // Relations (hydrated automatically)
  contact: Contact
  insights?: Insight[]
  insightsCount?: number
  secureLinks?: SecureLink[]
  secureLinksCount?: number
  conversations?: Conversation[]
  conversationsCount?: number

  // Timestamps
  createdAt: DateValue
  updatedAt: DateValue

  // Custom primary key (default is 'id')
  public override primaryKey(): string {
    return 'ulid'
  }

  // Define type casting
  public override casts(): Record<string, Castable> {
    return {
      status: LeadStatus,
      callScheduledAt: DateValue,
      createdAt: DateValue,
      updatedAt: DateValue,
    }
  }

  // Define nested model relations
  public override relations(): Record<string, typeof Model> {
    return {
      contact: Contact,
      insights: Insight,
      secureLinks: SecureLink,
      conversations: Conversation,
    }
  }

  // Instance methods
  public isTest(): boolean {
    return this.testFlag
  }

  public hasSecureLinks(): boolean {
    return (this.secureLinksCount ?? 0) > 0
  }
}
```

### Model with Data Transformation

```typescript
// app/models/User.ts
import Model from '#layers/base/app/models/Model'
import { transformKeys } from '#layers/base/app/utils'
import { camelCase } from 'change-case'

export default class User extends Model {
  uuid: string
  sessionId: string
  conversationId: string
  name: string
  email: string
  active: boolean
  testFlag: boolean
  personalAccessToken?: string
  createdAt: string
  updatedAt: string

  // Transform snake_case API response to camelCase
  public override transform<D>(data: D): D {
    return transformKeys(data, camelCase)
  }

  public override booted() {
    // Register permissions after hydration
    if (this.permissions) {
      registerPermissions(this.permissions)
    }
  }
}
```

### Value Objects

Value objects encapsulate complex values with behavior:

```typescript
// app/values/DateValue.ts
import type { Castable } from '#layers/base/app/types'

export default class DateValue implements Castable {
  iso8601: string

  constructor(iso8601: string) {
    this.iso8601 = iso8601
  }

  // Format using dayjs (auto-imported from layer)
  format(format: string = 'DD MMM YYYY'): string | undefined {
    if (!this.iso8601) return undefined
    return useDayjs()(this.iso8601).format(format)
  }

  toDate(): Date {
    return new Date(this.iso8601)
  }

  // Static cast method for Model casting system
  static cast(value: string): DateValue {
    return new DateValue(value)
  }
}
```

### Model Usage Examples

```typescript
// Hydrate single model from API response
const lead = Lead.hydrate(apiResponse.data)

// Hydrate collection
const leads = Lead.collect(apiResponse.data)

// Create instance (same as hydrate)
const lead = Lead.instance({ ulid: '...', status: 'new lead' })

// Access typed properties
lead.status              // LeadStatus enum instance
lead.status.color()      // 'primary' (color from enum method)
lead.createdAt.format()  // 'DD MMM YYYY' formatted string
lead.contact.name        // Nested model property

// Compare models
lead.is(otherLead)       // true if same primary key

// Clone
const copy = lead.clone()
```

---

## 5. Repository Pattern

### Base Repository

All repositories extend `BaseRepository<T>` from the base layer:

```typescript
// #layers/base/app/repositories/base-repository.ts
export abstract class BaseRepository<T = any> extends ApiClient {
  protected abstract resource: string
  protected hydration: boolean = false
  protected hydrator: undefined | Hydrator = undefined

  // CRUD operations
  public async list(params?: GenericQueryParams): Promise<CollectionResponse<T> | PaginatedResponse<T>>
  public async get(id: number | string, params?): Promise<DataResponse<T>>
  public async create(data?): Promise<DataResponse<T>>
  public async update(id: number | string, data): Promise<DataResponse<T>>
  public async delete(id: number | string): Promise<void>

  // Hydration control
  public async withHydration<R>(callback: (repo: this) => Promise<R>): Promise<R>
  public async withoutHydration<R>(callback: (repo: this) => Promise<R>): Promise<R>
}
```

### Creating Repositories

```typescript
// app/repositories/LeadRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Lead from '~/models/Lead'
import type { CollectionResponse, GenericQueryParams, OutgoingQueryParams } from '#layers/base/app/types'

export default class LeadRepository extends BaseRepository<Lead> {
  // API resource path
  protected resource = '/lead-management/leads'

  // Enable automatic model hydration
  protected hydration = true
  protected hydrator = new ModelHydrator(Lead)

  // Custom methods for domain-specific operations
  async listByContact(
    contactUlid: string,
    params?: GenericQueryParams & OutgoingQueryParams
  ): Promise<CollectionResponse<Lead>> {
    return this.jsonGet(`/lead-management/contacts/${contactUlid}/leads`, params)
  }
}
```

```typescript
// app/repositories/SecureLinkRepository.ts
export default class SecureLinkRepository extends BaseRepository<SecureLink> {
  protected resource = '/lead-management/secure-links'
  protected hydration = true
  protected hydrator = new ModelHydrator(SecureLink)

  async createForLead(leadUlid: string): Promise<DataResponse<SecureLink>> {
    return this.jsonPost(`/lead-management/leads/${leadUlid}/secure-links`)
  }

  async resend(secureLink: SecureLink): Promise<DataResponse<SecureLink>> {
    return this.jsonPost(`${this.resource}/${secureLink.ulid}/resend`)
  }
}
```

### Registering Repositories

```typescript
// app/app.config.ts
import LeadRepository from '~/repositories/LeadRepository'
import ContactRepository from '~/repositories/ContactRepository'
import SecureLinkRepository from '~/repositories/SecureLinkRepository'

export default defineAppConfig({
  repositories: {
    // Simple format (uses global apiUrl)
    leads: LeadRepository,
    contacts: ContactRepository,
    secureLinks: SecureLinkRepository,

    // Extended format with custom fetch options
    externalService: {
      repository: ExternalRepository,
      fetchOptions: {
        baseURL: 'https://external-api.example.com',
        timeout: 30000,
        headers: { 'X-Service': 'external' },
      },
    },
  },
})
```

### Using Repositories

```typescript
// Get typed repository instance
const leadApi = useRepository('leads')  // Returns LeadRepository

// Basic CRUD
const { data: leads } = await leadApi.list()
const { data: lead } = await leadApi.get('ulid123')
const { data: newLead } = await leadApi.create({ name: 'John', email: 'john@example.com' })
await leadApi.update('ulid123', { name: 'Jane' })
await leadApi.delete('ulid123')

// With query parameters
const { data: leads } = await leadApi.list({
  include: 'contact,secure-links',
  filter: { status: 'new lead' },
  page: { number: 1, size: 25 },
})

// Custom methods
const { data: contactLeads } = await leadApi.listByContact('contact-ulid')

// Hydration control
const rawData = await leadApi.withoutHydration(async (repo) => {
  return repo.list()
})
```

---

## 6. Feature Module Pattern

### Overview

The feature module pattern (used in teri-for-ops) organizes domain logic into three layers:

```
features/{domain}/
├── queries/      # Data fetching with reactive filters
├── mutations/    # Pure data operations (API calls)
└── actions/      # Business logic with UI feedback
```

**Flow**: Component → Action → Mutation → Repository → API

### Queries (Data Fetching)

Queries wrap `useQuery` or `useFilterQuery` composables for reactive data fetching:

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

// Factory function returning query composable
export default function getLeadsQueryFactory() {
  const leadApi = useRepository('leads')

  return (filters: MaybeRef<GetLeadsFilters>) => {
    return useFilterQuery('leads', () => {
      // Build query params with JSON:API spec
      const params = useJsonSpec()
        .filters(filters, KebabCase)           // Convert filter keys to kebab-case
        .include('secure-links', 'contact')     // Include relations
        .build()

      return leadApi.list(params)
    }, filters)
  }
}
```

```typescript
// app/features/leads/queries/get-lead-query.ts
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

**Query Usage in Components**:

```typescript
// In page or component
const { filters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
})

const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh, isLoading, isFetching, pagination } = getLeadsQuery(filters)

// Data automatically refetches when filters change
```

### Mutations (Pure Data Operations)

Mutations handle API operations without UI concerns:

```typescript
// app/features/leads/mutations/create-lead-mutation.ts
export type CreateLeadData = {
  name: string
  email: string
  demand: string
  callScheduledAt: string
  sendLoginLink: boolean
  testFlag: boolean
}

export default function createLeadMutationFactory() {
  const leadApi = useRepository('leads')
  const { start, stop, waitingFor } = useWait()

  return async (data: CreateLeadData): Promise<Lead> => {
    try {
      start(waitingFor.leads.creating)
      const { data: lead } = await leadApi.create(data)
      return lead
    } finally {
      stop(waitingFor.leads.creating)
    }
  }
}
```

```typescript
// app/features/secure-links/mutations/create-secure-link-for-lead-mutation.ts
export default function createSecureLinkForLeadMutationFactory() {
  const secureLinkApi = useRepository('secureLinks')
  const { start, stop, waitingFor } = useWait()

  return async (leadUlid: string): Promise<SecureLink> => {
    try {
      start(waitingFor.secureLinks.creating)
      const { data: secureLink } = await secureLinkApi.createForLead(leadUlid)
      return secureLink
    } finally {
      stop(waitingFor.secureLinks.creating)
    }
  }
}
```

**Mutation Responsibilities**:
- Pure data operations (no UI awareness)
- Use repositories for API calls
- Manage loading states via `useWait`
- Define TypeScript data types
- Throw errors (let actions handle them)

### Actions (Business Logic + UI)

Actions orchestrate mutations and provide user feedback:

```typescript
// app/features/leads/actions/create-lead-action.ts
import createLeadMutationFactory, { type CreateLeadData } from '../mutations/create-lead-mutation'

export default function createLeadActionFactory() {
  const createLead = createLeadMutationFactory()
  const { copy, copied } = useClipboard()
  const flash = useFlash()
  const { trigger } = useConfirmationToast()
  const { handleActionError } = useHandleActionError()

  return async (data: CreateLeadData): Promise<Lead> => {
    try {
      const lead = await createLead(data)
      flash.success('Lead created successfully.')

      // Show confirmation with link if secure link was created
      if (lead.hasSecureLinks()) {
        const linkUrl = lead.secureLinks![0]!.fullUrl()
        trigger({
          title: 'The following link was sent to the customer',
          description: linkUrl,
          confirmLabel: 'Copy link',
          onConfirm: async () => {
            await copy(linkUrl)
            flash.success('Link copied to clipboard')
          },
        })
      }

      return lead
    } catch (error) {
      throw handleActionError(error, {
        entity: 'lead',
        operation: 'create',
      })
    }
  }
}
```

```typescript
// app/features/leads/actions/delete-lead-action.ts
import deleteLeadMutationFactory from '../mutations/delete-lead-mutation'

export default function deleteLeadActionFactory() {
  const deleteLead = deleteLeadMutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (lead: Lead): Promise<void> => {
    try {
      await deleteLead(lead.ulid)
      flash.success('Lead deleted successfully.')
    } catch (error) {
      throw handleActionError(error, {
        entity: 'lead',
        operation: 'delete',
      })
    }
  }
}
```

**Action Responsibilities**:
- Orchestrate one or more mutations
- UI-aware: Handle flash messages, confirmations, toasts
- Complex business logic (bulk operations, workflows)
- Error handling via `useHandleActionError`
- User-friendly feedback

### Using Feature Modules in Components

```vue
<script lang="ts" setup>
// Import action factory
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'
import type { CreateLeadData } from '~/features/leads/mutations/create-lead-mutation'

// Create action instance
const createLeadAction = createLeadActionFactory()

// Form data
const formData = ref<CreateLeadData>({
  email: '',
  name: '',
  demand: '',
  callScheduledAt: '',
  testFlag: false,
  sendLoginLink: true,
})

// Handle form submission
const onSubmit = async (data: CreateLeadData) => {
  const lead = await createLeadAction(data)
  // Action already shows success message
  emits('close', true)
}
</script>
```

---

## 7. Enums & Constants

### Enum Pattern

Enums extend a base `Enum` class and implement `Castable` for model integration:

```typescript
// app/enums/LeadStatus.ts
import Enum from '#layers/base/app/enums/Enum'
import type { Castable } from '#layers/base/app/types'

export default class LeadStatus extends Enum implements Castable {
  // Static enum values
  static readonly NewLead = LeadStatus.create('new lead')
  static readonly LinkSent = LeadStatus.create('link sent')
  static readonly TranscriptReceived = LeadStatus.create('transcript received')
  static readonly Completed = LeadStatus.create('completed')
  static readonly Cancelled = LeadStatus.create('cancelled')

  // Cast method for model system
  static cast(value: string | EnumStoreObject): LeadStatus {
    return LeadStatus.coerce(value)
  }

  // Behavior methods
  color(): string {
    switch (this.value) {
      case 'new lead':
        return 'primary'
      case 'link sent':
        return 'info'
      case 'transcript received':
        return 'warning'
      case 'completed':
        return 'success'
      case 'cancelled':
        return 'error'
      default:
        return 'neutral'
    }
  }

  // Text representation
  get text(): string {
    return this.value
  }
}
```

### Enum Usage

```typescript
// In models (via casts)
status: LeadStatus  // Automatically cast from API string

// In templates
<UBadge :color="lead.status.color()">
  {{ lead.status.text }}
</UBadge>

// Comparisons
if (lead.status.is(LeadStatus.Completed)) { ... }

// Get all values
LeadStatus.values()  // [NewLead, LinkSent, ...]
```

### Constants Organization

```typescript
// app/constants/permissions.ts
// Auto-generated from Laravel backend
export const ListLeads = 'leads.list'
export const ShowLead = 'leads.show'
export const CreateLead = 'leads.create'
export const UpdateLead = 'leads.update'
export const DeleteLead = 'leads.delete'

// Permission groups
export const LeadPermissions = [
  ListLeads,
  ShowLead,
  CreateLead,
  UpdateLead,
  DeleteLead,
]
```

```typescript
// app/constants/channels.ts
// WebSocket channel names
export const Leads = 'leads'
export const Lead = 'lead.{lead}'
export const Contacts = 'contacts'
```

```typescript
// app/constants/events.ts
// Event names for real-time updates
export const LeadCreated = 'LeadCreated'
export const LeadUpdated = 'LeadUpdated'
export const LeadDeleted = 'LeadDeleted'
export const SecureLinkSent = 'SecureLinkSent'
```

```typescript
// app/constants/symbols.ts
// Vue injection symbols
export const SlideoverKey = Symbol('slideover')
export const ModalKey = Symbol('modal')
export const ReactiveFiltersKey = Symbol('reactive-filters')
```

---

## 8. Composables

### Core Composables (from Base Layer)

| Composable | Purpose |
|------------|---------|
| `useRepository(name)` | Get typed repository instance |
| `useQuery(key, fetcher, options)` | Cached data fetching |
| `useFilterQuery(key, fetcher, filters)` | Reactive filtered data fetching |
| `useWait()` | Global loading state management |
| `useFlash()` | Toast notification system |
| `usePermissions()` | Permission checking |
| `useReactiveFilters(defaults, options)` | URL-synced reactive filters |
| `useForm(url, method, data, options)` | Form state management |
| `useFormBuilder()` | Fluent form configuration |
| `useErrorHandler()` | Centralized error handling |
| `useRealtime()` | WebSocket channel subscriptions |
| `useShadowCache()` | In-memory caching |
| `useCreateContext()` | Typed Vue context creation |

### App-Specific Composables

```typescript
// app/composables/useUser.ts
let user = ref<User>()

export default function useUser() {
  const getUser = () => user

  const setUser = (userData: BaseEntity) => {
    user.value = User.hydrate(userData)
  }

  const clearUser = () => {
    user.value = undefined
  }

  return {
    user,
    getUser,
    setUser,
    clearUser,
  }
}
```

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
      // Supports nested path access: "category.subcategory.type"
    })
  }

  return { getCategoryTree, builtTree }
}
```

```typescript
// app/composables/useHandleActionError.ts
export function useHandleActionError() {
  const flash = useFlash()

  const handleActionError = (error: unknown, context: { entity: string; operation: string }) => {
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

### useWait Pattern

```typescript
// Global loading state management
const { start, stop, is, waitingFor } = useWait()

// Define waiting states
waitingFor.leads.creating     // Boolean ref for "creating leads"
waitingFor.lead.loading(id)   // Boolean ref for specific lead loading
waitingFor.lead.deleting(id)  // Boolean ref for specific lead deletion

// In mutations
start(waitingFor.leads.creating)
// ... do work
stop(waitingFor.leads.creating)

// In templates
const isCreating = is(waitingFor.leads.creating)
<UButton :loading="isCreating">Create Lead</UButton>
```

### useReactiveFilters Pattern

```typescript
// In page component
const { filters, hasFilters, resetFilters, resetFilter } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
  page: 1,
  size: 25,
}, {
  syncWithUrl: true,              // Sync filters to URL query params
  neverResetOn: ['page'],         // Don't reset page on filter changes
  debounceUrlUpdate: 300,         // Debounce URL updates
  provideAs: 'LeadFilters',       // Provide to children
})

// Filters are reactive - changes trigger query refetch
filters.status = 'new lead'

// Check if any filters are active
if (hasFilters.value) {
  // Show "Clear filters" button
}

// Reset specific filter
resetFilter('status')

// Reset all filters
resetFilters()
```

---

## 9. Components

### Component Organization

```
components/
├── Common/              # Shared generic components
│   └── TeriLogo.vue
├── Detail/              # Detail view components
│   ├── LeadDetail.vue
│   └── ContactDetail.vue
├── Form/                # Form input components
│   ├── ContactEmailInput.vue
│   └── LeadDemandRow.vue
├── Modals/              # Modal dialogs
│   ├── DeleteLeadModal.vue
│   ├── DeleteContactModal.vue
│   └── CreateSecureLinkModal.vue
├── Nav/                 # Navigation components
│   └── UserMenu.vue
├── Slideovers/          # Slideout panels
│   ├── CreateLeadSlideover.vue
│   └── UpdateLeadSlideover.vue
├── TabSections/         # Tab content components
│   ├── LeadInsightsTab.vue
│   └── ContactLeadsTab.vue
├── Tables/              # Table components
│   ├── LeadsTable.vue
│   └── ContactsTable.vue
└── Feedback/            # Domain-specific components
    └── CategoryRatingField.vue
```

### Script Setup Order Convention

Follow this order in `<script setup>`:

```vue
<script lang="ts" setup>
// 1. Imports
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'

// 2. Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits(['close'])

// 3. Composables and definitions
const flash = useFlash()
const { filters } = useReactiveFilters()

// 4. Injections
const slideover = inject(SlideoverKey)

// 5. Refs
const formRef = useTemplateRef('formRef')
const existingContact = ref<Contact>()

// 6. Toggles (boolean refs)
const isOpen = ref(false)

// 7. Reactive props
const formData = ref<CreateLeadData>({
  email: '',
  name: '',
})

// 8. Computed props
const canSubmit = computed(() => formData.value.email && formData.value.name)

// 9. Fetch + associated calls
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh } = getLeadsQuery(filters)

// 10. Builders
const createLeadAction = createLeadActionFactory()

// 11. Watchers
watch(existingContact, (contact) => {
  formData.value.name = contact?.name || ''
})

// 12. Methods
const onSubmit = async (data: CreateLeadData) => {
  await createLeadAction(data)
  emits('close', true)
}

// 13. Real-time listeners
const { privateChannel } = useRealtime()
privateChannel(Leads).on(LeadCreated, refresh)

// 14. Provides
provide(SlideoverKey, { isOpen })

// 15. Lifecycles
onMounted(() => { /* ... */ })
</script>
```

### Component Example: Slideover

```vue
<!-- app/components/Slideovers/CreateLeadSlideover.vue -->
<script lang="ts" setup>
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'
import type { CreateLeadData } from '~/features/leads/mutations/create-lead-mutation'

// Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits(['close'])

// Refs
const formRef = useTemplateRef('formRef')
const existingContact = ref<Contact>()

// Form data
const formData = ref<CreateLeadData>({
  email: props.contact?.email || '',
  name: props.contact?.name || '',
  demand: '',
  callScheduledAt: '',
  testFlag: false,
  sendLoginLink: true,
})

// Watchers
watch(existingContact, (contact) => {
  formData.value.name = contact?.name || ''
})

// Methods
const createLeadAction = createLeadActionFactory()

const onSubmit = (data: CreateLeadData) => createLeadAction(data)
const onSuccess = () => emits('close', true)
</script>

<template>
  <USlideover title="Create Lead">
    <XForm
      ref="formRef"
      url="/lead-management/leads"
      method="POST"
      :data="formData"
      @submit="onSubmit"
      @success="onSuccess"
    >
      <div class="space-y-4">
        <ContactEmailInput
          v-model:email="formData.email"
          v-model:contact="existingContact"
        />

        <UFormField label="Name" name="name">
          <UInput v-model="formData.name" />
        </UFormField>

        <UFormField label="Demand" name="demand">
          <UTextarea v-model="formData.demand" />
        </UFormField>

        <UCheckbox v-model="formData.testFlag" label="Test lead" />
      </div>

      <template #actions>
        <UButton type="submit" label="Create Lead" />
      </template>
    </XForm>
  </USlideover>
</template>
```

### Component Example: Table

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

const emits = defineEmits(['edit', 'delete'])

// Get columns from builder
const columns = leadsColumnBuilder.all()

// Row actions
const rowActions = computed(() => (row: Row<Lead>) => [
  { label: 'View lead', to: `/leads/${row.original.ulid}` },
  { label: 'View contact', to: `/contacts/${row.original.contact.ulid}` },
  { label: 'Edit lead', onSelect: () => emits('edit', row.original) },
  { label: 'Delete lead', onSelect: () => emits('delete', [row.original]) },
])
</script>

<template>
  <XTable
    :data="leads"
    :columns="columns"
    :loading="loading"
    :fetching="fetching"
    :row-actions="rowActions"
    row-id="ulid"
  />
</template>
```

---

## 10. Pages & Routing

### Page Structure

```
pages/
├── index.vue                    # Dashboard/redirect
├── profile.vue                  # User profile
├── settings.vue                 # Settings
├── auth/
│   └── login.vue               # Login page
├── leads/
│   ├── index.vue               # Leads list with table
│   └── [ulid].vue              # Lead detail with tabs
├── contacts/
│   ├── index.vue               # Contacts list
│   └── [ulid].vue              # Contact detail
└── eval-feedback/
    ├── index.vue               # Evaluations list
    └── [uuid]/
        └── review.vue          # Evaluation review
```

### Page Pattern: List with Filters

```vue
<!-- app/pages/leads/index.vue -->
<script lang="ts" setup>
import getLeadsQueryFactory, { type GetLeadsFilters } from '~/features/leads/queries/get-leads-query'
import deleteLeadActionFactory from '~/features/leads/actions/delete-lead-action'
import { ListLeads, CreateLead } from '~/constants/permissions'

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
setBreadcrumbs([{ label: 'Lead Management' }, { label: 'Leads' }])

// Reactive filters
const { filters } = useReactiveFilters<GetLeadsFilters>({
  status: undefined,
  testFlag: undefined,
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
const deleteLeads = async (leads: Lead[]) => {
  for (const lead of leads) {
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
    <!-- Filters -->
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
        v-if="can(CreateLead)"
        label="Create Lead"
        @click="openCreateSlideover()"
      />
    </div>

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
    />

    <!-- Slideovers -->
    <CreateLeadSlideover @close="refresh" />
    <UpdateLeadSlideover @close="refresh" />
  </div>
</template>
```

### Page Pattern: Detail with Tabs

```vue
<!-- app/pages/leads/[ulid].vue -->
<script lang="ts" setup>
import getLeadQueryFactory from '~/features/leads/queries/get-lead-query'
import { ShowLead } from '~/constants/permissions'

definePageMeta({
  permissions: ShowLead,
})

const route = useRoute()
const ulid = computed(() => route.params.ulid as string)

// Query
const getLeadQuery = getLeadQueryFactory()
const { data: lead, isLoading } = getLeadQuery(ulid)

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

// Tabs
const tabs = [
  { label: 'Details', slot: 'details' },
  { label: 'Secure Links', slot: 'secure-links', badge: lead.value?.data.secureLinksCount },
  { label: 'Insights', slot: 'insights', badge: lead.value?.data.insightsCount },
  { label: 'Conversations', slot: 'conversations', badge: lead.value?.data.conversationsCount },
]
</script>

<template>
  <div v-if="!isLoading && lead">
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
</template>
```

---

## 11. Plugins & Initialization

### Plugin Execution Order

Plugins execute in alphabetical order by filename:

1. `fetch.ts` - Register fetch provider
2. `init.ts` - Initialize user and app state
3. `session.ts` - Session expiry handling

### fetch.ts - Fetch Provider

```typescript
// app/plugins/fetch.ts
export default defineNuxtPlugin({
  async setup() {
    // Register Sanctum client as fetch provider
    registerFetchProvider(10, () => useSanctumClient())
  },
})
```

### init.ts - User Initialization

```typescript
// app/plugins/init.ts
export default defineNuxtPlugin(async (nuxtApp) => {
  const { init: fetchUser, user: sanctumUser, isAuthenticated } = useSanctumAuth()
  const { setUser, clearUser } = useUser()
  const { startSessionWatcher, stopSessionWatcher } = useSessionWatcher()

  const initUser = async () => {
    try {
      await fetchUser()
      setUser(sanctumUser.value!.data)
      startSessionWatcher()

      // Set tokens for external services if needed
      setPersonalAccessToken(getUser().value?.personalAccessToken)
    } catch (error) {
      // Allowed to fail - auth middleware will redirect
    }
  }

  // Watch for logout
  watch(isAuthenticated, (newValue, oldValue) => {
    if (newValue === false && oldValue === true) {
      clearUser()
      closeAllSlideovers()
      stopSessionWatcher()
    }
  })

  // Handle login event
  nuxtApp.hook('sanctum:login', async () => {
    await initUser()
  })

  // Initial load
  await initUser()
})
```

### session.ts - Session Expiry

```typescript
// app/plugins/session.ts
export default defineNuxtPlugin((nuxtApp) => {
  const { trigger } = useConfirmationToast()
  const flash = useFlash()

  // Pre-expiry warning
  nuxtApp.hook('session-watcher:pre-expire', async () => {
    trigger({
      title: 'Session Expiring',
      description: 'Your session will expire soon. Stay logged in?',
      confirmLabel: 'Stay Logged In',
      cancelLabel: 'Logout',
      onConfirm: () => refreshSession(),
      onCancel: () => logout(),
    })
  })

  // Session expired
  nuxtApp.hook('session-watcher:expire', async () => {
    flash.info('Session expired', 'You have been logged out.')
    await logout()
  })
})
```

---

## 12. Interceptors

### Request Interceptors

```typescript
// app/interceptors/request/append-source.ts
import type { NuxtApp } from '#app'
import type { FetchContext } from 'ofetch'
import type { ConsolaInstance } from 'consola'
import type { InterceptorResult } from '#layers/base/app/types'

export default async function sourceTrackerRequestInterceptor(
  app: NuxtApp,
  ctx: FetchContext,
  logger: ConsolaInstance
): Promise<InterceptorResult> {
  const requestUrl = typeof ctx.request === 'string'
    ? ctx.request
    : ctx.request.toString()

  // Parse URL
  let url = new URL(
    requestUrl.startsWith('http') ? requestUrl : requestUrl,
    'http://localhost'
  )

  // Add source parameter
  if (!url.searchParams.has('source')) {
    url.searchParams.append('source', 't4o')
    ctx.request = requestUrl.startsWith('http')
      ? url.toString()
      : url.pathname + url.search
  }
}
```

### Response Interceptors

```typescript
// app/interceptors/response/error-handler.ts
import type { NuxtApp } from '#app'
import type { FetchContext } from 'ofetch'

export default async function handleResponseErrors(
  app: NuxtApp,
  { response }: FetchContext
): Promise<InterceptorResult> {
  const { handleError } = useErrorHandler()

  if (response) {
    return handleError(response)
  }
}
```

### Registering Interceptors

```typescript
// app/app.config.ts
import appendSource from '~/interceptors/request/append-source'
import errorHandler from '~/interceptors/response/error-handler'

export default defineAppConfig({
  interceptors: {
    request: [appendSource],
    response: [errorHandler],
  },
})
```

---

## 13. Error Handling

### Error Classes

```typescript
// From base layer: #layers/base/app/errors/

// Base error class
export class ErrorResponse<T = any> {
  constructor(
    public response: FetchResponse<T>,
    public status: number
  ) {}
}

// Validation errors (422)
export class ValidationError<T = any> extends ErrorResponse<T> {
  public errors: Record<string, string[]>
  public message: string

  mapToFormErrors(): FormError[]
}

// Other error classes
export class ConflictError extends ErrorResponse {}        // 409
export class TooManyRequestsError extends ErrorResponse {} // 429
export class InternalServerError extends ErrorResponse {}  // 500
```

### Error Handler Configuration

```typescript
// app/app.config.ts
export default defineAppConfig({
  errorHandlers: {
    401: async ({ response, flash }) => {
      flash.error('Session expired. Please log in again.')
      return navigateTo('/auth/login')
    },
    403: async ({ flash }) => {
      flash.error('Access denied.')
    },
    404: '/not-found',
    422: async ({ response }) => {
      return Promise.reject(new ValidationError(response))
    },
    429: async ({ response }) => {
      return Promise.reject(new TooManyRequestsError(response))
    },
    500: async ({ flash }) => {
      flash.error('Server error. Please try again later.')
    },
  },
})
```

### Action Error Handling Pattern

```typescript
// In actions
import { useHandleActionError } from '~/composables/useHandleActionError'

export default function deleteLeadActionFactory() {
  const deleteLead = deleteLeadMutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (lead: Lead): Promise<void> => {
    try {
      await deleteLead(lead.ulid)
      flash.success('Lead deleted successfully.')
    } catch (error) {
      // Converts error to user-friendly message
      throw handleActionError(error, {
        entity: 'lead',
        operation: 'delete',
      })
    }
  }
}
```

---

## 14. Forms

### Form Builder Pattern

```typescript
// Using useFormBuilder
const form = useFormBuilder<CreateLeadData>()
  .post('/lead-management/leads')
  .data({
    name: '',
    email: '',
    demand: '',
  })
  .waiting('leads.creating')
  .resetOnSuccess(true)
  .success(({ response }) => {
    flash.success('Lead created!')
    emits('close')
  })
  .error(({ error }) => {
    if (error instanceof ValidationError) {
      // Handle validation errors
    }
  })
  .build()

// Submit
await form.submit()

// Access state
form.processing  // true during submission
form.errors      // FormErrors object
form.data        // Reactive form data
```

### XForm Component Pattern

```vue
<XForm
  ref="formRef"
  url="/lead-management/leads"
  method="POST"
  :data="formData"
  :waiting="waitingFor.leads.creating"
  @submit="onSubmit"
  @success="onSuccess"
  @error="onError"
>
  <UFormField label="Name" name="name" :error="form?.errors.first('name')">
    <UInput v-model="formData.name" />
  </UFormField>

  <template #actions>
    <UButton type="submit" label="Create" :loading="form?.processing" />
  </template>
</XForm>
```

### Form Error Display

```typescript
// In form component
const form = useForm(url, method, data)

// Check for errors
form.errors.has('email')        // boolean
form.errors.first('email')      // string | undefined
form.errors.get('email')        // string[] | undefined
form.errors.any()               // boolean

// Clear errors
form.errors.clear()
form.errors.forget('email')
```

---

## 15. Tables

### Column Builder Pattern

```typescript
// app/utils/createColumnBuilder.ts
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
    build: (columnKeys: string[]) => columnKeys.map(key => columns[key]),
    except: (columnsToExclude: string[]) =>
      Object.keys(columns)
        .filter(key => !columnsToExclude.includes(key))
        .map(key => columns[key]),
  }
}
```

### Table Configuration

```typescript
// app/tables/leads.ts
import { h } from 'vue'
import type { TableColumn } from '@tanstack/vue-table'
import type Lead from '~/models/Lead'
import Copyable from '~/components/Common/Copyable.vue'

const ulidColumn: TableColumn<Lead> = {
  id: 'ulid',
  accessorKey: 'ulid',
  header: 'ULID',
  cell: ({ row }) => h(Copyable, { content: row.original.ulid }, () =>
    truncateMiddle(row.original.ulid, 8)
  ),
}

const statusColumn: TableColumn<Lead> = {
  id: 'status',
  accessorKey: 'status',
  header: 'Status',
  cell: ({ row }) => h(UBadge, {
    color: row.getValue('status').color()
  }, () => row.getValue('status').text),
}

const contactColumn: TableColumn<Lead> = {
  id: 'contact',
  accessorKey: 'contact.name',
  header: 'Contact',
  cell: ({ row }) => h(NuxtLink, {
    to: `/contacts/${row.original.contact.ulid}`,
    class: 'text-primary-500 hover:underline',
  }, () => row.original.contact.name),
}

const datesColumn: TableColumn<Lead> = {
  id: 'dates',
  header: 'Created',
  cell: ({ row }) => row.original.createdAt.format('DD MMM YYYY'),
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

### XTable Component

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

---

## 16. Authentication & Authorization

### Authentication Flow

1. App boots → `init.ts` plugin runs
2. `useSanctumAuth().init()` fetches user
3. If authenticated → set user state, start session watcher
4. If not → allow app to load, auth middleware redirects protected routes

### Permission System

```typescript
// Register permissions after login
const { registerPermissions, can, cannot, before } = usePermissions()

// In User model booted()
registerPermissions(this.permissions)

// Admin bypass
before(() => {
  if (user.isAdmin) return true
  return null // Check normally
})

// Check permissions
can('leads.create')                    // Single permission
can(['leads.create', 'leads.update'])  // ANY of these (OR)
cannot('leads.delete')                 // Negation
```

### Page-Level Authorization

```typescript
// In page
definePageMeta({
  permissions: 'leads.list',
})

// Or multiple
definePageMeta({
  permissions: ['leads.list', 'contacts.list'],
})
```

### Component-Level Authorization

```vue
<template>
  <UButton v-if="can('leads.create')" @click="createLead">
    Create Lead
  </UButton>

  <UButton v-if="cannot('leads.delete')" disabled>
    Delete (No Permission)
  </UButton>
</template>
```

---

## 17. Real-time Features

### WebSocket Setup

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-laravel-echo'],

  runtimeConfig: {
    public: {
      echo: {
        key: undefined,
        host: undefined,
        scheme: undefined,
        port: undefined,
      },
    },
  },
})
```

### Channel Subscriptions

```typescript
// In composable or component
const { privateChannel, presenceChannel, leaveChannel } = useRealtime()

// Subscribe to private channel
const channel = privateChannel('leads.{id}', leadId)

// Listen for events
channel.on('LeadUpdated', (event) => {
  // Handle update
  refresh()
})

// Multiple events
channel.on(['LeadUpdated', 'LeadDeleted'], (event) => {
  refresh()
})

// Error handling
channel.error((error) => {
  console.error('Channel error:', error)
})

// Cleanup
onUnmounted(() => {
  leaveChannel('leads.{id}', leadId)
})
```

### Real-time in Composables

```typescript
// app/composables/useRemainingJobCount.ts
const { privateChannel } = useRealtime()
const remainingJobCount = useState<number>('remaining-job-count', () => 0)

const init = () => {
  fetchJobCount() // Initial fetch

  privateChannel(MatchJobs).on(
    [MatchJobCreated, MatchJobRejected, MatchJobComplete],
    async () => {
      await fetchJobCount()
    }
  )
}
```

---

## 18. Code Style & Conventions

### General Rules

- 100 character line length
- Single quotes for strings
- 2-space indentation
- Composition API with `<script setup>`
- TypeScript everywhere

### Component Conventions

- PascalCase for component names
- Multi-word names allowed (eslint rule disabled)
- Props with type inference
- Emits with type definition

```vue
<script lang="ts" setup>
// Props with defaults
const props = withDefaults(defineProps<{
  lead: Lead
  loading?: boolean
}>(), {
  loading: false,
})

// Emits with types
const emits = defineEmits<{
  close: [success: boolean]
  update: [lead: Lead]
}>()
</script>
```

### Function Conventions

- Arrow functions for component methods
- Named exports for composables
- Factory functions for actions/mutations/queries

```typescript
// Arrow function arguments always wrapped
const handleClick = (event) => { ... }

// Blank line above return
function getData() {
  const result = compute()

  return result
}
```

### Import Organization

```typescript
// 1. External packages
import { ref, computed, watch } from 'vue'
import { h } from 'vue'

// 2. Layer imports
import Model from '#layers/base/app/models/Model'
import type { Castable } from '#layers/base/app/types'

// 3. App imports (aliases)
import Lead from '~/models/Lead'
import { ListLeads } from '~/constants/permissions'

// 4. Relative imports
import createLeadMutation from '../mutations/create-lead-mutation'
```

### Naming Patterns

| Pattern | Example |
|---------|---------|
| Models | `Lead`, `SecureLink`, `FeedbackCategory` |
| Enums | `LeadStatus`, `SecureLinkStatus` |
| Composables | `useUser`, `useFeedbackCategories` |
| Factories | `createLeadActionFactory`, `getLeadsQueryFactory` |
| Pages | `leads/index.vue`, `leads/[ulid].vue` |
| Components | `CreateLeadSlideover`, `LeadsTable` |

---

## 19. TypeScript Patterns

### Generic Repository Types

```typescript
// Response types
interface DataResponse<T> {
  data: T
}

interface CollectionResponse<T> extends DataResponse<T[]> {}

interface PaginatedResponse<T> extends CollectionResponse<T> {
  meta: LaravelPagination
  links: LaravelLinks
}

// Usage
async function list(): Promise<PaginatedResponse<Lead>> { ... }
async function get(id: string): Promise<DataResponse<Lead>> { ... }
```

### Castable Interface

```typescript
interface Castable {
  // Static method signature
}

// Implementation
class LeadStatus extends Enum implements Castable {
  static cast(value: string): LeadStatus {
    return LeadStatus.coerce(value)
  }
}
```

### Filter Types

```typescript
interface Filters {
  page?: number
  size?: number
  search?: string
  [index: string]: any
}

// Extend for specific features
interface GetLeadsFilters extends Pick<Filters, 'page' | 'size' | 'search'> {
  status?: string
  testFlag?: boolean
}
```

### Model Factory Types

```typescript
// Query factory
type QueryFactory<TFilters, TResponse> =
  () => (filters: MaybeRef<TFilters>) => UseQueryReturn<TResponse>

// Mutation factory
type MutationFactory<TData, TResult> =
  () => (data: TData) => Promise<TResult>

// Action factory
type ActionFactory<TData, TResult> =
  () => (data: TData) => Promise<TResult>
```

---

## 20. Nuxt Layers

### Layer Architecture

```
nuxt-layers/
├── base/                 # Foundation layer
│   ├── app/
│   │   ├── composables/  # Core composables (useRepository, useQuery, etc.)
│   │   ├── errors/       # Error classes
│   │   ├── interceptors/ # Base interceptors
│   │   ├── models/       # Base Model class
│   │   ├── plugins/      # Core plugins
│   │   ├── repositories/ # BaseRepository, ApiClient
│   │   ├── types/        # Core type definitions
│   │   └── utils/        # 49 utility functions
│   └── nuxt.config.ts
├── nuxt-ui/              # Nuxt UI integration layer
│   └── ...
└── x-ui/                 # Extended UI components layer
    └── ...
```

### Extending Layers

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  extends: [
    // Local paths
    '../../../nuxt-layers/base',
    '../../../nuxt-layers/nuxt-ui',
    '../../../nuxt-layers/x-ui',

    // Or from GitHub
    // ['github:org/nuxt-layers/base#v4.0.0', { install: true }],
  ],
})
```

### Layer Imports

```typescript
// Import from layers using aliases
import Model from '#layers/base/app/models/Model'
import type { Castable, DataResponse } from '#layers/base/app/types'
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
```

### What Each Layer Provides

**Base Layer**:
- `Model` base class with hydration, relations, casting
- `BaseRepository` with CRUD operations
- Core composables: `useRepository`, `useQuery`, `useWait`, `usePermissions`, etc.
- Error classes: `ValidationError`, `ConflictError`, etc.
- 49 utility functions (array, string, object, async, etc.)
- Type definitions
- Interceptor system
- Plugin infrastructure

**Nuxt UI Layer**:
- Pre-configured Nuxt UI setup
- Theme configuration
- Component defaults

**X-UI Layer**:
- Extended/custom UI components
- Application-specific UI patterns

---

## Quick Reference

### Creating a New Feature

1. **Create Model** (`app/models/Feature.ts`)
2. **Create Enum** (if needed) (`app/enums/FeatureStatus.ts`)
3. **Create Repository** (`app/repositories/FeatureRepository.ts`)
4. **Register Repository** in `app/app.config.ts`
5. **Create Query** (`app/features/feature/queries/get-features-query.ts`)
6. **Create Mutation** (`app/features/feature/mutations/create-feature-mutation.ts`)
7. **Create Action** (`app/features/feature/actions/create-feature-action.ts`)
8. **Create Table Config** (`app/tables/features.ts`)
9. **Create Components** (`app/components/Tables/FeaturesTable.vue`, etc.)
10. **Create Page** (`app/pages/features/index.vue`)

### Common Patterns Checklist

- [ ] Model extends `Model` with proper casts and relations
- [ ] Repository extends `BaseRepository` with hydration enabled
- [ ] Queries use `useFilterQuery` with typed filters
- [ ] Mutations use `useWait` for loading states
- [ ] Actions handle errors with `useHandleActionError`
- [ ] Pages set header, breadcrumbs, and permissions
- [ ] Components follow script setup order convention
- [ ] Tables use column builder pattern
- [ ] Forms use `XForm` or `useFormBuilder`

---

*This document is generated from analysis of the teri-for-ops, flowx-app, and nuxt-layers codebases.*
