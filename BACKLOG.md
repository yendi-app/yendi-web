# Backlog

Work that's out of scope for the current build but worth revisiting. Each item
notes why it was deferred and where to pick it up.

## Auth providers

- **Apple Sign-In** — deferred until the $99 Apple Developer account exists (needs a
  Services ID + signing key). Required by App Store rules once a native iOS app ships
  alongside other social logins, so wire it before native.
- **Facebook Login** — relevant for LATAM reach, more setup friction. Add only if
  Google + email leaves a meaningful gap. (Instagram SSO is not available — Meta
  deprecated the Basic Display API in Dec 2024.)

## Shareable event links (the viral loop)

- Public event page (`app/e/[slug]/page.tsx`) as a Server Component with
  `generateMetadata()` emitting per-event Open Graph tags, so links shared on
  WhatsApp/iMessage get rich preview cards. Core to growth — build with the events feature.

## PWA

- `manifest.webmanifest`, icons, and a service worker so the app is installable
  ("Add to Home Screen") and feels native without the store. Add once the core flow works.

## Tooling

- **ESLint boundary enforcement** for the `shared → features → app` layering rule
  (e.g. `eslint-plugin-boundaries`), so cross-feature imports fail CI instead of
  relying on discipline.
- **Generated DB types** — wire `types/database.ts` (from `supabase gen types`) as the
  generic on the Supabase clients for end-to-end typed queries.
- **TanStack Query** — only if/when a screen genuinely needs client-side fetching/caching.
  Server Components cover the current surface.
