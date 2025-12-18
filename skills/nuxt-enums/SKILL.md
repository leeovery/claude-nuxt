---
name: nuxt-enums
description: TypeScript enum pattern with Castable interface for model integration. Use when creating enums with behavior methods (colors, labels), defining fixed value sets, or integrating enums with the model casting system.
---

# Nuxt Enums

Class-based enums with behavior methods and model casting integration.

## Core Concepts

**[enums.md](references/enums.md)** - Complete enum patterns, behavior methods, UI integration

## Basic Enum

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
  static cast(value: string): LeadStatus {
    return LeadStatus.coerce(value)
  }

  // Behavior methods
  color(): string {
    switch (this.value) {
      case 'new lead': return 'primary'
      case 'link sent': return 'info'
      case 'completed': return 'success'
      case 'cancelled': return 'error'
      default: return 'neutral'
    }
  }

  get text(): string {
    return this.value
  }
}
```

## Usage

```typescript
// In models (auto-cast from API string)
status: LeadStatus

// Comparisons
lead.status.is(LeadStatus.Completed)

// Get all values
LeadStatus.values()

// In templates
<UBadge :color="lead.status.color()">{{ lead.status.text }}</UBadge>
```
