# Yendi Web

**Yendi** (from Spanish _"Yendo"_ — going) is a social event-planning app: create a
plan, share one link, and watch RSVPs and logistics sort themselves out. This
repository is the **product web app** — a mobile-first PWA where users sign in, create
plans, and RSVP. It's the primary surface; native (Expo/React Native) comes later.

The marketing site lives in a separate repo,
[`yendi-landing-page`](https://github.com/yendi-app/yendi-landing-page).

## Why web-first

Yendi's growth loop is link-shared: a host invites people, who invite people, over
WhatsApp/iMessage. An app-store install wall sits right on top of that loop and kills
it — especially in LATAM, where WhatsApp is the distribution channel and install
friction is costly. So the core flow is a URL that opens instantly. Native apps come
once the product is validated; the backend and business logic carry over, the UI is
rebuilt.

## Tech Stack

- **[Next.js 16](https://nextjs.org)** (App Router, Server-Components-first) +
  **TypeScript** + **[Tailwind](https://tailwindcss.com)**.
- **[Supabase](https://supabase.com)** — auth (Google, Apple, email magic link) and
  Postgres — via `@supabase/ssr`.
- **[zod](https://zod.dev)** + **[react-hook-form](https://react-hook-form.com)** for
  forms validated on both client and server.
- Deploys on **[Vercel](https://vercel.com)**.

## Architecture

Feature-modular, with strict one-way layering and the database as the security
boundary. The rules that keep it maintainable:

- **One-way layering: `shared → features → app`.** `app/` only composes; features hold
  the real code; shared (`components/ui`, `lib`, `types`) is used anywhere. A feature
  **never imports from another feature** — shared code gets promoted to `shared`.
- **Server-Components-first.** Reads happen in Server Components via the server Supabase
  client. Writes happen in **Server Actions** colocated in `features/<x>/api/`. Client
  Components only where interactivity is needed.
- **Auth uses `getClaims()`**, never `getSession()` on the server — a session read from
  storage can be spoofed; `getClaims()` verifies the JWT.
- **Postgres RLS is the real boundary.** Every table has row-level policies, so security
  survives frontend bugs.
- **Business rules are plain TypeScript** (no React/Next/DOM imports), so they port
  cleanly to the future native app.

### Next.js 16 notes

- The old `middleware.ts` is renamed to **`proxy.ts`** (function `proxy()`), defaulting
  to the Node.js runtime. Ours runs the Supabase session refresh on every request.
- `cookies()` is **async** — the server client wrapper is `async`.

## Project Structure

```text
/
├── app/                        # routing layer — route groups, pages, layouts (thin)
│   ├── (auth)/login/           # login screen
│   └── (app)/                  # authenticated shell
├── features/                   # feature modules — the app's real code
│   └── <feature>/              # components/  api/  hooks/  types  (auth, onboarding, profile, …)
├── components/ui/              # shared, presentational UI primitives
├── lib/supabase/               # Supabase clients: client.ts, server.ts, proxy.ts
├── types/                      # shared types, incl. generated database.ts
├── supabase/migrations/        # SQL migrations — schema as code
├── proxy.ts                    # session refresh on every request (Next 16 "proxy")
└── package.json
```

## Getting Started

All commands run from the project root:

| Command           | Action                                          |
| :---------------- | :---------------------------------------------- |
| `npm install`     | Install dependencies                            |
| `npm run dev`     | Start the dev server at `localhost:3000`        |
| `npm run build`   | Production build                                |
| `npm run start`   | Serve the production build                      |
| `npm run lint`    | Run ESLint                                      |
| `npx tsc --noEmit`| Typecheck                                       |

### Environment

Copy `.env.local.example` to `.env.local` and fill in your Supabase values:

```bash
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
```

Get both from the Supabase dashboard (Settings → API). Then apply the schema by running
`supabase/migrations/0001_init_profiles.sql` in the SQL Editor.

```text
Add Google sign-in and auth callback route
Create profiles table with row-level security
Fix session refresh dropping cookies on redirect
```

## Backlog

Deferred work — additional auth providers, the shareable-link Open Graph cards, PWA
install support, and tooling follow-ups — is tracked in [BACKLOG.md](./BACKLOG.md).
