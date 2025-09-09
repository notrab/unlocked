# XP for Saas

Add a game-like XP system to your SaaS app (think Fortnite XP, Xbox Achievements, COD Battle Pass), with a clean, NextAuth-inspired developer experience.

Users earn XP for actions (profile complete, purchase, daily login, referrals, etc.), progress through tiers, and unlock rewards.
Bring your own database (Redis, Prisma, Postgres, etc.) via adapters.


## 1. Install

```bash
npm install @notrab/xp
```

## 2. Define your config

Create a `xp.ts` file in your app:

```ts
// lib/xp.ts
import { createXP, XPRoutes } from '@notrab/xp/server'
import { createRedisAdapter } from '@notrab/xp/adapters/redis-adapter'
import { auth } from '@/lib/auth' // NextAuth, Clerk, etc.
import { redis } from './redis'

// Define tiers, actions, hooks
export const xp = createXP({
  config: {
    tiers: [
      { name: 'Newcomer', threshold: 0, color: '#94a3b8', badge: '🌱' },
      { name: 'Explorer', threshold: 100, color: '#3b82f6', badge: '🔍' },
      { name: 'Expert', threshold: 500, color: '#8b5cf6', badge: '⭐' },
      { name: 'Master', threshold: 1500, color: '#f59e0b', badge: '👑' },
    ],
    actions: {
      profile_complete: { xp: 50, label: 'Complete Profile', unique: true },
      first_purchase: { xp: 200, label: 'First Purchase', unique: true },
      daily_login: { xp: 10, label: 'Daily Login', cooldownSeconds: 86400 },
    },
    hooks: {
      onAction: async ({ userId, actionKey, amount }) => {
        console.log(`[xp] ${userId} +${amount} XP for ${actionKey}`)
      },
      onLevelUp: async ({ userId, toTier }) => {
        if (toTier.name === 'Explorer') {
          await sendDiscountEmail(userId, 'WELCOME10')
        }
        if (toTier.name === 'Expert') {
          await promotePlanUpgrade(userId)
        }
      },
    },
  },
  adapter: createRedisAdapter(redis),
  getUserId: async (req) => {
    const session = await auth()
    return session?.user?.id ?? null
  },
})

// Export API routes
export const { GET, POST } = XPRoutes(xp)
```

## 3. API Route

Create `app/api/xp/route.ts`:

```ts
export { GET, POST } from '@/lib/xp'
```

## 4. Wrap your App

```tsx
// app/layout.tsx
import { XPProvider } from '@notrab/xp'
import { xp } from '@/lib/xp'
import { useAuth } from '@clerk/nextjs'

export default function RootLayout({ children }) {
  const { user } = useAuth()
  return (
    <html>
      <body>
        <XPProvider config={xp.config} userId={user?.id}>
          {children}
        </XPProvider>
      </body>
    </html>
  )
}
```

## 5. Use in components

```tsx
import { XPProgressBar, useXP, useXPAction } from '@notrab/xp'

export function Dashboard() {
  const { xp, tier, isLoading } = useXP()
  const triggerProfileComplete = useXPAction('profile_complete')

  if (isLoading) return <div>Loading...</div>

  return (
    <div>
      <h1>Welcome back!</h1>
      <XPProgressBar />
      <p>Level: {tier} ({xp} XP)</p>
      <button onClick={() => triggerProfileComplete()}>
        Complete Profile (+50 XP)
      </button>
    </div>
  )
}
```
