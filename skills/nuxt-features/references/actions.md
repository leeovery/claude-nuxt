# Action Patterns

Actions orchestrate mutations and provide user feedback. They're the glue between UI and data operations.

## Core Pattern

```typescript
export default function {verb}{Entity}ActionFactory() {
  const mutation = {verb}{Entity}MutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (data: {Verb}{Entity}Data): Promise<{Entity}> => {
    try {
      const result = await mutation(data)
      flash.success('{Entity} {verb}d successfully.')
      return result
    } catch (error) {
      throw handleActionError(error, {
        entity: '{entity}',
        operation: '{verb}',
      })
    }
  }
}
```

---

## Complete Action Examples

### Create Lead Action

```typescript
// app/features/leads/actions/create-lead-action.ts
import createLeadMutationFactory, { type CreateLeadData } from '../mutations/create-lead-mutation'
import type Lead from '~/models/Lead'

export default function createLeadActionFactory() {
  const createLead = createLeadMutationFactory()
  const { copy } = useClipboard()
  const flash = useFlash()
  const { trigger } = useConfirmationToast()
  const { handleActionError } = useHandleActionError()

  return async (data: CreateLeadData): Promise<Lead> => {
    try {
      const lead = await createLead(data)
      flash.success('Lead created successfully.')

      // Show confirmation with secure link if created
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

### Update Lead Action

```typescript
// app/features/leads/actions/update-lead-action.ts
import updateLeadMutationFactory, { type UpdateLeadData } from '../mutations/update-lead-mutation'
import type Lead from '~/models/Lead'

export default function updateLeadActionFactory() {
  const updateLead = updateLeadMutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (ulid: string, data: UpdateLeadData): Promise<Lead> => {
    try {
      const lead = await updateLead(ulid, data)
      flash.success('Lead updated successfully.')
      return lead
    } catch (error) {
      throw handleActionError(error, {
        entity: 'lead',
        operation: 'update',
      })
    }
  }
}
```

### Delete Lead Action

```typescript
// app/features/leads/actions/delete-lead-action.ts
import deleteLeadMutationFactory from '../mutations/delete-lead-mutation'
import type Lead from '~/models/Lead'

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

---

## Action Responsibilities

| Do | Don't |
|----|-------|
| Call mutations | Call repositories directly |
| Show success messages | Manage loading states (mutation does this) |
| Show error messages | Define data types (mutation does this) |
| Trigger confirmations | Swallow errors silently |
| Navigate after success | Make direct API calls |
| Orchestrate multiple mutations | |

---

## useHandleActionError

Standardized error handling with user-friendly messages:

```typescript
// app/composables/useHandleActionError.ts
export function useHandleActionError() {
  const flash = useFlash()

  const handleActionError = (
    error: unknown,
    context: { entity: string; operation: string }
  ) => {
    const message = `Failed to ${context.operation} ${context.entity}.`

    if (error instanceof ValidationError) {
      flash.error(message, error.message)
    } else {
      flash.error(message, 'An unexpected error occurred.')
    }

    return error  // Re-throw for component handling
  }

  return { handleActionError }
}
```

### Usage

```typescript
catch (error) {
  throw handleActionError(error, {
    entity: 'lead',
    operation: 'create',
  })
}
// Shows: "Failed to create lead. [error details]"
```

---

## Action with Confirmation

```typescript
// app/features/leads/actions/delete-lead-with-confirm-action.ts
export default function deleteLeadWithConfirmActionFactory() {
  const deleteLead = deleteLeadMutationFactory()
  const flash = useFlash()
  const { trigger } = useConfirmationToast()
  const { handleActionError } = useHandleActionError()

  return (lead: Lead): Promise<boolean> => {
    return new Promise((resolve) => {
      trigger({
        title: 'Delete Lead?',
        description: `Are you sure you want to delete "${lead.contact.name}"?`,
        confirmLabel: 'Delete',
        cancelLabel: 'Cancel',
        onConfirm: async () => {
          try {
            await deleteLead(lead.ulid)
            flash.success('Lead deleted successfully.')
            resolve(true)
          } catch (error) {
            handleActionError(error, { entity: 'lead', operation: 'delete' })
            resolve(false)
          }
        },
        onCancel: () => resolve(false),
      })
    })
  }
}
```

---

## Action with Navigation

```typescript
// app/features/leads/actions/create-and-navigate-action.ts
export default function createAndNavigateActionFactory() {
  const createLead = createLeadMutationFactory()
  const flash = useFlash()
  const router = useRouter()
  const { handleActionError } = useHandleActionError()

  return async (data: CreateLeadData): Promise<Lead> => {
    try {
      const lead = await createLead(data)
      flash.success('Lead created successfully.')

      // Navigate to new lead
      await router.push(`/leads/${lead.ulid}`)

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

---

## Action Orchestrating Multiple Mutations

```typescript
// app/features/leads/actions/close-lead-action.ts
export default function closeLeadActionFactory() {
  const updateLead = updateLeadMutationFactory()
  const createInsight = createInsightMutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (lead: Lead, closureNotes: string): Promise<Lead> => {
    try {
      // Create closure insight
      await createInsight({
        leadUlid: lead.ulid,
        category: 'closure',
        summary: closureNotes,
      })

      // Update lead status
      const updatedLead = await updateLead(lead.ulid, {
        status: 'completed',
      })

      flash.success('Lead closed successfully.')
      return updatedLead
    } catch (error) {
      throw handleActionError(error, {
        entity: 'lead',
        operation: 'close',
      })
    }
  }
}
```

---

## Bulk Actions

```typescript
// app/features/leads/actions/bulk-delete-action.ts
export default function bulkDeleteLeadsActionFactory() {
  const deleteLead = deleteLeadMutationFactory()
  const flash = useFlash()
  const { handleActionError } = useHandleActionError()

  return async (leads: Lead[]): Promise<{ success: number; failed: number }> => {
    let success = 0
    let failed = 0

    for (const lead of leads) {
      try {
        await deleteLead(lead.ulid)
        success++
      } catch {
        failed++
      }
    }

    if (failed === 0) {
      flash.success(`${success} lead(s) deleted successfully.`)
    } else {
      flash.warning(`${success} deleted, ${failed} failed.`)
    }

    return { success, failed }
  }
}
```

---

## Using Actions in Components

```vue
<script lang="ts" setup>
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

// Handle submission
const onSubmit = async (data: CreateLeadData) => {
  const lead = await createLeadAction(data)
  // Action handles success message
  emits('close', true)
}
</script>
```

---

## Naming Conventions

| Convention | Example |
|------------|---------|
| File name | `{verb}-{entity}-action.ts` |
| Factory function | `{verb}{Entity}ActionFactory` |

Common patterns:
- `create-lead-action.ts` → `createLeadActionFactory`
- `update-lead-action.ts` → `updateLeadActionFactory`
- `delete-lead-action.ts` → `deleteLeadActionFactory`
- `bulk-delete-leads-action.ts` → `bulkDeleteLeadsActionFactory`
- `close-lead-action.ts` → `closeLeadActionFactory`

---

## Related Skills

- **[nuxt-features](../../nuxt-features/SKILL.md)** - Overview
- **[nuxt-composables](../../nuxt-composables/SKILL.md)** - useFlash, useConfirmationToast
- **[nuxt-errors](../../nuxt-errors/SKILL.md)** - Error handling patterns
