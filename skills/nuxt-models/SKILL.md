---
name: nuxt-models
description: Domain model classes with automatic hydration, relations, and type casting. Use when creating models for API entities, defining relationships between models, casting properties to enums/dates, or creating value objects.
---

# Nuxt Models

Type-safe domain models with automatic hydration, relations, and property casting.

## Core Concepts

**[models.md](references/models.md)** - Complete model patterns, lifecycle, relations, casts
**[values.md](references/values.md)** - Value objects for typed wrappers (DateValue, etc.)

## Basic Model

```typescript
// app/models/Lead.ts
import Model from '#layers/base/app/models/Model'
import type { Castable } from '#layers/base/app/types'
import LeadStatus from '~/enums/LeadStatus'
import DateValue from '~/values/DateValue'
import Contact from '~/models/Contact'

export default class Lead extends Model {
  ulid: string
  status: LeadStatus
  demand: string
  testFlag: boolean
  contact: Contact
  createdAt: DateValue

  public override primaryKey(): string {
    return 'ulid'
  }

  public override casts(): Record<string, Castable> {
    return {
      status: LeadStatus,
      createdAt: DateValue,
    }
  }

  public override relations(): Record<string, typeof Model> {
    return {
      contact: Contact,
    }
  }

  public isTest(): boolean {
    return this.testFlag
  }
}
```

## Model Lifecycle

```
API Response → booting() → transform() → Property Assignment
     → Relations Hydrated → Casts Applied → booted() → Ready
```

## Usage

```typescript
// Hydrate from API response
const lead = Lead.hydrate(apiResponse.data)

// Hydrate collection
const leads = Lead.collect(apiResponse.data)

// Access typed properties
lead.status.color()           // Enum method
lead.createdAt.format()       // Value object method
lead.contact.name             // Relation property

// Compare models
lead.is(otherLead)            // Same primary key?
```
