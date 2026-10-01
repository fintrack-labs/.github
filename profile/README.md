<!-- Taruh file ini di: fintrack-labs/.github -> profile/README.md -->

# FinTrack Labs

> A personal-finance platform built as a hands-on lab for **distributed backend design, secure token-based auth, and AI-assisted receipt scanning**.

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Vue 3](https://img.shields.io/badge/Vue_3-42B883?logo=vuedotjs&logoColor=white)
![NestJS](https://img.shields.io/badge/NestJS-E0234E?logo=nestjs&logoColor=white)
![Fastify](https://img.shields.io/badge/Fastify-000000?logo=fastify&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/status-learning_project-blue)

**FinTrack** lets a user track accounts, categories, and transactions, and scan a receipt photo to pre-fill a transaction using a vision AI model. It is a **personal learning project** (no clients, not production), but it is built the way I would build a real system: separate services, separate databases, and explicit trust boundaries.

<!-- TODO: tambahkan screenshot / GIF demo di sini -->
<!-- ![FinTrack demo](./assets/demo.gif) -->

**Live demo:** <!-- TODO: https://... --> &nbsp;|&nbsp; **Demo account:** <!-- TODO: email / password -->

---

## Why this project is interesting

- **Auth as its own service.** Issues RS256-signed JWTs and publishes a **JWKS** endpoint, so other services verify tokens locally without calling auth on every request.
- **Safe money handling.** Transfers update two balances inside one database transaction with **pessimistic row locking**.
- **AI treated as untrusted input.** The OCR service validates the model's JSON and replaces any AI-chosen IDs with values from trusted backend data. The AI provider never sees the user's token.
- **Clear data ownership.** Auth and finance services use separate databases; OCR stores nothing.
- **Honest engineering notes.** Known risks and trade-offs are documented rather than hidden (see [Known limitations](#known-limitations--roadmap)).

## Architecture

```mermaid
flowchart LR
    User[User / Browser]
    FE[fe-web<br/>Vue SPA]
    AUTH[be-auth-ts<br/>Auth API]
    CORE[be-node-ts<br/>Finance API]
    OCR[be-ai-ocr-service<br/>OCR adapter]
    AUTHDB[(Auth PostgreSQL)]
    COREDB[(Finance PostgreSQL)]
    AI[OpenAI-compatible<br/>vision provider]

    User --> FE
    FE -->|login / refresh| AUTH
    AUTH --> AUTHDB
    FE -->|accounts, categories, transactions| CORE
    CORE -->|fetch public keys (JWKS)| AUTH
    CORE --> COREDB
    FE -->|receipt image + bearer token| OCR
    OCR -->|forward token, read master data| CORE
    OCR -->|image + minimized candidates| AI
```

## Repositories

| Repository | What it does | Stack |
| --- | --- | --- |
| [`fe-web`](https://github.com/fintrack-labs/fe-web) | Web app: auth, accounts, transactions, receipt scan | Vue 3, Vite, Pinia, Vue Router, Axios |
| [`be-auth-ts`](https://github.com/fintrack-labs/be-auth-ts) | Registration, login, refresh/revoke, JWKS | Fastify, Prisma, PostgreSQL |
| [`be-node-ts`](https://github.com/fintrack-labs/be-node-ts) | Finance domain API (accounts, categories, transactions) | NestJS on Fastify, TypeORM, PostgreSQL |
| [`be-ai-ocr-service`](https://github.com/fintrack-labs/be-ai-ocr-service) | Stateless receipt analysis via a vision model | Fastify, OpenAI-compatible API |
| [`be-api-client-test`](https://github.com/fintrack-labs/be-api-client-test) | Bruno API collection for local testing | Bruno |

## Key design decisions

1. **Local JWT verification via JWKS** instead of per-request calls to auth, which removes a runtime dependency and a latency hop.
2. **Refresh tokens are stored only as SHA-256 hashes** and can be revoked on logout.
3. **Authorization comes from the verified token subject**, never from client-supplied user IDs; all finance reads and writes are scoped by `userId`.
4. **OCR never creates transactions.** It returns a draft; the user reviews and submits it separately.
5. **Services own their data.** No shared tables between auth and finance.

## Known limitations & roadmap

I deliberately documented what is not production-ready yet:

- [ ] Enforce JWT issuer, audience, and algorithm explicitly in the finance API
- [ ] Durable signing-key storage and key rotation (JWKS publishing old keys during rollover)
- [ ] Move tokens from `localStorage` to HttpOnly secure cookies (or a BFF) to reduce XSS impact
- [ ] Rethink client authentication: a `VITE_` secret is public by nature in a browser app
- [ ] Consistent, environment-driven CORS across all services
- [ ] OCR timeouts, concurrency limits, and cost controls
- [ ] Replace mock dashboard data with real analytics from the finance API
- [ ] Shared, versioned API contract (OpenAPI) to prevent DTO drift between services

## Run it locally

Each repository has its own README with setup steps. Typical local ports:

| Service | Port |
| --- | --- |
| `be-node-ts` (finance API) | 8080 |
| `be-auth-ts` (auth API) | 8081 |
| `be-ai-ocr-service` | 8888 |

Start order: databases, then `be-auth-ts`, `be-node-ts`, `be-ai-ocr-service`, and finally `fe-web`.

## About me

<!-- TODO: isi 2-3 kalimat singkat -->
Hi, I'm **[Your Name]**, a software engineer. I build FinTrack in my own time to go deeper on system design, security, and applied AI.

- LinkedIn: <!-- TODO -->
- Email: <!-- TODO -->
- Blog / notes: <!-- TODO -->

> This is a personal project, built independently on my own time and equipment, and contains no code or data from any employer.
