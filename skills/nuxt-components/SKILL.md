---
name: nuxt-components
description: Vue component patterns with Composition API and script setup. Use when creating components, understanding script setup order convention, organizing component directories, or implementing component patterns like slideovers, modals, and tables.
---

# Nuxt Components

Vue 3 Composition API components with standardized organization and patterns.

## Core Concepts

**[components.md](references/components.md)** - Script setup order, patterns, organization

## Directory Structure

```
components/
├── Common/       # Shared utilities (Copyable, LoadingLine)
├── Detail/       # Entity detail views (LeadDetail)
├── Form/         # Reusable form inputs (ContactEmailInput)
├── Modals/       # Confirmation/action modals
├── Nav/          # Navigation elements
├── Slideovers/   # Slideout panels (CreateLeadSlideover)
├── TabSections/  # Tab content sections
└── Tables/       # Data tables (LeadsTable)
```

## Script Setup Order

```vue
<script lang="ts" setup>
// 1. Imports
import createLeadActionFactory from '~/features/leads/actions/create-lead-action'

// 2. Props & Emits
const props = defineProps<{ contact?: Contact }>()
const emits = defineEmits<{ close: [success: boolean] }>()

// 3. Composables
const flash = useFlash()

// 4. Injections
const slideover = inject(SlideoverKey)

// 5. Refs
const formRef = useTemplateRef('formRef')

// 6. Toggles
const isOpen = ref(false)

// 7. Reactive props
const formData = ref<CreateLeadData>({ email: '', name: '' })

// 8. Computed
const canSubmit = computed(() => formData.value.email && formData.value.name)

// 9. Fetch + queries
const { data: leads, refresh } = getLeadsQuery(filters)

// 10. Builders (action/mutation factories)
const createLeadAction = createLeadActionFactory()

// 11. Watchers
watch(existingContact, (c) => { formData.value.name = c?.name || '' })

// 12. Methods
const onSubmit = async (data) => { await createLeadAction(data) }

// 13. Real-time listeners
privateChannel(Leads).on(LeadCreated, refresh)

// 14. Provides
provide(SlideoverKey, { isOpen })

// 15. Lifecycles
onMounted(() => { /* ... */ })
</script>
```
