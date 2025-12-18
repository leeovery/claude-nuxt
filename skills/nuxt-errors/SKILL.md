---
name: nuxt-errors
description: Error handling with error classes, handlers, and interceptors. Use when handling API errors, displaying validation errors, configuring error handlers, or implementing error interceptors.
---

# Nuxt Error Handling

Centralized error handling with typed error classes and configurable handlers.

## Core Concepts

**[errors.md](references/errors.md)** - Error classes, handlers, interceptors

## Error Classes

```typescript
import { ValidationError } from '#layers/base/app/errors/validation-error'
import { ConflictError } from '#layers/base/app/errors/conflict-error'
import { TooManyRequestsError } from '#layers/base/app/errors/too-many-requests-error'

// ValidationError (422)
if (error instanceof ValidationError) {
  error.errors    // Record<string, string[]>
  error.message   // string
  error.mapToFormErrors()  // FormError[]
}
```

## Error Handlers Configuration

```typescript
// app.config.ts
export default defineAppConfig({
  errorHandlers: {
    401: async ({ flash }) => navigateTo('/auth/login'),
    422: async ({ response }) => Promise.reject(new ValidationError(response)),
    500: async ({ flash }) => flash.error('Server error'),
  },
})
```

## Action Error Handling

```typescript
const { handleActionError } = useHandleActionError()

try {
  await createLead(data)
} catch (error) {
  throw handleActionError(error, {
    entity: 'lead',
    operation: 'create',
  })
}
// Shows: "Failed to create lead. [error details]"
```
