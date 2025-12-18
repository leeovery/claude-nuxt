# Real-time Patterns

## Configuration

### Environment Variables

```bash
# .env
NUXT_PUBLIC_ECHO_KEY=your-pusher-key
NUXT_PUBLIC_ECHO_HOST=echo.example.com
NUXT_PUBLIC_ECHO_SCHEME=https
NUXT_PUBLIC_ECHO_PORT=443
```

### Nuxt Config

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-laravel-echo'],

  runtimeConfig: {
    public: {
      echo: {
        key: undefined,
        host: undefined,
        scheme: undefined,
        port: undefined,
      },
    },
  },
})
```

---

## Constants

### Channel Names

```typescript
// app/constants/channels.ts

// Collection channels
export const Leads = 'leads'
export const Contacts = 'contacts'
export const Evaluations = 'evaluations'

// Resource channels (with parameter)
export const Lead = 'lead.{lead}'
export const Contact = 'contact.{contact}'
export const Conversation = 'conversation.{conversation}'

// User-specific channels
export const UserNotifications = 'user.{user}.notifications'
```

### Event Names

```typescript
// app/constants/events.ts

// Lead events
export const LeadCreated = 'LeadCreated'
export const LeadUpdated = 'LeadUpdated'
export const LeadDeleted = 'LeadDeleted'

// SecureLink events
export const SecureLinkSent = 'SecureLinkSent'
export const SecureLinkDelivered = 'SecureLinkDelivered'
export const SecureLinkBounced = 'SecureLinkBounced'

// Conversation events
export const MessageReceived = 'MessageReceived'
export const ConversationEnded = 'ConversationEnded'

// System events
export const NotificationReceived = 'NotificationReceived'
```

---

## useRealtime Composable

```typescript
const {
  privateChannel,    // Private channel subscription
  presenceChannel,   // Presence channel (with user info)
  leaveChannel,      // Unsubscribe from channel
} = useRealtime()
```

---

## Private Channel Pattern

### Basic Subscription

```typescript
import { Leads, LeadCreated, LeadUpdated } from '~/constants/channels'
import { LeadCreated, LeadUpdated } from '~/constants/events'

const { privateChannel } = useRealtime()

// Subscribe to private channel
const channel = privateChannel(Leads)

// Listen for single event
channel.on(LeadCreated, (event) => {
  console.log('New lead:', event.lead)
  refresh()
})

// Listen for multiple events
channel.on([LeadCreated, LeadUpdated], (event) => {
  refresh()
})
```

### Channel with Parameter

```typescript
import { Lead, LeadUpdated, LeadDeleted } from '~/constants'

// Channel with dynamic ID
const channel = privateChannel(Lead, lead.ulid)
// Subscribes to: lead.{ulid}

channel.on([LeadUpdated, LeadDeleted], refresh)
```

### Cleanup on Unmount

```typescript
const { privateChannel, leaveChannel } = useRealtime()

// Subscribe
const channel = privateChannel(Lead, lead.ulid)
channel.on(LeadUpdated, refresh)

// Cleanup
onUnmounted(() => {
  leaveChannel(Lead, lead.ulid)
})
```

---

## Complete Page Example

```vue
<!-- app/pages/leads/[ulid].vue -->
<script lang="ts" setup>
import { Lead, LeadUpdated, LeadDeleted } from '~/constants/channels'
import { LeadUpdated, LeadDeleted } from '~/constants/events'

const route = useRoute()
const router = useRouter()
const ulid = computed(() => route.params.ulid as string)

// Query
const getLeadQuery = getLeadQueryFactory()
const { data: lead, refresh } = getLeadQuery(ulid)

// Real-time updates
const { privateChannel, leaveChannel } = useRealtime()

// Subscribe to lead channel
const channel = privateChannel(Lead, ulid.value)

// Handle updates
channel.on(LeadUpdated, () => {
  refresh()
})

// Handle deletion
channel.on(LeadDeleted, () => {
  flash.info('This lead has been deleted')
  router.push('/leads')
})

// Cleanup on unmount
onUnmounted(() => {
  leaveChannel(Lead, ulid.value)
})
</script>
```

---

## List Page with Real-time

```vue
<!-- app/pages/leads/index.vue -->
<script lang="ts" setup>
import { Leads, LeadCreated, LeadUpdated, LeadDeleted } from '~/constants'

// Query
const getLeadsQuery = getLeadsQueryFactory()
const { data: leads, refresh } = getLeadsQuery(filters)

// Real-time updates for list
const { privateChannel } = useRealtime()

privateChannel(Leads).on(
  [LeadCreated, LeadUpdated, LeadDeleted],
  refresh
)
</script>
```

---

## Composable with Real-time

```typescript
// app/composables/useRemainingJobCount.ts
import { MatchJobs, MatchJobCreated, MatchJobComplete } from '~/constants'

export default function useRemainingJobCount() {
  const remainingJobCount = useState<number>('remaining-job-count', () => 0)
  const jobApi = useRepository('jobs')
  const { privateChannel } = useRealtime()

  const fetchJobCount = async () => {
    const { data } = await jobApi.list({ filter: { status: 'pending' } })
    remainingJobCount.value = data.length
  }

  const init = () => {
    // Initial fetch
    fetchJobCount()

    // Real-time updates
    privateChannel(MatchJobs).on(
      [MatchJobCreated, MatchJobComplete],
      fetchJobCount
    )
  }

  return { remainingJobCount, init }
}
```

---

## Presence Channel Pattern

For channels that track online users:

```typescript
const { presenceChannel } = useRealtime()

// Join presence channel
const channel = presenceChannel('chat.{room}', roomId)

// Handle events
channel.on('here', (users) => {
  // Initial list of users in channel
  onlineUsers.value = users
})

channel.on('joining', (user) => {
  // User joined
  onlineUsers.value.push(user)
})

channel.on('leaving', (user) => {
  // User left
  onlineUsers.value = onlineUsers.value.filter(u => u.id !== user.id)
})

// Listen for custom events too
channel.on('MessageSent', (event) => {
  messages.value.push(event.message)
})
```

---

## Error Handling

```typescript
const { privateChannel } = useRealtime()

const channel = privateChannel(Leads)

channel.error((error) => {
  console.error('Channel error:', error)
  flash.error('Real-time connection failed')
})
```

---

## Best Practices

### 1. Always Clean Up

```typescript
// Store channel reference for cleanup
const channel = privateChannel(Lead, id)
channel.on(LeadUpdated, refresh)

onUnmounted(() => {
  leaveChannel(Lead, id)
})
```

### 2. Use Constants

```typescript
// DON'T: Magic strings
privateChannel('leads').on('LeadCreated', ...)

// DO: Use constants
privateChannel(Leads).on(LeadCreated, ...)
```

### 3. Debounce Refreshes

```typescript
import { useDebounceFn } from '@vueuse/core'

const debouncedRefresh = useDebounceFn(refresh, 300)

channel.on([LeadCreated, LeadUpdated], debouncedRefresh)
```

### 4. Check Connection

```typescript
const { isConnected } = useRealtime()

// Show connection status
<UBadge v-if="!isConnected" color="warning">Reconnecting...</UBadge>
```

---

## Directory Structure

```
app/
└── constants/
    ├── channels.ts    # Channel name constants
    └── events.ts      # Event name constants
```

---

## Related Skills

- **[nuxt-config](../../nuxt-config/SKILL.md)** - Echo configuration
- **[nuxt-composables](../../nuxt-composables/SKILL.md)** - useRealtime
- **[nuxt-features](../../nuxt-features/SKILL.md)** - Real-time in queries
