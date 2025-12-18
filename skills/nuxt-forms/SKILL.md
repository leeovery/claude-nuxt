---
name: nuxt-forms
description: Form handling with XForm component and useFormBuilder. Use when creating forms, handling validation errors, managing form state, or building form-based slideovers and modals.
---

# Nuxt Forms

Form handling with XForm component and validation error display.

## Core Concepts

**[forms.md](references/forms.md)** - XForm, useFormBuilder, validation patterns

## XForm Component

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

## Form Data Pattern

```typescript
const formData = ref<CreateLeadData>({
  email: '',
  name: '',
  demand: '',
  testFlag: false,
})

const onSubmit = async (data: CreateLeadData) => {
  await createLeadAction(data)
}

const onSuccess = () => {
  emits('close', true)
}
```

## Validation Errors

```typescript
// Access errors
form.errors.has('email')       // boolean
form.errors.first('email')     // string | undefined
form.errors.get('email')       // string[] | undefined
form.errors.any()              // boolean

// Display in template
<UFormField label="Email" name="email" :error="form?.errors.first('email')">
```
