# Repository Patterns

## BaseRepository Class

All repositories extend `BaseRepository<T>` from the base layer:

```typescript
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
```

### Core Properties

```typescript
class MyRepository extends BaseRepository<MyModel> {
  // API resource path (required)
  protected resource = '/api/my-resource'

  // Enable model hydration (optional, default: false)
  protected hydration = true

  // Hydrator instance (required if hydration: true)
  protected hydrator = new ModelHydrator(MyModel)
}
```

### Inherited CRUD Methods

```typescript
// List all (with optional params)
list(params?: GenericQueryParams): Promise<CollectionResponse<T> | PaginatedResponse<T>>

// Get single by ID
get(id: string | number, params?): Promise<DataResponse<T>>

// Create new
create(data?): Promise<DataResponse<T>>

// Update existing
update(id: string | number, data): Promise<DataResponse<T>>

// Delete
delete(id: string | number): Promise<void>
```

### HTTP Methods

```typescript
// Available for custom methods
jsonGet(url: string, params?): Promise<DataResponse<T>>
jsonPost(url: string, data?): Promise<DataResponse<T>>
jsonPut(url: string, data?): Promise<DataResponse<T>>
jsonPatch(url: string, data?): Promise<DataResponse<T>>
jsonDelete(url: string): Promise<void>
```

---

## Complete Repository Example

```typescript
// app/repositories/LeadRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Lead from '~/models/Lead'
import type { CollectionResponse, DataResponse, GenericQueryParams, OutgoingQueryParams } from '#layers/base/app/types'

export default class LeadRepository extends BaseRepository<Lead> {
  // API resource path
  protected resource = '/lead-management/leads'

  // Enable automatic model hydration
  protected hydration = true
  protected hydrator = new ModelHydrator(Lead)

  // Custom: List leads by contact
  async listByContact(
    contactUlid: string,
    params?: GenericQueryParams & OutgoingQueryParams
  ): Promise<CollectionResponse<Lead>> {
    return this.jsonGet(`/lead-management/contacts/${contactUlid}/leads`, params)
  }

  // Custom: Update lead status
  async updateStatus(
    ulid: string,
    status: string
  ): Promise<DataResponse<Lead>> {
    return this.jsonPatch(`${this.resource}/${ulid}/status`, { status })
  }
}
```

---

## Model Hydration

### Enabling Hydration

```typescript
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Lead from '~/models/Lead'

class LeadRepository extends BaseRepository<Lead> {
  protected hydration = true
  protected hydrator = new ModelHydrator(Lead)
}
```

### How It Works

1. Repository makes API call
2. Response passes through hydrator
3. Single items: `Model.hydrate(data)`
4. Collections: `Model.collect(data)`
5. Typed model instances returned

### Hydration Control

```typescript
const leadApi = useRepository('leads')

// Temporarily disable hydration
const rawData = await leadApi.withoutHydration(async (repo) => {
  return repo.list()  // Returns raw API response
})

// Ensure hydration (even if normally disabled)
const models = await leadApi.withHydration(async (repo) => {
  return repo.list()  // Returns hydrated models
})
```

---

## Repository Registration

### Simple Registration

```typescript
// app/app.config.ts
import LeadRepository from '~/repositories/LeadRepository'
import ContactRepository from '~/repositories/ContactRepository'
import SecureLinkRepository from '~/repositories/SecureLinkRepository'

export default defineAppConfig({
  repositories: {
    leads: LeadRepository,
    contacts: ContactRepository,
    secureLinks: SecureLinkRepository,
  },
})
```

### With Custom Fetch Options

```typescript
export default defineAppConfig({
  repositories: {
    // Simple format (uses global apiUrl)
    leads: LeadRepository,

    // Extended format with custom options
    externalService: {
      repository: ExternalRepository,
      fetchOptions: {
        baseURL: 'https://external-api.example.com',
        timeout: 30000,
        headers: {
          'X-Service': 'external',
        },
      },
    },
  },
})
```

### Environment Variable Overrides

```bash
# Override specific repository base URLs
NUXT_PUBLIC_REPOSITORIES_LEADS_FETCH_OPTIONS_BASE_URL=https://api.example.com
NUXT_PUBLIC_REPOSITORIES_CONTACTS_FETCH_OPTIONS_BASE_URL=https://contacts-api.example.com
```

---

## Using Repositories

### Basic Usage

```typescript
// Get typed repository instance
const leadApi = useRepository('leads')  // Returns LeadRepository

// List all
const { data: leads } = await leadApi.list()

// Get single
const { data: lead } = await leadApi.get('ulid123')

// Create
const { data: newLead } = await leadApi.create({
  name: 'John Doe',
  email: 'john@example.com',
})

// Update
const { data: updated } = await leadApi.update('ulid123', {
  name: 'Jane Doe',
})

// Delete
await leadApi.delete('ulid123')
```

### With Query Parameters

```typescript
// Include relations
const { data: leads } = await leadApi.list({
  include: 'contact,secure-links',
})

// With filters
const { data: leads } = await leadApi.list({
  filter: {
    status: 'new lead',
    'test-flag': false,
  },
})

// Pagination
const { data: leads } = await leadApi.list({
  page: { number: 1, size: 25 },
})

// Combined
const { data: leads } = await leadApi.list({
  include: 'contact',
  filter: { status: 'new lead' },
  page: { number: 1, size: 25 },
})
```

### Custom Methods

```typescript
const leadApi = useRepository('leads')

// Use custom method
const { data: contactLeads } = await leadApi.listByContact('contact-ulid')

const secureLinkApi = useRepository('secureLinks')
const { data: link } = await secureLinkApi.createForLead('lead-ulid')
```

---

## More Repository Examples

### ContactRepository

```typescript
// app/repositories/ContactRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Contact from '~/models/Contact'

export default class ContactRepository extends BaseRepository<Contact> {
  protected resource = '/lead-management/contacts'
  protected hydration = true
  protected hydrator = new ModelHydrator(Contact)

  // Search contacts by term
  async search(searchTerm: string) {
    return this.jsonGet(`${this.resource}/search`, {
      filter: { search: searchTerm },
    })
  }
}
```

### SecureLinkRepository

```typescript
// app/repositories/SecureLinkRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import SecureLink from '~/models/SecureLink'

export default class SecureLinkRepository extends BaseRepository<SecureLink> {
  protected resource = '/lead-management/secure-links'
  protected hydration = true
  protected hydrator = new ModelHydrator(SecureLink)

  // Create secure link for a lead
  async createForLead(leadUlid: string) {
    return this.jsonPost(`/lead-management/leads/${leadUlid}/secure-links`)
  }

  // Resend secure link
  async resend(secureLink: SecureLink) {
    return this.jsonPost(`${this.resource}/${secureLink.ulid}/resend`)
  }
}
```

### EvaluationRepository

```typescript
// app/repositories/EvaluationRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Evaluation from '~/models/Evaluation'

export default class EvaluationRepository extends BaseRepository<Evaluation> {
  protected resource = '/evaluations'
  protected hydration = true
  protected hydrator = new ModelHydrator(Evaluation)

  // No custom methods - uses standard CRUD
}
```

---

## Response Types

### DataResponse

Single item response:

```typescript
interface DataResponse<T> {
  data: T
}

const { data: lead } = await leadApi.get('ulid')
// lead is typed as Lead
```

### CollectionResponse

Array response:

```typescript
interface CollectionResponse<T> {
  data: T[]
}

const { data: leads } = await leadApi.list()
// leads is typed as Lead[]
```

### PaginatedResponse

Paginated array response:

```typescript
interface PaginatedResponse<T> {
  data: T[]
  meta: {
    current_page: number
    from: number
    last_page: number
    per_page: number
    to: number
    total: number
  }
  links: {
    first: string
    last: string
    prev: string | null
    next: string | null
  }
}
```

---

## Directory Structure

```
app/
└── repositories/
    ├── LeadRepository.ts
    ├── ContactRepository.ts
    ├── SecureLinkRepository.ts
    └── EvaluationRepository.ts
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | PascalCase + "Repository": `LeadRepository.ts` |
| Class name | PascalCase + "Repository": `LeadRepository` |
| Config key | camelCase plural: `leads`, `secureLinks` |
| Resource path | kebab-case: `/lead-management/leads` |

---

## Related Skills

- **[nuxt-models](../../nuxt-models/SKILL.md)** - Models for hydration
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Using repositories in queries/mutations
- **[nuxt-config](../../nuxt-config/SKILL.md)** - Repository configuration
