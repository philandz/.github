# 🏦 Philandz v2

Philandz v2 is the next-generation architecture of [Philand](https://github.com/fissama/philand) — a modern budget tracking and financial management platform. v2 migrates the v1 monolith to a **microservices** architecture using the Strangler Fig Pattern, progressively extracting services while maintaining full backward compatibility.

---

## ✨ What is Philand?

Philand is a collaborative financial management platform that lets individuals and teams manage budgets, track transactions, split expenses, and analyze spending — with multi-role access and internationalization (English & Vietnamese).

**Core capabilities:**

- 🔐 **Secure Authentication** — JWT + bcrypt, role-based access, organization IAM
- 💰 **Multi-Budget Management** — Personal, shared, business and project budgets
- 👥 **Team Collaboration** — Invite members with granular role permissions
- 📊 **Analytics & Reports** — Visual charts and monthly summaries
- 🌍 **Internationalization** — English and Vietnamese support
- 📱 **Mobile Optimized** — PWA with responsive design

---

## 🏗️ v2 Architecture

v2 decomposes the v1 monolith into independently deployable services orchestrated behind a central API gateway.

```
Client
  │
  ▼
┌─────────────────────────────┐
│         gateway             │  ← Public HTTP entrypoint
│  (Rust · Axum · tower-http) │    Routes to identity or legacy upstream
└────────────┬────────────────┘
             │
     ┌───────┴────────┐
     │                │
     ▼                ▼
┌──────────┐    ┌───────────────────┐
│ identity │    │  legacy upstream  │
│ (gRPC /  │    │  (Philand v1)     │
│  HTTP)   │    │                   │
└──────────┘    └───────────────────┘
     │
     ▼
  MySQL + Consul
```

### Repositories

| Repository | Language | Description |
|---|---|---|
| [`gateway`](https://github.com/philandz/gateway) | Rust | Public HTTP entrypoint — routes `/api/identity/*` to identity service; all other `/api/*` to legacy upstream |
| [`identity`](https://github.com/philandz/identity) | Rust | Identity service — authentication, JWT, organization IAM, gRPC + REST |
| `libs` | Rust | Shared libraries — config contracts, storage helpers |
| `protobuf` | Protobuf | Shared gRPC service definitions |
| `infra` | Kubernetes / Helm | Infrastructure-as-code — deployment manifests, CI/CD configs |

---

## 🔧 Services

### gateway

The API gateway is the single public entrypoint for all client traffic.

- **Routing**: `/api/identity/*` → identity service (HTTP proxy or gRPC transcoding); all other `/api/*` → legacy monolith
- **Request ID propagation**: Every request receives a `x-request-id` UUID header
- **Swagger aggregation**: Unified API docs combining gateway and all downstream specs
- **Health**: `GET /health`

### identity

The identity service handles all authentication and authorization concerns.

- **Authentication**: `POST /login`, `POST /logout`, `POST /refresh`
- **Registration**: `POST /register`
- **Profile**: `GET /profile`, `POST /update`
- **Password**: `POST /change-password`, `POST /forgot-password`, `POST /reset-password`
- **Organizations**: member listing, invitations, role management, member removal
- **Transport**: gRPC (`127.0.0.1:50051`) + REST (`127.0.0.1:3001`)
- **OpenAPI**: `GET /api-docs/openapi.json` · Swagger UI: `GET /docs`
- **Token security**: SHA-256 hashed revocation tokens stored in DB

---

## 🚀 Tech Stack

| Layer | Technology |
|---|---|
| **Backend** | Rust · Axum · SQLx · tonic (gRPC) |
| **Frontend** | Next.js 14 · React 18 · TypeScript · Tailwind CSS · Radix UI |
| **Database** | MySQL 8.0 |
| **Service Discovery** | Consul |
| **Container Registry** | Harbor (`harbor.philand.io.vn`) |
| **Orchestration** | Kubernetes |
| **API Contracts** | Protocol Buffers (protobuf) |
| **CI/CD** | GitHub Actions — semantic versioning, container build & push, infra manifest update |

---

## 🔐 Security & Roles

| Role | Permissions |
|---|---|
| **Owner** | Full control, manage members, delete budget |
| **Manager** | Manage categories and settings, view all data |
| **Contributor** | Add/edit transactions, view budget data |
| **Viewer** | Read-only access |

JWT claims carry `{ sub, email, org_id, exp }`. Logout revokes tokens via SHA-256 hashed storage in `revoked_tokens`.

---

## 📦 CI/CD

Each service uses a **three-stage GitHub Actions pipeline**:

1. **ci-sandbox** — Build, test (`cargo test`), and lint (`cargo clippy`) on every push
2. **pre-release-main** — Pre-release validation on main branch
3. **release-main** — Semantic version tagging → Harbor image build & push → infra deploy manifest update

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/): `feat`, `fix`, `docs`, `refactor`, `chore`, etc. Breaking changes bump the major version.

---

## 🗺️ Roadmap

| Phase | Target | Theme |
|---|---|---|
| Phase 1 (Q4 2025) | Enhanced budgets | Budget types, entry tags, bulk operations |
| Phase 2 (Q1 2026) | Collaborative finance | Budget sharing, expense splitting, balance tracking |
| Phase 3 (Q2 2026) | Financial profiles | Savings goals, debt management, investment tracking |
| Phase 4 (Q3 2026) | Advanced operations | Inter-budget transfers, rollback system |
| Phase 5 (Q4 2026) | Audit & compliance | Comprehensive audit log, data export |
| Future (2027+) | Mobile & AI | Native apps, bank sync, AI-powered insights |

---

## 🌐 Links

- **Website**: [www.phila.cloud](https://www.phila.cloud)
- **Docs**: [www.phila.cloud/docs](https://www.phila.cloud/docs)
- **v1 Source**: [fissama/philand](https://github.com/fissama/philand)

---

<div align="center">

**Built with ❤️ by the Philand Team**

</div>