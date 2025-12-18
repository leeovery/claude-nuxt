# Form Patterns

## XForm Component

The XForm component from x-ui layer handles form state, submission, and validation:

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
  @before="onBefore"
  @after="onAfter"
  @finish="onFinish"
>
  <!-- Form content -->

  <template #actions>
    <UButton type="submit" />
  </template>
</XForm>
```

### Props

| Prop | Type | Description |
|------|------|-------------|
| `url` | `string` | API endpoint |
| `method` | `string` | HTTP method (POST, PUT, PATCH, DELETE) |
| `data` | `object` | Form data (reactive) |
| `waiting` | `WaitRef` | Loading state ref |
| `resetOnSuccess` | `boolean` | Reset form after success |

### Events

| Event | Payload | Description |
|-------|---------|-------------|
| `@submit` | `data` | Before submission (can be async) |
| `@before` | `{ data }` | Before API call |
| `@success` | `{ response }` | On successful response |
| `@error` | `{ error }` | On error response |
| `@after` | `{ response?, error? }` | After API call (success or error) |
| `@finish` | - | After everything (cleanup) |

---

## Complete Form Example

```vue
<!-- app/components/Slideovers/CreateLeadSlideover.vue -->
<script lang="ts" setup>
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'
import type { CreateLeadData } from '~/features/leads/mutations/create-lead-mutation'

// Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits<{ close: [success: boolean] }>()

// Composables
const { is, waitingFor } = useWait()

// Refs
const formRef = useTemplateRef('formRef')
const existingContact = ref<Contact>()

// Form data
const formData = ref<CreateLeadData>({
  email: props.contact?.email || '',
  name: props.contact?.name || '',
  demand: '',
  callScheduledAt: '',
  testFlag: false,
  sendLoginLink: true,
})

// Watchers
watch(existingContact, (contact) => {
  if (contact) {
    formData.value.name = contact.name
    formData.value.email = contact.email
  }
})

// Methods
const createLeadAction = createLeadActionFactory()

const onSubmit = async (data: CreateLeadData) => {
  return createLeadAction(data)
}

const onSuccess = () => {
  emits('close', true)
}

const onError = ({ error }: { error: unknown }) => {
  // Error already handled by action
  // Additional handling if needed
}
</script>

<template>
  <USlideover title="Create Lead">
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
      <div class="space-y-4">
        <!-- Contact lookup -->
        <ContactEmailInput
          v-model:email="formData.email"
          v-model:contact="existingContact"
        />

        <!-- Name -->
        <UFormField label="Name" name="name">
          <UInput v-model="formData.name" />
        </UFormField>

        <!-- Demand -->
        <UFormField label="Demand" name="demand">
          <UTextarea v-model="formData.demand" rows="3" />
        </UFormField>

        <!-- Call scheduled -->
        <UFormField label="Call Scheduled" name="callScheduledAt">
          <UInput v-model="formData.callScheduledAt" type="datetime-local" />
        </UFormField>

        <!-- Options -->
        <div class="space-y-2">
          <UCheckbox v-model="formData.testFlag" label="Test lead" />
          <UCheckbox v-model="formData.sendLoginLink" label="Send login link" />
        </div>
      </div>

      <template #actions>
        <UButton
          variant="ghost"
          @click="emits('close', false)"
        >
          Cancel
        </UButton>
        <UButton
          type="submit"
          label="Create Lead"
          :loading="is(waitingFor.leads.creating)"
        />
      </template>
    </XForm>
  </USlideover>
</template>
```

---

## Validation Error Display

### Error Object Methods

```typescript
// Check if field has errors
form.errors.has('email')        // boolean

// Get first error for field
form.errors.first('email')      // string | undefined

// Get all errors for field
form.errors.get('email')        // string[] | undefined

// Check if any errors exist
form.errors.any()               // boolean

// Clear all errors
form.errors.clear()

// Clear specific field error
form.errors.forget('email')
```

### Display in Template

```vue
<!-- Using UFormField error prop -->
<UFormField
  label="Email"
  name="email"
  :error="form?.errors.first('email')"
>
  <UInput v-model="formData.email" />
</UFormField>

<!-- Manual error display -->
<div>
  <label>Email</label>
  <UInput v-model="formData.email" />
  <p v-if="form?.errors.has('email')" class="text-error text-sm">
    {{ form.errors.first('email') }}
  </p>
</div>
```

---

## useFormBuilder (Alternative)

Fluent API for form configuration:

```typescript
const form = useFormBuilder<CreateLeadData>()
  .post('/lead-management/leads')
  .data({
    name: '',
    email: '',
    demand: '',
  })
  .waiting('leads.creating')
  .resetOnSuccess(true)
  .before(({ data }) => {
    // Transform data before submit
  })
  .success(({ response }) => {
    flash.success('Lead created!')
    emits('close')
  })
  .error(({ error }) => {
    if (error instanceof ValidationError) {
      // Handle validation errors
    }
  })
  .build()

// Submit
await form.submit()

// Access state
form.processing     // boolean
form.errors         // FormErrors
form.data           // reactive data
```

---

## Form Input Components

### Text Input

```vue
<UFormField label="Name" name="name">
  <UInput v-model="formData.name" placeholder="Enter name" />
</UFormField>
```

### Textarea

```vue
<UFormField label="Description" name="description">
  <UTextarea v-model="formData.description" rows="4" />
</UFormField>
```

### Select

```vue
<UFormField label="Status" name="status">
  <USelect
    v-model="formData.status"
    :options="LeadStatus.values()"
    option-attribute="text"
  />
</UFormField>
```

### Checkbox

```vue
<UCheckbox v-model="formData.testFlag" label="Test lead" />
```

### Date/Time

```vue
<UFormField label="Scheduled" name="scheduledAt">
  <UInput v-model="formData.scheduledAt" type="datetime-local" />
</UFormField>
```

### Custom Input Component

```vue
<!-- app/components/Form/ContactEmailInput.vue -->
<script lang="ts" setup>
const email = defineModel<string>('email')
const contact = defineModel<Contact>('contact')

const contactApi = useRepository('contacts')
const searching = ref(false)
const results = ref<Contact[]>([])

const search = useDebounceFn(async () => {
  if (!email.value || email.value.length < 3) {
    results.value = []
    return
  }

  searching.value = true
  try {
    const { data } = await contactApi.search(email.value)
    results.value = data
  } finally {
    searching.value = false
  }
}, 300)

watch(email, search)

const selectContact = (c: Contact) => {
  contact.value = c
  email.value = c.email
  results.value = []
}
</script>

<template>
  <div>
    <UFormField label="Email" name="email">
      <UInput v-model="email" type="email" />
    </UFormField>

    <div v-if="results.length" class="mt-2 border rounded">
      <button
        v-for="c in results"
        :key="c.ulid"
        class="block w-full p-2 text-left hover:bg-gray-100"
        @click="selectContact(c)"
      >
        {{ c.name }} ({{ c.email }})
      </button>
    </div>
  </div>
</template>
```

---

## Edit Form Pattern

```vue
<script lang="ts" setup>
const props = defineProps<{ lead: Lead }>()
const emits = defineEmits<{ close: [success: boolean] }>()

// Initialize form with existing data
const formData = ref<UpdateLeadData>({
  name: props.lead.contact.name,
  demand: props.lead.demand,
  testFlag: props.lead.testFlag,
})

// Watch for prop changes
watch(() => props.lead, (lead) => {
  formData.value = {
    name: lead.contact.name,
    demand: lead.demand,
    testFlag: lead.testFlag,
  }
}, { immediate: true })

const updateLeadAction = updateLeadActionFactory()

const onSubmit = async (data: UpdateLeadData) => {
  return updateLeadAction(props.lead.ulid, data)
}
</script>

<template>
  <USlideover title="Edit Lead">
    <XForm
      :url="`/lead-management/leads/${lead.ulid}`"
      method="PUT"
      :data="formData"
      @submit="onSubmit"
      @success="emits('close', true)"
    >
      <!-- Form fields -->
    </XForm>
  </USlideover>
</template>
```

---

## Naming Conventions

| Type | Convention |
|------|------------|
| Form data type | `Create{Entity}Data`, `Update{Entity}Data` |
| Form ref | `formRef` |
| Submit handler | `onSubmit` |
| Success handler | `onSuccess` |

---

## Related Skills

- **[nuxt-components](../../nuxt-components/SKILL.md)** - Component patterns
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Actions for form submission
- **[nuxt-errors](../../nuxt-errors/SKILL.md)** - Error handling
