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
