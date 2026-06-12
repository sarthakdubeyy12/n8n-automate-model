# 🚀 Abacus ERP × n8n Integration Portal
### **Enterprise-Grade Automation & Interactive API Gateway**

![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)

---

## 🌟 Key Highlights
- **Interactive Gateway Explorer**: A React-powered console for developers to test ERP modules in real-time with deep-linking support.
- **Resilient "Demo Mode"**: Built-in mock fallbacks ensure the platform remains functional for testing even during upstream service maintenance.
- **Standards Compliant**: Fully typed custom n8n node following the latest n8n community standards and npm provenance security.

---

# Abacus ERP × n8n Integration

> A production-ready n8n community node for automating [Abacus ERP](https://www.abacus.ch) — built with a local mock server for development, an API gateway for routing, and a fully typed TypeScript node published to npm.

**📚 Interactive API Documentation:** [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)  
All endpoints, request/response schemas, and live testing available via Swagger UI when the mock server is running.

---

## What This Project Is

[Abacus ERP](https://www.abacus.ch) is a Swiss enterprise resource planning system used by thousands of businesses. It exposes a REST API, but getting access to a real instance requires a paid license and a service-user setup — which makes development and testing painful.

This project solves that problem end-to-end:

1. **Mock Server** — a Python FastAPI server that speaks the exact same REST API language as real Abacus. No license needed. Run it locally, test everything.
2. **API Gateway** — a lightweight TypeScript/Express proxy that sits between n8n and the Abacus API, handling routing and environment-specific config.
3. **n8n Community Node** — the main deliverable. A fully typed TypeScript node you install into n8n that lets you automate Abacus ERP operations in any workflow — no HTTP Request node needed.

---


## 🚀 Live API Docs (Swagger UI)

Explore and test all endpoints interactively:

👉 **http://127.0.0.1:8000/docs**

Includes:
- 🔐 OAuth2 Authentication (token flow)
- 📍 Addresses
- 👤 Customers
- 🧾 Invoices
- 📦 Orders
- 📁 Projects
- 📊 Request/Response schemas
- 🧪 Try-it-out functionality

> ⚠️ Note: The docs are available only when the mock server is running locally.

---


## Repository Structure

```
/
├── abacus-mock-server/        # Python FastAPI mock of the Abacus REST API
├── api-gateway/               # TypeScript/Express API gateway
├── n8n-nodes-abacus/          # n8n community node (npm package)
└── README.md                  # This file
```

---

## Folder 1 — `abacus-mock-server/`

### What it does

A fake Abacus ERP server. It pretends to be a real Abacus instance so you can develop and test the n8n node without needing a real ERP license. It speaks the exact same REST API language that real Abacus uses.

### Why Python + FastAPI

- **FastAPI** gives automatic OpenAPI/Swagger docs at `/docs` — invaluable for exploring endpoints during development
- **Pydantic v2** handles request validation and response serialization with zero boilerplate
- **SQLAlchemy** with SQLite locally and PostgreSQL in production — same codebase, different `DATABASE_URL`
- **Uvicorn** is the fastest Python ASGI server available, with `--reload` for instant hot-reloading during dev

### Architecture

```
abacus-mock-server/
├── app/
│   ├── core/
│   │   ├── config.py          # Pydantic settings — one source of truth for all env vars
│   │   ├── database.py        # SQLAlchemy engine (SQLite/PostgreSQL auto-detection)
│   │   └── lifecycle.py       # FastAPI lifespan — DB init on startup
│   ├── models/                # SQLAlchemy ORM models (5 tables)
│   ├── schemas/               # Pydantic request/response schemas
│   ├── routers/               # FastAPI route handlers (one file per resource)
│   ├── services/              # Business logic (one file per resource)
│   └── utils/
│       ├── auth.py            # Bearer token verification
│       ├── helpers.py         # get_record_or_404, apply_partial_update
│       └── responses.py       # Consistent { success, data } envelope
├── seed.py                    # Faker-based DB seeder (100 records per table)
├── test_endpoints.py          # Python smoke tests (35 assertions)
├── render.yaml                # One-click Render.com deployment
├── requirements.txt
└── .env.example
```

### Data Models

| Model | Table | ID Format | Key Fields |
|---|---|---|---|
| `AddressModel` | `addresses` | UUID | `firstName`, `lastName`, `street`, `zip`, `city`, `createdAt` |
| `CustomerModel` | `customers` | `CUST-XXXXXX` | `name`, `email` (unique), `addressId` FK |
| `ProjectModel` | `projects` | `PROJ-XXXXXX` | `projectName`, `status`, `customerId` FK |
| `InvoiceModel` | `invoices` | `INV-XXXXXX` | `customerId` FK, `amount`, `currency` (CHF), `status` |
| `OrderModel` | `orders` | `ORD-XXXXXX` | `customerId` FK, `totalAmount` |

Prefixed IDs (`CUST-A3F9B2`) are intentional — they mirror how real Abacus ERP formats its IDs, making the mock indistinguishable from the real thing.

### API Endpoints

All resource endpoints follow the same RESTful pattern:

```
GET    /rest/v1/{resource}/          → list with limit/offset pagination
GET    /rest/v1/{resource}/{id}      → get one, 404 if not found
POST   /rest/v1/{resource}/          → create, returns 201
PATCH  /rest/v1/{resource}/{id}      → partial update (only provided fields)
DELETE /rest/v1/{resource}/{id}      → delete, returns 204
```

Auth endpoints:

```
GET  /.well-known/openid-configuration   → token + API endpoint discovery (no auth)
POST /oauth/token                        → OAuth2 Client Credentials (form data)
GET  /rest/v1/info                       → ERP version info (requires token)
```

### Why the response envelope matters

Every response is wrapped in `{ "success": true, "data": ... }`. This is consistent with how real Abacus formats responses and makes error handling in the n8n node predictable — you always know where the data is.

### Running locally

```bash
cd abacus-mock-server
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload
```

**→ Server runs at:** `http://127.0.0.1:8000`  
**→ Interactive API docs (Swagger UI):** `http://127.0.0.1:8000/docs`

The `/docs` endpoint gives you a full interactive API explorer where you can test every endpoint, see request/response schemas, and execute calls directly from your browser.

### Seeding test data

```bash
python seed.py
# Inserts 100 fake records per table using Faker
```

### Running the test suite

```bash
python test_endpoints.py
# 35/35 assertions covering all CRUD operations, auth, pagination, and error cases
```

### Deploying to Render.com

```bash
# render.yaml is already configured
# Push to GitHub → connect repo in Render → deploy
# PostgreSQL is provisioned automatically
```

---

## Folder 2 — `api-gateway/`

### What it does

A TypeScript/Express API gateway that sits between n8n and the Abacus API. Handles environment-specific routing, so the n8n node always talks to the same gateway URL regardless of whether you're hitting the mock server or a real Abacus instance.

### Why a gateway

- Decouples the n8n node from the Abacus instance URL
- Lets you swap between mock and production without changing n8n credentials
- Central place to add logging, rate limiting, or request transformation later

### Running locally

```bash
cd api-gateway
npm install
cp .env.example .env
npm run dev
# → http://localhost:3000
```

---

## Folder 3 — `n8n-nodes-abacus/`

### What it does

The main deliverable. A custom n8n community node that lets you automate Abacus ERP operations in any n8n workflow. Install it once, then drag it into any workflow — no HTTP Request node, no manual auth headers, no JSON wrangling.

### Why TypeScript

- n8n's entire codebase is TypeScript — using it means full type safety against `n8n-workflow` interfaces
- Compile-time errors catch mistakes before they reach production
- `INodeType`, `IExecuteFunctions`, `IDataObject` — all properly typed, no guessing

### Why zero runtime dependencies

n8n's community node standard requires zero runtime dependencies. Everything must be available at compile time or provided by n8n itself. This keeps the installed package tiny and avoids version conflicts inside n8n's own dependency tree.

### Architecture — 4 Layers

```
Layer 1: UI         → descriptions/*.ts      (what the user sees in n8n)
Layer 2: Controller → Abacus.node.ts         (routes resource+operation to service)
Layer 3: Service    → services/*.service.ts  (builds payloads, calls API layer)
Layer 4: API        → GenericFunctions.ts    (HTTP client, auth, retry, pagination)
```

Each layer has one job. Adding a new resource means adding one description file and one service file — the controller and HTTP engine don't change.

### Project Structure

```
n8n-nodes-abacus/
├── src/
│   ├── credentials/
│   │   └── AbacusApi.credentials.ts    # OAuth2 credential definition
│   └── nodes/
│       └── Abacus/
│           ├── Abacus.node.ts          # Main node — metadata + execute()
│           ├── GenericFunctions.ts     # HTTP engine — apiRequest, pagination, retry
│           ├── abacus.svg              # Node icon shown in n8n UI
│           ├── descriptions/           # UI definitions (10 files)
│           │   ├── AddressDescription.ts
│           │   ├── CustomerDescription.ts
│           │   ├── InvoiceDescription.ts
│           │   ├── OrderDescription.ts
│           │   ├── ProjectDescription.ts
│           │   ├── SubjectDescription.ts
│           │   ├── PaymentDescription.ts
│           │   ├── DeliveryNoteDescription.ts
│           │   ├── FinancialAccountDescription.ts
│           │   └── ItemDescription.ts
│           └── services/               # Business logic (10 files)
│               ├── address.service.ts
│               ├── customer.service.ts
│               ├── invoice.service.ts
│               ├── order.service.ts
│               ├── project.service.ts
│               ├── subject.service.ts
│               ├── payment.service.ts
│               ├── delivery-note.service.ts
│               ├── financial-account.service.ts
│               └── item.service.ts
├── tests/
│   └── mock-server.test.mjs            # Integration tests (47 assertions)
├── scripts/
│   └── jsonplaceholder-mock.mjs        # Offline testing helper
├── docs/
│   ├── coverage-matrix.md              # Which resources are implemented
│   └── release-checklist.md           # Pre-publish checklist
├── .github/
│   └── workflows/
│       ├── ci.yml                      # Lint + build on every push/PR
│       └── publish.yml                 # npm publish with provenance on release
├── dist/                               # Compiled output (committed to npm, not git)
├── package.json
├── tsconfig.json
└── .env.example
```

### Supported Resources

| Resource | Get | Get All | Create | Update | Delete | Search |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Address | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Customer | ✅ | ✅ | ✅ | ✅ | — | ✅ |
| Invoice | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Order | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Project | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Subject | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Payment | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Delivery Note | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Financial Account | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Item | ✅ | ✅ | ✅ | ✅ | ✅ | — |

### Authentication — How it works

The node uses **OAuth2 Client Credentials** — the standard machine-to-machine auth flow. Here's exactly what happens:

1. User fills in `instanceUrl`, `clientId`, `clientSecret` in the n8n credential form
2. n8n hits `{instanceUrl}/.well-known/openid-configuration` to auto-discover the token endpoint
3. n8n exchanges `client_id` + `client_secret` for a Bearer token via `POST /oauth/token` (form data — required by Abacus)
4. Every subsequent API call includes `Authorization: Bearer {token}` automatically
5. On 401 → token is refreshed once and the request is retried

Why form data and not JSON? Because the real Abacus OAuth2 endpoint requires `application/x-www-form-urlencoded`. The mock server matches this exactly.

### Error Handling

Every error case is handled explicitly — no silent failures:

| Status | Behaviour |
|---|---|
| `401` | Refresh OAuth2 token, retry once. If still 401 → throw with "Authentication failed" |
| `404` | Throw `NodeApiError` with "Resource not found at {endpoint}" |
| `409` | Throw with "Conflict — resource already exists" |
| `429` | Read `Retry-After` header (or exponential backoff: 1s → 2s → 4s, max 10s), retry up to 3 times |
| `500` | Throw with "Abacus server error — try again later" |
| Any other 4xx/5xx | Throw with HTTP status code in description |

### Pagination

The `Get All` operation handles pagination automatically:

```
pageSize = clamp(requestedLimit, 1, 500)

loop:
  GET /endpoint?limit={pageSize}&offset={currentOffset}
  collect items
  if page.length < pageSize → last page, stop
  if collected.length >= requestedLimit → stop
  currentOffset += pageSize

return collected.slice(0, requestedLimit)
```

This means you can request `limit=1000` and the node will make as many API calls as needed, transparently.

### Data Normalization

Raw API responses are normalized before being passed to n8n:

- **All ID fields** (`id`, `customerId`, `addressId`, etc.) are coerced to strings — n8n expects string IDs
- **Date fields** matching ISO 8601 patterns are converted to proper ISO 8601 strings
- **Nested objects** are flattened using dot notation: `{ address: { city: "Zurich" } }` → `{ "address.city": "Zurich" }` — making them usable in n8n expressions like `{{ $json["address.city"] }}`

### Installing in n8n

**Via npm (recommended):**
```bash
npm install n8n-nodes-abacus
```

**Via n8n UI:**
Settings → Community Nodes → Install → `n8n-nodes-abacus`

**For local development (link mode):**
```bash
cd n8n-nodes-abacus
npm run build
npm link
# In your n8n custom extensions directory:
npm link n8n-nodes-abacus
```

### Setting up credentials

1. In n8n, go to **Credentials → New → Abacus ERP API**
2. Fill in:
   - **Instance URL**: `http://localhost:8000` (mock) or `https://your-company.abacus.ch` (production)
   - **Client ID**: your service user client ID
   - **Client Secret**: your service user secret
   - **Ignore SSL Issues**: enable only for local dev
3. Click **Test** — n8n hits `/.well-known/openid-configuration` and confirms the connection

### Building from source

```bash
cd n8n-nodes-abacus
npm install
npm run build       # compiles TypeScript → dist/
npm run dev         # watch mode
npm run lint        # ESLint
npm run lint:fix    # auto-fix
```

### Running integration tests

Make sure the mock server is running first:

```bash
# Terminal 1
cd abacus-mock-server && uvicorn app.main:app --reload

# Terminal 2
cd n8n-nodes-abacus && node tests/mock-server.test.mjs
# → 47/47 passed
```

### CI/CD

Two GitHub Actions workflows:

**`ci.yml`** — runs on every push and pull request:
- Installs dependencies
- Runs ESLint
- Runs `tsc` build — fails if any TypeScript error exists

**`publish.yml`** — runs when a GitHub Release is published:
- Builds the package
- Publishes to npm with `--provenance` flag (npm provenance attestation — mandatory for trusted packages)
- Requires `NPM_TOKEN` secret in repository settings

To release:
```bash
git tag v0.2.0
git push origin v0.2.0
# Create a GitHub Release from the tag → publish.yml triggers automatically
```

---

## End-to-End Development Flow

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   n8n workflow                                              │
│       │                                                     │
│       ▼                                                     │
│   [Abacus ERP node]  ←── AbacusApi credential              │
│       │                   (instanceUrl, clientId, secret)   │
│       │                                                     │
│       ▼                                                     │
│   api-gateway (localhost:3000)                              │
│       │                                                     │
│       ▼                                                     │
│   abacus-mock-server (localhost:8000)                       │
│       │                                                     │
│       ▼                                                     │
│   SQLite database (abacus.db)                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

For production, replace `abacus-mock-server` with a real Abacus instance and update the `instanceUrl` in the n8n credential. Everything else stays the same.

---

## Tech Stack Decisions

| Component | Technology | Why |
|---|---|---|
| Mock server | Python + FastAPI | Auto Swagger docs, Pydantic validation, fastest iteration speed |
| ORM | SQLAlchemy | Works with both SQLite (dev) and PostgreSQL (prod) from the same codebase |
| Config | pydantic-settings | Type-safe env vars, `.env` file support, one import everywhere |
| API gateway | TypeScript + Express | Lightweight, familiar, easy to extend |
| n8n node | TypeScript | Required by n8n, full type safety against n8n-workflow interfaces |
| Auth | OAuth2 Client Credentials | Industry standard for machine-to-machine auth, matches real Abacus |
| Deployment | Render.com | Free tier PostgreSQL, zero-config Python deployment via `render.yaml` |
| CI/CD | GitHub Actions | Native GitHub integration, free for public repos, provenance support |

---

## Security

- No secrets are hardcoded anywhere — all credentials come from `.env` files
- `.env` is in `.gitignore` in every subfolder — it will never be committed
- OAuth2 tokens are managed entirely by n8n — the node never stores them
- `ignoreSSLIssues` is a boolean toggle, disabled by default, only for local dev
- The mock server validates every request against a Bearer token — no unauthenticated access to resource endpoints

---

## License

MIT — use it, fork it, ship it.
