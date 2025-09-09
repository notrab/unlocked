# 🎮

Boost user engagement with a gamified XP system built for Next.js. Get users hooked with tiers, achievements, and progress tracking.

## 🚀 Installation

```bash
npm install @notrab/xp
# or
yarn add @notrab/xp
```

## ⚡ Quick Setup

### 1. Create your XP configuration

```typescript
// xp.config.ts
export const xpConfig = {
  tiers: [
    { name: 'Newcomer', threshold: 0, color: '#94a3b8', badge: '🌱' },
    { name: 'Explorer', threshold: 100, color: '#3b82f6', badge: '🔍' },
    { name: 'Expert', threshold: 500, color: '#8b5cf6', badge: '⭐' },
    { name: 'Master', threshold: 1500, color: '#f59e0b', badge: '👑' }
  ],
  actions: {
    'profile_complete': { xp: 50, label: 'Complete Profile' },
    'first_purchase': { xp: 200, label: 'First Purchase' },
    'feature_discovery': { xp: 25, label: 'Try New Feature' },
    'daily_login': { xp: 10, label: 'Daily Login' },
    'referral': { xp: 100, label: 'Refer a Friend' },
    'tutorial_complete': { xp: 75, label: 'Finish Tutorial' }
  }
}
```

### 2. Setup your database adapter

```typescript
// lib/xp-adapter.ts
import { prismaAdapter } from '@notrab/xp/adapters'
import { prisma } from './prisma'

export const xpAdapter = prismaAdapter(prisma)

// Add to your User model:
// model User {
//   id   String @id @default(cuid())
//   xp   Int    @default(0)
//   ...
// }
```

### 3. Wrap your app with XPProvider

```tsx
// app/layout.tsx or _app.tsx
import { XPProvider } from '@notrab/xp'
import { xpConfig } from '../xp.config'
import { xpAdapter } from '../lib/xp-adapter'

export default function RootLayout({ children }) {
  const { user } = useAuth() // Your auth solution

  return (
    <html>
      <body>
        <XPProvider 
          config={xpConfig} 
          adapter={xpAdapter}
          userId={user?.id}
        >
          {children}
          <XPNotifications />
        </XPProvider>
      </body>
    </html>
  )
}
```

### 4. Add XP components and actions

```tsx
// components/Dashboard.tsx
import { XPProgressBar, useXP, useXPAction } from '@notrab/xp'

export function Dashboard() {
  const { xp, tier, isLoading } = useXP()
  const triggerProfileComplete = useXPAction('profile_complete')

  if (isLoading) return <div>Loading...</div>

  return (
    <div>
      <h1>Welcome back!</h1>
      
      {/* Show user's progress */}
      <XPProgressBar />
      <p>Level: {tier?.name} ({xp} XP)</p>
      
      {/* Trigger XP on user action */}
      <button onClick={triggerProfileComplete}>
        Complete Profile (+50 XP)
      </button>
    </div>
  )
}
```

## 🎯 Real-World Examples

### E-commerce Onboarding
```typescript
// When user completes checkout
const { addXP } = useXP()

const handleCheckout = async () => {
  await processPayment()
  await addXP('first_purchase') // +200 XP, level up notification!
}
```

### SaaS Feature Adoption
```typescript
// Track when users try new features
const { addXP } = useXP()

const handleFeatureClick = (feature: string) => {
  // Your feature logic
  addXP('feature_discovery') // +25 XP per new feature
}
```

### Social Platform Engagement
```typescript
// Reward daily activity
useEffect(() => {
  const lastLogin = localStorage.getItem('lastLogin')
  const today = new Date().toDateString()
  
  if (lastLogin !== today) {
    addXP('daily_login') // +10 XP daily
    localStorage.setItem('lastLogin', today)
  }
}, [])
```

## 🎨 Built-in Components

```tsx
import { 
  XPProgressBar,    // Shows current progress to next tier
  XPBadge,          // Display current tier badge
  XPLeaderboard,    // Show top users (optional)
  XPNotifications   // Toast notifications for XP gains
} from '@notrab/xp'

<XPProgressBar showXP showTier animated />
<XPBadge size="large" showTier />
<XPLeaderboard limit={10} />
```

## 🔧 API Routes (Auto-generated)

The package automatically creates these API endpoints:

- `GET /api/xp/[userId]` - Get user's current XP
- `GET /api/tier/[userId]` - Get user's current tier  
- `POST /api/xp/add` - Add XP for an action
- `GET /api/leaderboard` - Get top users

## 🎉 That's it!

Your users will now see:
- ✅ Progress bars showing their advancement
- ✅ Satisfying notifications when they gain XP  
- ✅ Clear tiers to work towards
- ✅ Gamified onboarding that drives engagement

Built with ❤️ for the Next.js community
