# crm-base

> A solid, self-hostable CRM template built on **[Twenty](https://github.com/twentyhq/twenty)** — fork it per use case, customize, ship.

This repository is a **reusable base** for building tailored CRMs. It starts from the open-source [Twenty](https://twenty.com) codebase and is meant to be declined into purpose-specific products (one repo per use case) while keeping a clean, upstream-trackable foundation.

It is published as a **GitHub Template repository**: hit **`Use this template`** to spin up an independent repo for a new use case, then customize it in isolation without touching this base.

---

## Why this exists

Twenty is a great, modern CRM you can own and version like the rest of your stack. Rather than starting each project from scratch, this repo keeps **one solid base** that:

- carries shared conventions and any base-level customizations,
- can be **forked into multiple use-case CRMs** via _Use this template_,
- stays **synced with upstream Twenty** so it benefits from ongoing fixes and features.

> ℹ️ The original Twenty README (cloud signup, app scaffolding, full feature tour) is preserved at **[`README.upstream.md`](./README.upstream.md)**.

---

## Stack

TypeScript monorepo managed with **[Nx](https://nx.dev)** + **Yarn 4**.

- **Backend** — [NestJS](https://nestjs.com), [PostgreSQL](https://www.postgresql.org), [Redis](https://redis.io), [BullMQ](https://docs.bullmq.io) (background worker), GraphQL (code-first, schema generated dynamically from metadata).
- **Frontend** — [React](https://react.dev) with [Jotai](https://jotai.org) (state), [Linaria](https://linaria.dev) (zero-runtime CSS-in-JS), [Lingui](https://lingui.dev) (i18n), Vite.

### How it fits together

```
 twenty-front (React, :3001)  ──GraphQL──▶  twenty-server (NestJS, :3000)
                                                  │
                         ┌────────────────────────┼────────────────────────┐
                         ▼                         ▼                         ▼
                    PostgreSQL                  Redis  ◀──BullMQ──▶  worker (async jobs)
              (multi-tenant: one                (cache +
               schema per workspace)            job queues)
```

The GraphQL API is generated **dynamically** from object/field metadata, so users can define custom objects and fields that become real Postgres tables — that's the core of Twenty's customizability.

---

## Repository layout

```
packages/
├── twenty-front/          # React frontend application
├── twenty-server/         # NestJS backend API + worker
├── twenty-ui/             # Shared UI component library
├── twenty-shared/         # Common types and utilities (build first)
├── twenty-emails/         # Transactional email templates (React Email)
├── twenty-website/        # Marketing website (Next.js)
├── twenty-docs/           # Documentation site
├── twenty-zapier/         # Zapier integration
└── twenty-e2e-testing/    # Playwright E2E tests
```

---

## Getting started

### Prerequisites

- **Node** `24.16.0` (see [`.nvmrc`](./.nvmrc)) — `nvm install && nvm use`
- **Yarn 4** (via Corepack: `corepack enable`)
- **PostgreSQL 16** and **Redis** — running locally, or via the bundled dev Docker Compose

> ⚠️ **Heads-up on disk:** a full install of this monorepo needs **~12–15 GB** free (`node_modules` + Yarn cache + builds). For development, a **cloud workspace** (e.g. GitHub Codespaces) is the path of least resistance — the heavy install lives in the cloud, not on your machine. See [Cloud development](#cloud-development-recommended) below.

### Local setup

```bash
# 1. Provision Postgres + Redis, create databases, copy .env files (idempotent)
bash packages/twenty-utils/setup-dev-env.sh
#    flags: --docker (force Docker)  |  --down (stop)  |  --reset (wipe + restart)

# 2. Install dependencies
yarn install

# 3. Initialize schema, run migrations, seed demo data
npx nx database:reset twenty-server

# 4. Start frontend + backend + worker
yarn start
```

Then open:

| Service | URL |
|---|---|
| Frontend | http://localhost:3001 |
| Backend / GraphQL | http://localhost:3000 |

Sign in via **"Continue with Email"** — credentials are prefilled in development (`SIGN_IN_PREFILLED=true`).

### Cloud development (recommended)

This repo is template-ready and works out of the box with cloud dev environments:

1. From this repo (or a use-case repo created from it), launch a **GitHub Codespace** (4 cores minimum; 8 for faster builds).
2. Run the same four commands from [Local setup](#local-setup) inside the Codespace.
3. Codespaces forwards the ports automatically — open the forwarded `:3001` URL in your browser.

Your machine stays clean; the install and build live in the cloud.

---

## Deriving a new use case

1. Click **`Use this template`** on this repo → create a new repository (e.g. `crm-real-estate`).
2. Clone it (or open it directly in a Codespace) and customize for that use case.
3. The new repo is fully independent — changes there never affect this base.

Keep base-level changes here in `crm-base`; keep use-case-specific changes in their own repos.

---

## Staying in sync with upstream Twenty

This repo keeps a reference to the original project so you can pull improvements:

```bash
git remote -v
# origin    -> your fork (this repo)
# upstream  -> https://github.com/twentyhq/twenty.git

git fetch upstream
git merge upstream/main        # resolve conflicts, then commit
```

> Keep customizations **additive and isolated** where possible — it makes upstream merges far less painful.

---

## Deployment

This base ships the standard Twenty self-hosting setup (Docker Compose) under `packages/twenty-docker/`. See the upstream [self-hosting docs](https://docs.twenty.com/developers/self-host/capabilities/docker-compose) for production deployment.

---

## Credits & license

Built on **[Twenty](https://github.com/twentyhq/twenty)** by the Twenty team and contributors. All credit for the underlying CRM goes to them. This repository is a personal template/fork.

License terms are inherited from upstream — see [`LICENSE`](./LICENSE). Review them before using any part of this code in production or commercially.
