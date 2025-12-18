# Component Patterns

## Script Setup Order Convention

Always follow this order in `<script setup>`:

```vue
<script lang="ts" setup>
// 1. Imports
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'
import type { CreateLeadData } from '~/features/leads/mutations/create-lead-mutation'

// 2. Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits<{
  close: [success: boolean]
  update: [lead: Lead]
}>()

// 3. Composables and definitions
const flash = useFlash()
const { filters } = useReactiveFilters()
const { start, stop, waitingFor } = useWait()

// 4. Injections
const slideover = inject(SlideoverKey)

// 5. Refs (template refs)
const formRef = useTemplateRef('formRef')

// 6. Toggles (boolean refs)
const isOpen = ref(false)
const showAdvanced = ref(false)

// 7. Reactive props (complex state)
const formData = ref<CreateLeadData>({
  email: '',
  name: '',
  demand: '',
})
const existingContact = ref<Contact>()

// 8. Computed props
const canSubmit = computed(() =>
  formData.value.email && formData.value.name
)
const isEditing = computed(() => !!props.lead)

// 9. Fetch + associated calls (queries)
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh, isLoading } = getLeadsQuery(filters)

// 10. Builders (action/mutation factories)
const createLeadAction = createLeadActionFactory()
const deleteLeadAction = deleteLeadActionFactory()

// 11. Watchers
watch(existingContact, (contact) => {
  formData.value.name = contact?.name || ''
})

watch(() => props.lead, (lead) => {
  if (lead) populateForm(lead)
}, { immediate: true })

// 12. Methods
const onSubmit = async (data: CreateLeadData) => {
  await createLeadAction(data)
  emits('close', true)
}

const resetForm = () => {
  formData.value = { email: '', name: '', demand: '' }
}

// 13. Real-time listeners
const { privateChannel } = useRealtime()
privateChannel(Leads).on(LeadCreated, refresh)
privateChannel(Leads).on(LeadUpdated, refresh)

// 14. Provides
provide(SlideoverKey, { isOpen })

// 15. Lifecycles
onMounted(() => {
  // Initialize component
})

onUnmounted(() => {
  // Cleanup
})
</script>
```

---

## Directory Organization

Components organized by **UI pattern**, not domain:

```
components/
├── Common/              # Shared generic components
│   ├── TeriLogo.vue
│   ├── Copyable.vue
│   └── LoadingLine.vue
├── Detail/              # Entity detail view components
│   ├── LeadDetail.vue
│   └── ContactDetail.vue
├── Form/                # Reusable form input components
│   ├── ContactEmailInput.vue
│   └── LeadDemandRow.vue
├── Modals/              # Modal dialogs
│   ├── DeleteLeadModal.vue
│   ├── DeleteContactModal.vue
│   └── ConfirmActionModal.vue
├── Nav/                 # Navigation components
│   ├── UserMenu.vue
│   └── Sidebar.vue
├── Slideovers/          # Slideout panel components
│   ├── CreateLeadSlideover.vue
│   └── UpdateLeadSlideover.vue
├── TabSections/         # Tab content components
│   ├── LeadInsightsTab.vue
│   └── ContactLeadsTab.vue
├── Tables/              # Table components
│   ├── LeadsTable.vue
│   └── ContactsTable.vue
└── Feedback/            # Domain-specific (optional)
    └── CategoryRatingField.vue
```

---

## Component Patterns

### Slideover Component

```vue
<!-- app/components/Slideovers/CreateLeadSlideover.vue -->
<script lang="ts" setup>
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'
import type { CreateLeadData } from '~/features/leads/mutations/create-lead-mutation'

// Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits<{ close: [success: boolean] }>()

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
  formData.value.name = contact?.name || ''
})

// Methods
const createLeadAction = createLeadActionFactory()

const onSubmit = (data: CreateLeadData) => createLeadAction(data)
const onSuccess = () => emits('close', true)
</script>

<template>
  <USlideover title="Create Lead">
    <XForm
      ref="formRef"
      url="/lead-management/leads"
      method="POST"
      :data="formData"
      @submit="onSubmit"
      @success="onSuccess"
    >
      <div class="space-y-4">
        <ContactEmailInput
          v-model:email="formData.email"
          v-model:contact="existingContact"
        />

        <UFormField label="Name" name="name">
          <UInput v-model="formData.name" />
        </UFormField>

        <UFormField label="Demand" name="demand">
          <UTextarea v-model="formData.demand" />
        </UFormField>

        <UCheckbox v-model="formData.testFlag" label="Test lead" />
      </div>

      <template #actions>
        <UButton type="submit" label="Create Lead" />
      </template>
    </XForm>
  </USlideover>
</template>
```

### Table Component

```vue
<!-- app/components/Tables/LeadsTable.vue -->
<script lang="ts" setup>
import { leadsColumnBuilder } from '~/tables/leads'
import type { Row } from '@tanstack/vue-table'

const props = defineProps<{
  leads: Lead[]
  loading?: boolean
  fetching?: boolean
}>()

const emits = defineEmits<{
  edit: [lead: Lead]
  delete: [leads: Lead[]]
}>()

// Get columns from builder
const columns = leadsColumnBuilder.all()

// Row actions
const rowActions = computed(() => (row: Row<Lead>) => [
  { label: 'View lead', to: `/leads/${row.original.ulid}` },
  { label: 'View contact', to: `/contacts/${row.original.contact.ulid}` },
  { label: 'Edit lead', onSelect: () => emits('edit', row.original) },
  { label: 'Delete lead', onSelect: () => emits('delete', [row.original]) },
])
</script>

<template>
  <XTable
    :data="leads"
    :columns="columns"
    :loading="loading"
    :fetching="fetching"
    :row-actions="rowActions"
    row-id="ulid"
  />
</template>
```

### Detail Component

```vue
<!-- app/components/Detail/LeadDetail.vue -->
<script lang="ts" setup>
const props = defineProps<{ lead: Lead }>()

const emits = defineEmits<{
  edit: []
  delete: []
}>()
</script>

<template>
  <XCard>
    <div class="space-y-4">
      <div class="flex justify-between">
        <h2 class="text-lg font-semibold">{{ lead.contact.name }}</h2>
        <UBadge :color="lead.status.color()">
          {{ lead.status.text }}
        </UBadge>
      </div>

      <dl class="grid grid-cols-2 gap-4">
        <div>
          <dt class="text-sm text-gray-500">Email</dt>
          <dd>{{ lead.contact.email }}</dd>
        </div>
        <div>
          <dt class="text-sm text-gray-500">Demand</dt>
          <dd>{{ lead.demand }}</dd>
        </div>
        <div>
          <dt class="text-sm text-gray-500">Created</dt>
          <dd>{{ lead.createdAt.format() }}</dd>
        </div>
      </dl>

      <div class="flex gap-2">
        <UButton @click="emits('edit')">Edit</UButton>
        <UButton color="error" @click="emits('delete')">Delete</UButton>
      </div>
    </div>
  </XCard>
</template>
```

### Modal Component

```vue
<!-- app/components/Modals/DeleteLeadModal.vue -->
<script lang="ts" setup>
import deleteLeadActionFactory from '~/features/leads/actions/delete-lead-action'

const props = defineProps<{ lead: Lead }>()
const emits = defineEmits<{ close: [deleted: boolean] }>()

const deleteLeadAction = deleteLeadActionFactory()
const { is, waitingFor } = useWait()

const onConfirm = async () => {
  await deleteLeadAction(props.lead)
  emits('close', true)
}
</script>

<template>
  <UModal title="Delete Lead">
    <p>Are you sure you want to delete this lead?</p>
    <p class="text-sm text-gray-500">{{ lead.contact.name }}</p>

    <template #footer>
      <UButton variant="ghost" @click="emits('close', false)">
        Cancel
      </UButton>
      <UButton
        color="error"
        :loading="is(waitingFor.lead.deleting(lead.ulid))"
        @click="onConfirm"
      >
        Delete
      </UButton>
    </template>
  </UModal>
</template>
```

---

## Props & Emits

### Typed Props

```typescript
// Simple props
const props = defineProps<{
  lead: Lead
  loading?: boolean
}>()

// With defaults
const props = withDefaults(defineProps<{
  lead: Lead
  loading?: boolean
}>(), {
  loading: false,
})
```

### Typed Emits

```typescript
// Object syntax with types
const emits = defineEmits<{
  close: [success: boolean]
  update: [lead: Lead]
  delete: [leads: Lead[]]
}>()

// Usage
emits('close', true)
emits('update', updatedLead)
```

---

## Template Refs

```vue
<script lang="ts" setup>
// Modern way (Nuxt 3.13+)
const formRef = useTemplateRef('formRef')

// Access
formRef.value?.submit()
</script>

<template>
  <XForm ref="formRef" />
</template>
```

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| File | PascalCase | `CreateLeadSlideover.vue` |
| Component | PascalCase | `<CreateLeadSlideover />` |
| Props | camelCase | `:loading="isLoading"` |
| Emits | camelCase | `@close="handleClose"` |

---

## Common Patterns

### Conditional Rendering

```vue
<template>
  <!-- v-if for expensive components -->
  <LeadDetail v-if="lead" :lead="lead" />

  <!-- v-show for frequent toggles -->
  <div v-show="showFilters">...</div>
</template>
```

### Loading States

```vue
<template>
  <!-- Global loading -->
  <LoadingLine v-if="isFetching" />

  <!-- Skeleton -->
  <USkeleton v-if="isLoading" class="h-64" />

  <!-- Content -->
  <LeadsTable v-else :leads="leads" />
</template>
```

### Permission-Based

```vue
<template>
  <UButton v-if="can(CreateLead)" @click="openCreate">
    Create Lead
  </UButton>
</template>
```

---

## Related Skills

- **[nuxt-forms](../../nuxt-forms/SKILL.md)** - Form patterns
- **[nuxt-tables](../../nuxt-tables/SKILL.md)** - Table patterns
- **[nuxt-pages](../../nuxt-pages/SKILL.md)** - Page-level patterns
