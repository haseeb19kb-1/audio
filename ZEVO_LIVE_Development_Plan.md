# ZEVO LIVE — Audio Streaming App Development Plan

## 1. Goal

Build a scalable international audio-live application inspired by BIGO Live. Audio streaming only; no video streaming.

## 2. Technology Stack

| Component | Technology |
|---|---|
| Mobile app | React Native + Expo + TypeScript |
| Authentication | Supabase Auth |
| Database | Supabase PostgreSQL |
| Database security | Strict Row-Level Security (RLS) |
| File storage | Supabase Storage |
| Backend logic | Supabase Edge Functions |
| Real-time audio | LiveKit Cloud |
| Cache and rate limiting | Upstash Redis |
| Push notifications | Expo Notifications + FCM/APNs |
| Mobile digital payments | Apple In-App Purchase + Google Play Billing |
| Admin dashboard | Next.js + TypeScript |
| Monitoring | Sentry + backend logs |

## 3. Essential Features

1. **Authentication:** Email, phone OTP, supported social login, account recovery and session management.
2. **User profiles:** Avatar, unique ID, bio, country, level, followers and following.
3. **Audio rooms:** Create/join rooms, discovery, public/private rooms, invitations, multiple speakers, listeners, mute/unmute, host/co-host, speaker requests and moderators.
4. **Virtual gifts:** Gift catalog, animations, sender/receiver details, gift history and atomic coin deduction.
5. **Wallet:** Coin packages, balance, transaction history, refunds and reconciliation.
6. **Host earnings:** Separate earnings ledger, eligibility checks, withdrawal requests and payout tracking.
7. **Social:** Follow, chat, likes, notifications, blocking and reporting.
8. **Safety:** Room moderation, user bans, spam prevention, age requirements, appeals and audit logs.
9. **Admin dashboard:** User management, room monitoring, reports, gifts, coin packages, transactions, payouts and role-based permissions.

## 4. Pricing Model

### Proposed user-facing pricing

| Plan | Proposed price | Features |
|---|---:|---|
| Free | $0/month | Join rooms, listen, follow, basic profile and gifts using available coins |
| VIP | $4.99/month | VIP badge, profile themes and extra customization |
| VIP Plus | $9.99/month | Additional badges, themes and eligible room perks |
| Coin packages | $0.99–$99.99 | Virtual gifts and other permitted in-app features |

These are starting proposals in USD, not finalized regional prices. Configure localized prices by country and platform. Virtual coins are digital goods; follow the applicable Apple and Google billing rules and any valid regional exceptions. Never credit coins based only on a client-side payment success message.

### Infrastructure budget approach

| Service | Development | Production |
|---|---|---|
| Supabase | Free tier | Consider Pro, commonly starting around $25/month; verify current pricing |
| Upstash Redis | Free allowance | Usage-based paid tier |
| LiveKit Cloud | Free allowance, subject to limits | Usage-based audio and participant costs |
| Expo | Free starting tier | Paid EAS plan if needed |
| Sentry | Free starting tier | Paid plan as monitoring volume grows |

Treat these as planning references, not a cost quote. Check current provider pricing and estimate costs from expected concurrent listeners, audio minutes, bandwidth, database usage, storage and monitoring. Free tiers may not be suitable for production.

## 5. Database Structure

Use PostgreSQL migrations, UUID primary keys, foreign keys, indexes, constraints and timestamps.

- `profiles` — public profile information.
- `user_private` — private account data.
- `user_roles` — admin, moderator and support permissions.
- `rooms` — room metadata and ownership.
- `room_members` — membership, roles and speaking status.
- `room_bans` — room-specific bans.
- `follows` — follower relationships.
- `messages` — room and private messages.
- `gifts` — gift catalog and asset references.
- `wallets` — coin balances.
- `wallet_transactions` — immutable coin ledger.
- `gift_transactions` — sender, receiver, room and gift details.
- `host_earnings` — separate host earnings ledger.
- `withdrawal_requests` — payout status and review.
- `coin_packages` — localized package definitions.
- `payment_events` — verified provider events and idempotency keys.
- `reports`, `blocks`, `notifications`, `audit_logs`.

Financial operations must use database transactions, constraints and server-side authorization. Clients must never directly update wallet balances, host earnings, roles or payment status.

## 6. Security Requirements — Mandatory

- Enable RLS on every exposed table; default-deny access.
- Write explicit `SELECT`, `INSERT`, `UPDATE` and `DELETE` policies where required.
- Users may access only their own private data and authorized room resources.
- Enforce admin and moderator permissions on the server.
- Keep Supabase service-role keys, payment secrets and LiveKit API secrets on trusted servers only.
- Issue short-lived LiveKit tokens through authenticated Edge Functions after checking room permissions.
- Use private Storage buckets where appropriate, strict object ownership policies and signed URLs.
- Validate requests server-side; never trust client-supplied prices, coin amounts, roles or earnings.
- Apply rate limits to OTP, login, room creation, messages, gifts, APIs and withdrawals.
- Use idempotency keys and verified webhooks to prevent duplicate payment events or coin credits.
- Add audit logs, abuse detection, secret management and tested backups.
- Test RLS using multiple users, unauthenticated requests and privileged roles before launch.

## 7. Supabase Connection Pooling

- Use Supabase's supported connection pooler for server-side PostgreSQL connections.
- Use the Supabase client SDK in React Native; never expose database credentials.
- Keep transactions short and configure pooling for the selected connection mode.
- Avoid long-running transactions, unbounded queries and excessive connections.
- Add indexes for room discovery, membership lookups, user IDs and transaction history.
- Apply all schema and policy changes through versioned migrations.

## 8. Upstash Redis Isolation

- Use Redis for temporary cache, rate limits, presence and short-lived counters—not as the source of truth for money or permissions.
- Namespace keys by environment and resource, e.g. `prod:user:{userId}:rate`.
- Derive identities from verified server-side authentication, never arbitrary client parameters.
- Include the correct user, tenant or authorization scope in cache keys to prevent cross-user leakage.
- Use atomic Redis operations for rate limits and TTLs for expiring data.
- Do not cache private responses under shared keys; authorize before returning cached data.
- Do not treat Redis cache contents as proof of authorization.

## 9. Project Structure

```text
zevo-live/
├── apps/
│   ├── mobile/                 # Expo + React Native
│   │   └── src/
│   │       ├── screens/
│   │       ├── components/
│   │       ├── features/
│   │       ├── navigation/
│   │       ├── services/
│   │       ├── hooks/
│   │       └── state/
│   └── admin/                  # Next.js dashboard
├── packages/
│   ├── shared-types/
│   ├── validation/
│   └── config/
├── supabase/
│   ├── migrations/
│   ├── functions/
│   ├── tests/
│   └── seed.sql
├── docs/
│   ├── architecture.md
│   ├── security.md
│   └── api.md
├── .env.example
└── README.md
```

## 10. Development Phases

1. **Foundation:** Expo setup, navigation, authentication, schema, migrations and RLS tests.
2. **Audio:** LiveKit integration, room lifecycle, speaker permissions, moderation and reconnection handling.
3. **Economy:** Coin purchases, gift sending, atomic transactions, earnings ledger and payment webhook verification.
4. **Social:** Profiles, follows, messaging, notifications and reporting.
5. **Admin:** Protected dashboard, roles, audit logs and payout management.
6. **Testing and launch:** Security tests, load tests, billing sandbox tests, crash monitoring and production deployment.

## 11. Antigravity Build Instructions

Build ZEVO LIVE as a production-oriented international audio-live application inspired by BIGO Live. Audio streaming only; do not implement video streaming.

### Required implementation

1. Use React Native, Expo and TypeScript for mobile; Supabase for Auth, PostgreSQL, RLS, Storage and Edge Functions; LiveKit Cloud for audio; Upstash Redis for temporary caching and rate limiting; Next.js and TypeScript for a separate admin dashboard.
2. Follow the project structure and database entities in this document.
3. Implement secure authentication, audio rooms, speaker management, virtual gifts, wallets, coin packages, host earnings, reporting and moderation.
4. Enforce authorization server-side and through tested RLS policies.
5. Use short-lived LiveKit tokens, verified payment webhooks, idempotent transactions and immutable financial ledgers.
6. Never expose service-role keys, provider secrets or privileged admin operations to clients.
7. Use safe connection pooling, indexed queries, bounded pagination, Redis key isolation and appropriate rate limits.
8. Include migrations, environment variable validation, automated tests, error handling, logging and deployment documentation.
9. Build the admin dashboard separately with server-enforced role-based access.
10. Use sandbox billing during development. Do not enable real payouts until compliance, verification and fraud controls are implemented.

### Workflow

1. Inspect the existing workspace before changing files.
2. Create the architecture document, database schema, migrations and RLS tests first.
3. Implement in small, testable phases.
4. After each phase, run available tests and fix errors.
5. Document completed work, setup instructions and remaining risks.
6. Never claim a security feature is complete without tests.

**Start with authentication and database security before implementing audio rooms or the wallet.**

## 12. Launch Checklist

- [ ] RLS policies tested against multiple users and roles.
- [ ] Secrets stored only in server-side environment variables.
- [ ] LiveKit room access and token expiration tested.
- [ ] Payment webhook signatures and duplicate-event handling tested.
- [ ] Wallet and earnings transactions verified for concurrency and rollback.
- [ ] Redis rate limits and cache isolation tested.
- [ ] Abuse reporting, moderation, blocking and appeals operational.
- [ ] Privacy policy, terms, age requirements and regional compliance reviewed.
- [ ] Load tests, backups, monitoring and incident procedures ready.
- [ ] Store billing and production credentials configured securely.
