# Enum Patterns

## Base Enum Class

All enums extend `Enum` from the base layer:

```typescript
import Enum from '#layers/base/app/enums/Enum'
```

### Core Methods

```typescript
// Create static enum value
static readonly Value = MyEnum.create('value')

// Coerce string to enum instance
static coerce(value: string): MyEnum

// Get all defined values
static values(): MyEnum[]

// Instance comparison
instance.is(MyEnum.Value)

// Get raw value
instance.value  // string
```

---

## Complete Enum Example

```typescript
// app/enums/LeadStatus.ts
import Enum from '#layers/base/app/enums/Enum'
import type { Castable, EnumStoreObject } from '#layers/base/app/types'

export default class LeadStatus extends Enum implements Castable {
  // Define all possible values
  static readonly NewLead = LeadStatus.create('new lead')
  static readonly LinkSent = LeadStatus.create('link sent')
  static readonly TranscriptReceived = LeadStatus.create('transcript received')
  static readonly Completed = LeadStatus.create('completed')
  static readonly Cancelled = LeadStatus.create('cancelled')

  // Required for model casting
  static cast(value: string | EnumStoreObject): LeadStatus {
    return LeadStatus.coerce(value)
  }

  // UI color mapping
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

  // Display text
  get text(): string {
    return this.value
  }

  // Custom display text (if different from value)
  get label(): string {
    switch (this.value) {
      case 'new lead':
        return 'New Lead'
      case 'link sent':
        return 'Link Sent'
      case 'transcript received':
        return 'Transcript Received'
      case 'completed':
        return 'Completed'
      case 'cancelled':
        return 'Cancelled'
      default:
        return this.value
    }
  }

  // Boolean helpers
  isActive(): boolean {
    return !this.is(LeadStatus.Completed) && !this.is(LeadStatus.Cancelled)
  }

  isTerminal(): boolean {
    return this.is(LeadStatus.Completed) || this.is(LeadStatus.Cancelled)
  }
}
```

---

## Castable Interface

Enums implement `Castable` for model integration:

```typescript
import type { Castable } from '#layers/base/app/types'

class LeadStatus extends Enum implements Castable {
  // Static cast method - called by model hydration
  static cast(value: string | EnumStoreObject): LeadStatus {
    return LeadStatus.coerce(value)
  }
}
```

### In Models

```typescript
import LeadStatus from '~/enums/LeadStatus'

class Lead extends Model {
  status: LeadStatus

  public override casts(): Record<string, Castable> {
    return {
      status: LeadStatus,  // Auto-casts string → LeadStatus
    }
  }
}
```

---

## Behavior Methods

### Color Methods

Map enum values to UI colors:

```typescript
color(): string {
  switch (this.value) {
    case 'pending': return 'warning'
    case 'active': return 'success'
    case 'failed': return 'error'
    default: return 'neutral'
  }
}
```

Colors align with Nuxt UI color scheme:
- `primary`, `secondary`
- `success`, `warning`, `error`, `info`
- `neutral`

### Icon Methods

Map enum values to icons:

```typescript
icon(): string {
  switch (this.value) {
    case 'pending': return 'i-heroicons-clock'
    case 'active': return 'i-heroicons-check-circle'
    case 'failed': return 'i-heroicons-x-circle'
    default: return 'i-heroicons-question-mark-circle'
  }
}
```

### Boolean Helpers

Add semantic checks:

```typescript
isPending(): boolean {
  return this.is(EvaluationStatus.Pending)
}

isCompleted(): boolean {
  return this.is(EvaluationStatus.Completed)
}

canBeEdited(): boolean {
  return this.is(EvaluationStatus.Pending) || this.is(EvaluationStatus.InProgress)
}
```

---

## Template Usage

### Badge Display

```vue
<UBadge :color="lead.status.color()">
  {{ lead.status.text }}
</UBadge>
```

### With Icon

```vue
<UBadge :color="lead.status.color()">
  <UIcon :name="lead.status.icon()" class="mr-1" />
  {{ lead.status.label }}
</UBadge>
```

### Conditional Rendering

```vue
<template>
  <div v-if="lead.status.isActive()">
    <!-- Show active lead UI -->
  </div>

  <UButton
    v-if="!lead.status.isTerminal()"
    @click="markComplete"
  >
    Complete
  </UButton>
</template>
```

### Select Options

```vue
<USelect
  v-model="filters.status"
  :options="statusOptions"
  placeholder="Filter by status"
/>

<script setup>
import LeadStatus from '~/enums/LeadStatus'

const statusOptions = computed(() =>
  LeadStatus.values().map(status => ({
    value: status.value,
    label: status.label,
  }))
)
</script>
```

---

## Comparison Methods

### Single Comparison

```typescript
// Check if enum matches specific value
if (lead.status.is(LeadStatus.Completed)) {
  // Handle completed lead
}
```

### Multiple Comparison

```typescript
// Check if enum is one of several values
const activeStatuses = [LeadStatus.NewLead, LeadStatus.LinkSent]
if (activeStatuses.some(s => lead.status.is(s))) {
  // Handle active lead
}

// Or add helper method
isActive(): boolean {
  return [LeadStatus.NewLead, LeadStatus.LinkSent].some(s => this.is(s))
}
```

---

## Static Methods

### Get All Values

```typescript
// Returns array of all defined enum instances
const allStatuses = LeadStatus.values()
// [NewLead, LinkSent, TranscriptReceived, Completed, Cancelled]
```

### Find by Value

```typescript
// Coerce string back to enum instance
const status = LeadStatus.coerce('completed')
status.is(LeadStatus.Completed)  // true
```

---

## More Enum Examples

### EvaluationStatus

```typescript
// app/enums/EvaluationStatus.ts
export default class EvaluationStatus extends Enum implements Castable {
  static readonly Pending = EvaluationStatus.create('pending')
  static readonly Scheduled = EvaluationStatus.create('scheduled')
  static readonly InProgress = EvaluationStatus.create('in_progress')
  static readonly Completed = EvaluationStatus.create('completed')
  static readonly Failed = EvaluationStatus.create('failed')
  static readonly Reviewed = EvaluationStatus.create('reviewed')

  static cast(value: string): EvaluationStatus {
    return EvaluationStatus.coerce(value)
  }

  color(): string {
    switch (this.value) {
      case 'pending': return 'warning'
      case 'scheduled': return 'info'
      case 'in_progress': return 'primary'
      case 'completed': return 'success'
      case 'failed': return 'error'
      case 'reviewed': return 'success'
      default: return 'neutral'
    }
  }
}
```

### SecureLinkStatus

```typescript
// app/enums/SecureLinkStatus.ts
export default class SecureLinkStatus extends Enum implements Castable {
  static readonly Pending = SecureLinkStatus.create('pending')
  static readonly Sent = SecureLinkStatus.create('sent')
  static readonly Delivered = SecureLinkStatus.create('delivered')
  static readonly Bounced = SecureLinkStatus.create('bounced')

  static cast(value: string): SecureLinkStatus {
    return SecureLinkStatus.coerce(value)
  }

  color(): string {
    switch (this.value) {
      case 'pending': return 'warning'
      case 'sent': return 'info'
      case 'delivered': return 'success'
      case 'bounced': return 'error'
      default: return 'neutral'
    }
  }

  wasSent(): boolean {
    return !this.is(SecureLinkStatus.Pending)
  }
}
```

---

## Directory Structure

```
app/
└── enums/
    ├── LeadStatus.ts
    ├── EvaluationStatus.ts
    ├── EvaluationReviewStatus.ts
    └── SecureLinkStatus.ts
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | PascalCase: `LeadStatus.ts` |
| Class name | PascalCase: `LeadStatus` |
| Static values | PascalCase: `NewLead`, `InProgress` |
| String values | lowercase/snake: `'new lead'`, `'in_progress'` |

---

## Related Skills

- **[nuxt-models](../../nuxt-models/SKILL.md)** - Using enums in models
- **[nuxt-components](../../nuxt-components/SKILL.md)** - Enum display patterns
