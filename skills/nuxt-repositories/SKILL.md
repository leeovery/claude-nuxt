---
name: nuxt-repositories
description: Repository pattern for API access with automatic model hydration. Use when creating repositories for API resources, configuring model hydration, adding custom API methods, or registering repositories in app config.
---

# Nuxt Repositories

Data access layer with CRUD operations and automatic model hydration.

## Core Concepts

**[repositories.md](references/repositories.md)** - Complete repository patterns, registration, custom methods

## Basic Repository

```typescript
// app/repositories/LeadRepository.ts
import { BaseRepository } from '#layers/base/app/repositories/base-repository'
import { ModelHydrator } from '#layers/base/app/repositories/hydrators/model-hydrator'
import Lead from '~/models/Lead'

export default class LeadRepository extends BaseRepository<Lead> {
  protected resource = '/lead-management/leads'
  protected hydration = true
  protected hydrator = new ModelHydrator(Lead)

  // Custom method
  async listByContact(contactUlid: string) {
    return this.jsonGet(`/lead-management/contacts/${contactUlid}/leads`)
  }
}
```

## Registration

```typescript
// app/app.config.ts
export default defineAppConfig({
  repositories: {
    leads: LeadRepository,
    contacts: ContactRepository,
  },
})
```

## Usage

```typescript
// Get typed repository instance
const leadApi = useRepository('leads')

// CRUD operations (returns hydrated models)
const { data: leads } = await leadApi.list()
const { data: lead } = await leadApi.get('ulid123')
const { data: newLead } = await leadApi.create({ name: 'John' })
await leadApi.update('ulid123', { name: 'Jane' })
await leadApi.delete('ulid123')

// With query params
const { data: leads } = await leadApi.list({
  include: 'contact,secure-links',
  filter: { status: 'new lead' },
})
```
