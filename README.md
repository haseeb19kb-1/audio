# ZEVO LIVE — Production Audio Streaming Platform

An international, scalable audio live-streaming platform inspired by BIGO Live.
**Audio streaming only** — high-performance, crystal-clear voice communication with real-time microtransactions and creator economy.

---

## 1. System Architecture

- **Mobile App**: React Native + Expo SDK 51 + TypeScript (`apps/mobile`)
- **Admin Dashboard**: Next.js 14 + TypeScript (`apps/admin`)
- **Database & Auth**: Supabase PostgreSQL + Auth + Storage (`supabase/`)
- **Security**: 100% strict PostgreSQL Row-Level Security (RLS) with default-deny policies
- **Real-time Voice**: LiveKit Cloud SFU (WebRTC Opus voice, dynamic mic seats)
- **Cache & Rate Limiting**: Upstash Redis (isolated keyspaces, sliding-window rate limits)
- **Monorepo Packages**:
  - `@zevo/shared-types` (`packages/shared-types`)
  - `@zevo/validation` (`packages/validation`)
  - `@zevo/config` (`packages/config`)

---

## 2. Monorepo Directory Layout

```text
zevo-live/
├── apps/
│   ├── mobile/                 # React Native + Expo Audio Streaming App
│   │   ├── src/
│   │   │   ├── components/     # Avatar, MicSeatGrid, GiftModal, GiftOverlay, RoomChat
│   │   │   ├── screens/        # HomeScreen, AudioRoomScreen, WalletScreen, ProfileScreen
│   │   │   ├── services/       # Supabase, LiveKit WebRTC, Virtual Gifts
│   │   │   └── state/          # Zustand global stores (auth, active room)
│   │   ├── App.tsx             # Main mobile entry with tab & full-screen stage navigators
│   │   └── package.json
│   └── admin/                  # Next.js 14 Management & Moderation Dashboard
│       ├── src/app/
│       │   ├── page.tsx        # Overview metrics & active room table
│       │   ├── rooms/          # Room kill-switch & inspection
│       │   ├── payouts/        # Creator diamond withdrawal approvals
│       │   ├── reports/        # Community safety & abuse report triage
│       │   └── audit/          # Immutable security audit logs
│       └── package.json
├── packages/
│   ├── shared-types/           # Common TypeScript interfaces & database schemas
│   ├── validation/             # Pure input & payment validators
│   └── config/                 # Tier perks, room categories, audio bitrate configs
├── supabase/
│   ├── migrations/             # Relational schema, RLS policies, and atomic RPCs
│   ├── functions/              # Edge Functions: livekit-token, send-gift, iap-webhook, withdrawal
│   ├── tests/                  # Automated RLS security test runner
│   └── seed.sql                # Default gift catalog & coin packages
├── docs/
│   ├── architecture.md         # System Architecture & Flow Specifications
│   ├── security.md             # Security Policies, RLS Models & Threat Model
│   └── api.md                  # REST & RPC Interface Contracts
├── .env.example
├── package.json
└── README.md
```

---

## 3. Getting Started

### 3.1 Prerequisites
- **Node.js**: v20.x or higher (Node 20.18 LTS is pre-configured in environment)
- **npm**: v10.x or higher

### 3.2 Environment Setup
Copy the template environment file:
```bash
cp .env.example .env
```
Fill in your credentials for Supabase, LiveKit Cloud, and Upstash Redis.

---

## 4. Running the Applications

### 4.1 Running the Automated Security Test Suite
Verify that all 21 database tables enforce Row-Level Security, balance non-negative check constraints, pessimistic row locks (`FOR UPDATE`), and anti-double-spend logic:
```bash
npm run test:security
```

### 4.2 Running the Mobile Audio Streaming App
```bash
npm run mobile:start
```
- Press `w` to open in web browser preview.
- Scan the QR code with Expo Go on iOS or Android to run on physical devices.

### 4.3 Running the Admin Moderation Dashboard
```bash
npm run admin:dev
```
Open [http://localhost:3001](http://localhost:3001) in your browser.

---

## 5. Security & Financial Guarantees

1. **Zero Client Trust**: Clients cannot modify wallet balances, diamond earnings, roles, or prices.
2. **Pessimistic Locking**: `process_gift_transaction` locks sender wallet and receiver host earnings rows `FOR UPDATE` inside an isolated transaction.
3. **Idempotency**: Webhook events and gift dispatch require unique idempotency keys to prevent duplicate deductions.
4. **LiveKit Token Security**: WebRTC room tokens are minted exclusively through Supabase Edge Functions with a 15-minute TTL after checking active room bans.
