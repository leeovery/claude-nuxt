# Model Patterns

## Base Model Class

All models extend `Model` from the base layer:

```typescript
import Model from '#layers/base/app/models/Model'
```

### Static Factory Methods

```typescript
// Hydrate single instance from data
const lead = Lead.hydrate(apiData)

// Hydrate collection from array
const leads = Lead.collect(apiDataArray)

// Create instance (alias for hydrate)
const lead = Lead.instance(data)
```

### Instance Methods

```typescript
// Compare by primary key
lead.is(otherLead)  // true if same primary key

// Clone instance
const copy = lead.clone()
```

### Override Methods

```typescript
class Lead extends Model {
  // Custom primary key (default: 'id')
  public override primaryKey(): string {
    return 'ulid'
  }

  // Transform raw data before assignment
  public override transform<D>(data: D): D {
    return transformKeys(data, camelCase)
  }

  // Define property casts
  public override casts(): Record<string, Castable> {
    return { status: LeadStatus, createdAt: DateValue }
  }

  // Define model relations
  public override relations(): Record<string, typeof Model> {
    return { contact: Contact, insights: Insight }
  }

  // Strict mode (default: true) - only assign declared properties
  public override strictProperties(): boolean {
    return true
  }
}
```

### Lifecycle Hooks

```typescript
class Lead extends Model {
  // Before hydration
  public override booting(): void {
    // Called before property assignment
  }

  // After hydration complete
  public override booted(): void {
    // Called after casts and relations applied
    // Good for computed setup, event registration
  }
}
```

---

## Model Lifecycle Flow

```
API Response Data
       ↓
booting() hook
       ↓
transform() - Transform raw data (e.g., snake_case → camelCase)
       ↓
Property Assignment (strict or loose based on strictProperties())
       ↓
applyRelations() - Hydrate nested models from relations()
       ↓
applyCasts() - Cast properties using casts()
       ↓
booted() hook
       ↓
Model Instance Ready
```

---

## Complete Model Example

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
  // Scalar properties
  ulid: string
  demand: string
  testFlag: boolean

  // Cast properties (from enums/values)
  status: LeadStatus
  callScheduledAt: DateValue
  createdAt: DateValue
  updatedAt: DateValue

  // Relation properties
  contact: Contact
  insights?: Insight[]
  insightsCount?: number
  secureLinks?: SecureLink[]
  secureLinksCount?: number
  conversations?: Conversation[]
  conversationsCount?: number

  // Custom primary key
  public override primaryKey(): string {
    return 'ulid'
  }

  // Define casts
  public override casts(): Record<string, Castable> {
    return {
      status: LeadStatus,
      callScheduledAt: DateValue,
      createdAt: DateValue,
      updatedAt: DateValue,
    }
  }

  // Define relations
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

  public hasInsights(): boolean {
    return (this.insightsCount ?? 0) > 0
  }
}
```

---

## Relations

### One-to-One

```typescript
// Contact is a single related model
contact: Contact

relations(): Record<string, typeof Model> {
  return {
    contact: Contact,  // Hydrates as Contact instance
  }
}
```

### One-to-Many

```typescript
// Insights is an array of related models
insights?: Insight[]

relations(): Record<string, typeof Model> {
  return {
    insights: Insight,  // Hydrates each item as Insight instance
  }
}
```

### Counts

API often returns counts alongside relations:

```typescript
// From API: { insights: [...], insights_count: 5 }
insights?: Insight[]
insightsCount?: number
```

The count is a scalar - no relation definition needed.

---

## Casts

### Castable Interface

Any class implementing `Castable` can be used in casts:

```typescript
interface Castable {
  // Static cast method
}

// Implementation
class LeadStatus extends Enum implements Castable {
  static cast(value: string): LeadStatus {
    return LeadStatus.coerce(value)
  }
}
```

### Built-in Castable Types

```typescript
// Enums - extend Enum base class
status: LeadStatus

// Value objects - implement Castable
createdAt: DateValue
scheduledFor: DateValue
```

### Nullable Casts

```typescript
// If property might be null, make it optional
callScheduledAt?: DateValue

// The cast still works - undefined/null values skip casting
casts(): Record<string, Castable> {
  return {
    callScheduledAt: DateValue,  // Only casts if value exists
  }
}
```

---

## Data Transformation

### Snake Case to Camel Case

```typescript
import { transformKeys } from '#layers/base/app/utils'
import { camelCase } from 'change-case'

export default class User extends Model {
  uuid: string
  sessionId: string      // From API: session_id
  conversationId: string // From API: conversation_id

  public override transform<D>(data: D): D {
    return transformKeys(data, camelCase)
  }
}
```

---

## Model with Booted Hook

```typescript
import Model from '#layers/base/app/models/Model'
import { registerPermissions } from '~/composables/usePermissions'

export default class User extends Model {
  uuid: string
  name: string
  email: string
  permissions?: string[]

  public override booted() {
    // Register permissions after hydration
    if (this.permissions) {
      registerPermissions(this.permissions)
    }
  }
}
```

---

## Simple Model (No Relations)

```typescript
// app/models/Insight.ts
import Model from '#layers/base/app/models/Model'
import type { Castable } from '#layers/base/app/types'
import DateValue from '~/values/DateValue'

export default class Insight extends Model {
  ulid: string
  category: string
  summary: string
  createdAt: DateValue

  public override primaryKey(): string {
    return 'ulid'
  }

  public override casts(): Record<string, Castable> {
    return {
      createdAt: DateValue,
    }
  }
}
```

---

## Usage Patterns

### Hydrating from Repository

```typescript
// Repositories with hydration enabled return model instances
const leadApi = useRepository('leads')
const { data: lead } = await leadApi.get('ulid123')

// lead is already a Lead instance
lead.status.color()
lead.contact.name
```

### Manual Hydration

```typescript
// When you have raw data
const rawData = { ulid: '...', status: 'new lead', ... }
const lead = Lead.hydrate(rawData)
```

### Collection Hydration

```typescript
// Hydrate array of items
const rawArray = [{ ... }, { ... }]
const leads = Lead.collect(rawArray)

leads.forEach(lead => {
  console.log(lead.status.color())
})
```

### Model Comparison

```typescript
// Compare by primary key
if (selectedLead.is(lead)) {
  // Same lead
}

// Find in array
const found = leads.find(l => l.is(targetLead))
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | PascalCase: `Lead.ts`, `SecureLink.ts` |
| Class name | PascalCase: `Lead`, `SecureLink` |
| Properties | camelCase: `createdAt`, `secureLinks` |
| Relations | camelCase, matches property: `contact`, `insights` |

---

## Related Skills

- **[nuxt-enums](../../nuxt-enums/SKILL.md)** - Enum casting
- **[nuxt-repositories](../../nuxt-repositories/SKILL.md)** - Repository hydration
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Using models in features
