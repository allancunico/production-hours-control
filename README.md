# Production Hours Control

Internal production management system for the electrical panel assembly team

Built as a Progressive Web App with push notification support, offline capability, and native installation on mobile devices.

---

## Project Background

This application was initially built by one production leads using the [Lovable](https://lovable.dev) platform and later taken over by a FullStack developer for refactoring, feature development, and production maintenance.

The business domain is specific: industrial electrical panel assembly with internally defined shifts, an operational calendar and production stages.

---

## Features

### For team members
- Log hours per project and production stage with start and end time
- Personal dashboard with real-time missing hours for the day and month
- Full log history with period filters
- View your own consolidated performance board
- Personal task management with status and due date
- Absence registration with categories (medical leave, vacation, off-site work, etc.)
- Receive push notifications for feedback and updates

### For administrators
- Operational dashboard with real-time team KPIs
- Consolidated Operations Board — individual and team capacity calculated from the operational calendar
- Productivity, time-logging and idleness rankings per team member
- Unproductivity cause ranking with root cause and impact analysis
- Financial indicators: executed cost vs. planned cost per project
- Full project management with stage-level planning (Internal Layout, External Layout, Component Assembly, Electrical Assembly, Busbars, Quality)
- Task delegation between team members
- Absence board with team approval flow
- History with 13 views: time logs, completed projects, performance, idleness, costs, operational, team members, tasks, feedback, absences, monthly archive and audit trail
- Automatic monthly closing with operational and financial indicator snapshots
- PDF and XLSX report exports

---

## Stack

| Layer | Technology |
|---|---|
| Framework | [TanStack Start](https://tanstack.com/start) (React 19, SSR) |
| Routing | TanStack Router (file-based) |
| Data fetching | TanStack Query v5 |
| UI | Tailwind CSS v4 + shadcn/ui + Radix UI |
| Charts | Recharts |
| Backend | [Supabase](https://supabase.com) (PostgreSQL + Auth + Realtime + RLS) |
| Deploy | Cloudflare Workers (edge SSR) |
| PWA | Web App Manifest + Service Worker + Web Push API (VAPID) |
| Reports | jsPDF + jsPDF AutoTable + xlsx |
| Testing | Vitest |
| Language | TypeScript 5.8 |

---

## Architecture

```
src/
├── routes/            # Thin routers only — no business logic
│   ├── _app.index.tsx          # 13 lines — delegates to AdminDashboard or ColabDashboard
│   ├── _app.historico.tsx      # 21 lines — delegates to AdminHistorico or ColabHistorico
│   ├── _app.projetos.tsx       # 6 lines  — renders ProjetosPage
│   ├── _app.desempenho.tsx     # 77 lines — orchestrates performance tabs
│   └── _app.ausencias.tsx      # orchestrates absence and idleness views
│
├── components/
│   ├── dashboard/     # KpiCard, SectionTitle, FaltantesCard, RankingCard,
│   │                  # CauseRankingCard, ConsolidatedMatrix,
│   │                  # AdminDashboard, ColabDashboard
│   ├── historico/     # 14 files — one file per tab
│   ├── projetos/      # PlanejamentoDialog, MatrizProcessosColabs,
│   │                  # NewProjectForm, EditPlanejamentoDialog,
│   │                  # DuplicateProjectDialog, Resumo, DelegateTaskDialog
│   ├── desempenho/    # GeralView, IndividualView, FeedbackOperacionalCard,
│   │                  # ProjetosEnvolvidosSection, Kpi, MiniStat
│   ├── ausencias/     # IndividuaisView, OciosidadeSection,
│   │                  # OciosidadeIndividualSection, KpiCard, MiniIndicator
│   └── feedback/      # FeedbackMiniCards, DescriptiveFeedback,
│                      # ConsolidatedMatrix, TeamAnnualMatrix, Metrics
│
├── hooks/             # Reusable hooks
│   ├── use-auth.tsx               # Session + user role
│   ├── use-calendar-capacity.ts   # Operational calendar capacity
│   ├── use-live-clock.ts          # Real-time clock
│   ├── use-realtime-invalidate.ts # Scoped query key invalidation
│   ├── use-jornada.ts             # Configurable daily shift
│   └── use-quadro-consolidado.tsx # Single source of truth for KPIs
│
├── lib/
│   ├── operational-engine.ts   # Productive time and overtime calculation engine
│   ├── feedback-engine.ts      # Pure performance analysis functions (fully testable)
│   └── constants.ts            # Operational parameters, formatters, helpers
│
└── integrations/supabase/
    ├── client.ts               # Public client (browser)
    ├── client.server.ts        # Admin client (server functions, service role)
    └── types.ts                # Auto-generated types from schema
```

---

## Operational Engine

The productive time calculation is centralized in `src/lib/operational-engine.ts`. It is the **single source of truth** for time logs, dashboard, productivity, costs, reports and rankings.

**Official shift:** 07:25 – 17:03
**Fixed automatic deductions (applied within the shift):**

| Period | Start | End | Duration |
|---|---|---|---|
| Morning break | 09:00 | 09:10 | 10 min |
| Time-logging (morning) | 11:50 | 11:55 | 5 min |
| Lunch | 12:00 | 12:50 | 50 min |
| Afternoon break | 15:00 | 15:10 | 10 min |
| Time-logging (end of day) | 16:55 | 17:00 | 5 min |

**Official net productive shift:** `8h18m (498 min)`

Overtime is calculated only for entries with explicit `start_time` and `end_time` outside the official shift window. Weekends are automatically treated as full overtime.

---

## Database

All tables have **Row Level Security (RLS) enabled** with granular policies.

| Table | Description | RLS |
|---|---|---|
| `profiles` | User profiles | select/update own · admin all |
| `user_roles` | Role (`admin` or `colab`) | select own · admin all |
| `projects` | Projects with stage-level planning | select auth · write admin |
| `time_entries` | Hour logs | select/update own or admin |
| `absences` | Absences and leave | select/update own or admin |
| `tarefas` | Tasks per project and team member | select own or admin |
| `system_settings` | Operational settings | read all · write admin |
| `notifications` | Push notifications | insert server · select own |
| `push_subscriptions` | Web Push subscriptions | own only |
| `monthly_snapshots` | Monthly performance snapshot | own or admin |
| `dashboard_monthly_archive` | Monthly operational closing | admin |
| `general_launch_audit` | Edit audit trail | admin |
| `project_deletions` | Deleted project log | admin |

---

## Running Locally

### Prerequisites

- [Bun](https://bun.sh) ≥ 1.1
- A [Supabase](https://supabase.com) account (free tier works)

### 1. Clone and install

```bash
git clone https://github.com/your-username/production-hours-control.git
cd production-hours-control
bun install
```

### 2. Set up environment variables

```bash
cp .env.example .env
```

Fill in `.env` with your Supabase credentials:

```env
SUPABASE_URL=https://YOUR_PROJECT.supabase.co
SUPABASE_PUBLISHABLE_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=eyJ...

VITE_VAPID_PUBLIC_KEY=...
VAPID_PRIVATE_KEY=...
WEBHOOK_SECRET=...
```

> Generate VAPID keys with: `npx web-push generate-vapid-keys`

### 3. Apply migrations

```bash
bunx supabase db push
```

Or apply the files in `supabase/migrations/` manually via the Supabase dashboard SQL editor.

### 4. Start the development server

```bash
bun run dev
```

Open `http://localhost:3000`.

---

## Deploy

The project is configured for deployment on **Cloudflare Workers** via Wrangler.

```bash
bun run build
bunx wrangler deploy
```

Set the following environment variables in the Cloudflare Workers dashboard:

```
SUPABASE_URL
SUPABASE_PUBLISHABLE_KEY
SUPABASE_SERVICE_ROLE_KEY
VITE_VAPID_PUBLIC_KEY
VAPID_PRIVATE_KEY
WEBHOOK_SECRET
```

---

## Environment Variables

| Variable | Context | Description |
|---|---|---|
| `VITE_SUPABASE_URL` | Browser | Supabase project public URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Browser | Supabase anon key (public) |
| `SUPABASE_URL` | Server | Project URL for server functions |
| `SUPABASE_PUBLISHABLE_KEY` | Server | Anon key for server functions |
| `SUPABASE_SERVICE_ROLE_KEY` | Server | Service role key — **never expose to the browser** |
| `VITE_VAPID_PUBLIC_KEY` | Browser | VAPID public key for Web Push |
| `VAPID_PRIVATE_KEY` | Server | VAPID private key — **never expose** |
| `WEBHOOK_SECRET` | Server | Webhook validation token |

---

## Scripts

```bash
bun run dev       # Development server with HMR
bun run build     # Production build
bun run preview   # Preview production build locally
bun run test      # Run unit tests
bun run lint      # ESLint
bun run format    # Prettier
```

---

## Security

- **RLS enabled** on all tables — users can only read and write their own data
- **Server functions** validate the user role before any admin operation
- **Client/server separation** in Supabase: `service_role_key` never reaches the browser
- **Authenticated webhook** via `WEBHOOK_SECRET` in the `x-webhook-secret` header
- **Audit trail** (`general_launch_audit`) records every admin edit on the Consolidated Board

---

## Author

**Allan Pedroso Cunico**
FullStack Developer
[LinkedIn](https://linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username)
