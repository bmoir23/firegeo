# Deployment Guide

This app is a **Next.js 15** project and deploys most smoothly on **Vercel**. Netlify also works with the Next.js adapter.

## Prerequisites

Before deploying, provision:

1. **PostgreSQL** — [Neon](https://neon.tech), [Supabase](https://supabase.com), or [Render Postgres](https://render.com/docs/databases)
2. **Autumn billing key** — [useautumn.com](https://useautumn.com) (`AUTUMN_SECRET_KEY`)
3. **Firecrawl API key** — for brand monitor scraping ([app.firecrawl.dev](https://app.firecrawl.dev/api-keys))
4. **At least one AI provider key** — OpenAI, Anthropic, Google, or Perplexity
5. **Resend API key** — optional, for password reset emails

## Required environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `BETTER_AUTH_SECRET` | Yes | `openssl rand -base64 32` |
| `NEXT_PUBLIC_APP_URL` | Yes | Production URL (e.g. `https://your-app.vercel.app`) |
| `AUTUMN_SECRET_KEY` | Yes | Autumn API secret (app won't start without it) |
| `FIRECRAWL_API_KEY` | For brand monitor | Firecrawl scraping |
| `OPENAI_API_KEY` | Optional | AI chat / analysis |
| `ANTHROPIC_API_KEY` | Optional | AI chat / analysis |
| `GOOGLE_GENERATIVE_AI_API_KEY` | Optional | AI chat / analysis |
| `PERPLEXITY_API_KEY` | Optional | AI chat / analysis |
| `RESEND_API_KEY` | Optional | Transactional email |

## Database setup (production)

After creating your Postgres database, run migrations once from your machine or CI:

```bash
# Apply Better Auth + app schema
psql "$DATABASE_URL" -f better-auth_migrations/initial-schema.sql
psql "$DATABASE_URL" -f migrations/001_create_app_schema.sql
npm run db:push
```

## Deploy to Vercel (recommended)

1. Push this repo to GitHub/GitLab
2. Import the project at [vercel.com/new](https://vercel.com/new)
3. Framework preset: **Next.js** (auto-detected)
4. Add all environment variables from the table above
5. Set `NEXT_PUBLIC_APP_URL` to your Vercel domain
6. Deploy

```bash
# Or via CLI
npm i -g vercel
vercel login
vercel --prod
```

### Vercel notes

- Use **Transaction mode** connection strings for serverless (Neon/Supabase pooler URLs)
- Run database migrations before first deploy
- Autumn webhooks are handled automatically by Autumn when Stripe is connected

## Deploy to Netlify

1. Push repo to GitHub/GitLab
2. Create site at [app.netlify.com](https://app.netlify.com)
3. Build settings:
   - **Build command:** `npm run build`
   - **Publish directory:** `.next` (Netlify Next.js plugin handles this)
4. Install the **@netlify/plugin-nextjs** plugin (auto with new Next sites)
5. Add environment variables in Site settings → Environment variables
6. Set `NEXT_PUBLIC_APP_URL` to your Netlify domain

```bash
npm i -g netlify-cli
netlify login
netlify init
netlify deploy --prod
```

## Local development

```bash
cp .env.example .env.local
# Edit DATABASE_URL, BETTER_AUTH_SECRET, AUTUMN_SECRET_KEY

npm install --legacy-peer-deps
npm run setup   # or apply SQL migrations manually
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## What's included

| Feature | Route | Notes |
|---------|-------|-------|
| Landing page | `/` | Marketing homepage |
| Auth (Better Auth) | `/login`, `/register` | Email/password |
| Dashboard | `/dashboard` | User hub (auth required) |
| AI Chat | `/chat` | Multi-provider chat |
| Brand Monitor | `/brand-monitor` | Firecrawl + AI competitive analysis |
| Pricing / Plans | `/plans` | Autumn billing integration |

## Troubleshooting

- **Build fails on `better-auth/nextjs`** — import must be `better-auth/next-js` (fixed in this branch)
- **Autumn key error** — `AUTUMN_SECRET_KEY` must be set (even a placeholder for local-only UI testing)
- **Auth tables missing** — run `better-auth_migrations/initial-schema.sql` and `migrations/001_create_app_schema.sql`
- **Brand monitor empty** — add `FIRECRAWL_API_KEY` and at least one AI provider key
