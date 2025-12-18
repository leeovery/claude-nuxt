---
name: nuxt-realtime
description: Real-time features with Laravel Echo and WebSockets. Use when subscribing to channels, listening for events, implementing live updates, or managing channel subscriptions.
---

# Nuxt Real-time

WebSocket real-time updates via Laravel Echo.

## Core Concepts

**[realtime.md](references/realtime.md)** - Complete real-time patterns

## Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-laravel-echo'],

  runtimeConfig: {
    public: {
      echo: {
        key: undefined,    // NUXT_PUBLIC_ECHO_KEY
        host: undefined,   // NUXT_PUBLIC_ECHO_HOST
        scheme: undefined, // NUXT_PUBLIC_ECHO_SCHEME
        port: undefined,   // NUXT_PUBLIC_ECHO_PORT
      },
    },
  },
})
```

## Channel Subscriptions

```typescript
const { privateChannel, presenceChannel, leaveChannel } = useRealtime()

// Subscribe to channel
const channel = privateChannel('leads.{id}', leadId)

// Listen for events
channel.on('LeadUpdated', (event) => {
  refresh()
})

// Multiple events
channel.on(['LeadCreated', 'LeadUpdated', 'LeadDeleted'], refresh)

// Cleanup
onUnmounted(() => {
  leaveChannel('leads.{id}', leadId)
})
```

## Constants

```typescript
// app/constants/channels.ts
export const Leads = 'leads'
export const Lead = 'lead.{lead}'

// app/constants/events.ts
export const LeadCreated = 'LeadCreated'
export const LeadUpdated = 'LeadUpdated'
```
