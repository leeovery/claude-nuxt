# Mutation Patterns

Mutations handle pure API operations with loading state management. No UI feedback - that's actions' job.

## Core Pattern

```typescript
export default function create{Entity}MutationFactory() {
  const api = useRepository('{entity}')
  const { start, stop, waitingFor } = useWait()

  return async (data: Create{Entity}Data): Promise<{Entity}> => {
    try {
      start(waitingFor.{entity}.creating)
      const { data: result } = await api.create(data)
      return result
    } finally {
      stop(waitingFor.{entity}.creating)
    }
  }
}
```

---

## Complete Mutation Examples

### Create Lead

```typescript
// app/features/leads/mutations/create-lead-mutation.ts
import type Lead from '~/models/Lead'

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

### Update Lead

```typescript
// app/features/leads/mutations/update-lead-mutation.ts
import type Lead from '~/models/Lead'

export type UpdateLeadData = Partial<{
  name: string
  email: string
  demand: string
  callScheduledAt: string
  testFlag: boolean
}>

export default function updateLeadMutationFactory() {
  const leadApi = useRepository('leads')
  const { start, stop, waitingFor } = useWait()

  return async (ulid: string, data: UpdateLeadData): Promise<Lead> => {
    try {
      start(waitingFor.lead.updating(ulid))
      const { data: lead } = await leadApi.update(ulid, data)
      return lead
    } finally {
      stop(waitingFor.lead.updating(ulid))
    }
  }
}
```

### Delete Lead

```typescript
// app/features/leads/mutations/delete-lead-mutation.ts
export default function deleteLeadMutationFactory() {
  const leadApi = useRepository('leads')
  const { start, stop, waitingFor } = useWait()

  return async (ulid: string): Promise<void> => {
    try {
      start(waitingFor.lead.deleting(ulid))
      await leadApi.delete(ulid)
    } finally {
      stop(waitingFor.lead.deleting(ulid))
    }
  }
}
```

---

## useWait Pattern

Global loading state management:

```typescript
const { start, stop, is, waitingFor } = useWait()

// Global operations (no ID)
waitingFor.leads.creating       // Boolean ref: creating any lead
waitingFor.leads.listing        // Boolean ref: listing leads

// Per-item operations (with ID)
waitingFor.lead.loading(ulid)   // Boolean ref: loading specific lead
waitingFor.lead.updating(ulid)  // Boolean ref: updating specific lead
waitingFor.lead.deleting(ulid)  // Boolean ref: deleting specific lead
```

### Starting/Stopping

```typescript
// Start waiting state
start(waitingFor.leads.creating)

// Stop waiting state
stop(waitingFor.leads.creating)

// Always use try/finally to ensure stop is called
try {
  start(waitingFor.leads.creating)
  // ... do work
} finally {
  stop(waitingFor.leads.creating)
}
```

### Checking State

```typescript
// Check if operation in progress
const isCreating = is(waitingFor.leads.creating)

// Use in templates
<UButton :loading="is(waitingFor.leads.creating)" />
```

---

## Mutation with Custom Repository Method

```typescript
// app/features/secure-links/mutations/create-secure-link-for-lead-mutation.ts
import type SecureLink from '~/models/SecureLink'

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

---

## Mutation with Multiple API Calls

```typescript
// app/features/leads/mutations/create-lead-with-link-mutation.ts
import type Lead from '~/models/Lead'

export default function createLeadWithLinkMutationFactory() {
  const leadApi = useRepository('leads')
  const secureLinkApi = useRepository('secureLinks')
  const { start, stop, waitingFor } = useWait()

  return async (data: CreateLeadData): Promise<Lead> => {
    try {
      start(waitingFor.leads.creating)

      // Create lead
      const { data: lead } = await leadApi.create(data)

      // Create secure link if requested
      if (data.sendLoginLink) {
        await secureLinkApi.createForLead(lead.ulid)
      }

      // Refetch with relations
      const { data: fullLead } = await leadApi.get(lead.ulid)
      return fullLead
    } finally {
      stop(waitingFor.leads.creating)
    }
  }
}
```

---

## Mutation Responsibilities

| Do | Don't |
|----|-------|
| Call repository methods | Show toast messages |
| Manage loading state | Handle UI confirmations |
| Define data types | Navigate to pages |
| Return results | Catch errors for display |
| Throw errors on failure | Log errors |

Mutations are **pure data operations**. UI concerns belong in actions.

---

## Data Types

Export types for use in actions and components:

```typescript
// Export data type for action/component use
export type CreateLeadData = {
  name: string
  email: string
  demand: string
  // ...
}

// Partial type for updates
export type UpdateLeadData = Partial<CreateLeadData>
```

---

## Error Handling

Mutations **throw errors** - don't catch and handle:

```typescript
// CORRECT: Let error propagate to action
return async (data: CreateLeadData): Promise<Lead> => {
  try {
    start(waitingFor.leads.creating)
    const { data: lead } = await leadApi.create(data)
    return lead
  } finally {
    stop(waitingFor.leads.creating)  // Always stop wait state
  }
}

// WRONG: Catching and logging in mutation
return async (data: CreateLeadData): Promise<Lead | null> => {
  try {
    start(waitingFor.leads.creating)
    const { data: lead } = await leadApi.create(data)
    return lead
  } catch (error) {
    console.error('Failed:', error)  // Don't do this
    return null
  } finally {
    stop(waitingFor.leads.creating)
  }
}
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | `{verb}-{entity}-mutation.ts` |
| Factory function | `{verb}{Entity}MutationFactory` |
| Data type | `{Verb}{Entity}Data` |

Common verbs: `create`, `update`, `delete`, `send`, `resend`, `archive`

---

## Related Skills

- **[nuxt-features](../../nuxt-features/SKILL.md)** - Overview
- **[nuxt-repositories](../../nuxt-repositories/SKILL.md)** - Repository methods
- **[nuxt-composables](../../nuxt-composables/SKILL.md)** - useWait details
