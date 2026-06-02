# CivicLedger — Backend API (Stellar / Soroban integration)

Government-grade procurement needs controlled APIs for agencies, escrow orchestration, and oversight exports—this Fastify layer is built for that operational reality.

---

## 🎯 What is this service?

CivicLedger’s contracts (`tender-registry`, `milestone-escrow`, `vendor-reputation`) anchor **facts** about tenders and disbursements. Agencies still run ERPs, PDF bid bonds, and citizen portals—this backend is where **authenticated connectors** land: posting tenders from procurement suites, accepting milestone evidence files, generating regulator-ready exports, and enforcing RBAC that complements on-chain roles.

---

## ❓ Problems the **protocol** solves (whole repo)

These come from the [root README](../../README.md) — shared context for why Stellar/Soroban exists here:

- Citizens and oversight bodies lack **timely, trustworthy** visibility into tenders and outcomes.
- Donor and taxpayer funds often move through **opaque** milestones with weak proof of delivery.
- Vendor selection rarely produces **portable reputation**, encouraging repeated failures.

---

## 🛠️ Problems **this API** solves specifically

The smart contracts hold **truth on-chain**; they cannot safely hold ERP passwords, IoT vendor keys, bulk files, or cron jobs. That is this service’s job:

- **Enterprise connectors**: Ministries won’t sign txs only from a marketing site—need server-trusted integrations.
- **Evidence payloads**: Large milestone proofs (documents, hashes) should upload to object storage via API—not chain.
- **Dual control**: Escrow release sometimes requires multi-approver workflows off-chain before final Soroban submit.
- **Observability**: Watchdog NGOs need aggregate APIs without raw PII leakage.

---

## ✅ Protocol goals this backend helps achieve

- Publish tender lifecycle events so awards are **tamper-evident**.
- Hold funds in **milestone escrow** until objective evidence is accepted.
- Surface **vendor performance** and disputes for future procurement decisions.
- Support **agency pilots** without forcing one vendor’s closed database as source of truth.

---

## ✨ Capabilities this backend enables (production roadmap)

- **Agency connectors**: POST tenders/bids from authorized systems with API keys + audit logs.
- **Evidence intake**: Virus-scanned uploads; store URIs + hashes matching `milestone-escrow` expectations.
- **Release workflows**: Approvals queues → single controlled submission to Soroban.
- **Transparency feeds**: Read-optimized endpoints backed by indexers for civic dashboards.

---

## 🔗 Soroban crates → API responsibilities

| Crate | What the HTTP layer typically does |
| ----- | ---------------------------------- |
| `tender-registry` | Publish structured tender lifecycle events from ERP; reconcile IDs with on-chain records. |
| `milestone-escrow` | Coordinate staged funding and evidence hashes; prepare payout transactions after approvals. |
| `vendor-reputation` | Aggregate performance signals from operational DB + chain for scoring APIs. |

---

## 🏗️ Architecture & stack

| Layer | Choice |
| ----- | ------ |
| HTTP framework | **Fastify** 5 — low overhead, schema-friendly |
| Language | **TypeScript** (strict, ESM, `verbatimModuleSyntax`) |
| Config | **Zod** parsing in `src/config/env.ts` |
| Blockchain | **Stellar** Horizon + **Soroban** RPC (server-side keys only) |
| Consumers | [`apps/web`](../web/README.md), partner systems, cron workers |

---

## 📁 Package layout

```
apps/backend/
├── .env.example
├── package.json
├── tsconfig.json
├── README.md
└── src/
    ├── index.ts              # Fastify bootstrap, CORS, route registration
    ├── config/env.ts         # Typed environment
    └── routes/
        ├── health.ts         # GET /health
        └── v1/index.ts       # Versioned API surface (expand here)
```

---

## 🚀 Quick start

### Prerequisites

- **Node.js** 20.x or **22.x** (LTS)
- npm (or pnpm/yarn per org standard)

### Install & run

```bash
cd apps/backend
npm install
cp .env.example .env
# Edit .env — see tables below
npm run dev
```

Default: **http://localhost:8080** · Health: **GET** `/health` · Meta: **GET** `/api/v1/meta`

### Run with the Next.js frontend

```bash
# Terminal A — API
cd apps/backend && npm run dev

# Terminal B — Web
cd apps/web && npm install && npm run dev
```

Set `CORS_ORIGIN` in `.env` to match the web origin (e.g. `http://localhost:3000`).

---

## 📜 Scripts

| Command | Purpose |
| ------- | ------- |
| `npm install` | Install dependencies |
| `npm run dev` | `tsx watch` — reload on change |
| `npm run build` | Compile to `dist/` |
| `npm start` | Run compiled server |
| `npm run lint` | `tsc --noEmit` typecheck |

---

## 🔐 Environment variables

### Baseline (implemented)

| Variable | Default | Purpose |
| -------- | ------- | ------- |
| `NODE_ENV` | `development` | Environment name |
| `PORT` | `8080` | Listen port |
| `API_PREFIX` | `/api/v1` | Prefix for versioned routes |
| `CORS_ORIGIN` | `http://localhost:3000` | Browser origin allowed by CORS |

### Production / integration (plan — **do not commit secrets**)

| Variable | Example | Purpose |
| -------- | ------- | ------- |
| `AGENCY_API_KEYS` | (secret map) | Per-agency credentials—prefer Vault. |
| `OBJECT_STORAGE_*` | S3-compatible | Evidence blobs. |
| `SOROBAN_RPC_URL` | `https://…` | Chain submission. |

---

## 🔌 HTTP surface

### Implemented (scaffold)

| Method | Path | Description |
| ------ | ---- | ----------- |
| GET | `/health` | Liveness for load balancers & CI |
| GET | `/api/v1/meta` | Service name / version |

### Planned themes (domain routes — implement under `src/routes/v1/`)

- `POST /api/v1/agencies/:id/tenders` — authenticated tender publication pipeline.
- `POST /api/v1/milestones/:id/evidence` — upload + hash registration.
- `GET /api/v1/transparency/summary` — public aggregates for dashboards.

---

## 🧪 Testing & quality

```bash
npm run lint
```

CI should mirror this (see [`.github/workflows/ci.yml`](../../.github/workflows/ci.yml)).

Add **contract integration tests** in the Rust workspace and **API integration tests** (e.g. `vitest` + `supertest`) as routes grow.

---

## 🚢 Deployment notes

- Run behind TLS termination (load balancer or reverse proxy).
- Store signing keys in **KMS/HSM**, never in repo.
- Restrict Soroban RPC by IP allowlist or private gateway when possible.
- Emit structured logs (JSON) with **request IDs** for regulator audits (especially MediProof / CivicLedger / ReliefFlow).

---

## 🤝 Contributing

See [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md). Contract changes must stay aligned with this API’s eventual routes and [`../../docs/SITE_MAP.md`](../../docs/SITE_MAP.md).

---

## 📄 License

Match the repository license (Apache-2.0 suggested for OSS grants — confirm per org).

---

## 📞 Support & related docs

| Doc | Link |
| --- | ---- |
| Monorepo overview | [`../../README.md`](../../README.md) |
| Frontend | [`../web/README.md`](../web/README.md) |
| Architecture notes | [`../../docs/layout-plan.md`](../../docs/layout-plan.md) |
| Milestones → issues | [`../../docs/milestones-issues.md`](../../docs/milestones-issues.md) |

---

**Package:** `civicledger-api` · **Slug:** `civicledger`
