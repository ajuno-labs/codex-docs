# Codex – Authentication Design

*Applies to the **docs** repo – reference for **api** & **web** implementers*

> **Scope:** MVP sign‑in / sign‑up for a single‑page app using Laravel 12 (API) + Vue 3 (Web). Supports e‑mail/password and OAuth (Google, GitHub) with refresh‑token rotation.

---

## 1 · Goals & Non‑Goals

| #   | Goal                                              | Metric / Acceptance                                   |
| --- | ------------------------------------------------- | ----------------------------------------------------- |
| G1  | Secure, stateless auth for SPA and mobile clients | Access token expiry ≤ 15 min; refresh <= 7 days       |
| G2  | Social login first, password fallback             | 80 % users choose OAuth providers                     |
| G3  | Protect against CSRF/XSS/Replay                   | All tokens httpOnly; refresh rotation; device‑binding |
| NG1 | Multi‑tenant SSO (Okta / SAML)                    | Enterprise roadmap, not MVP                           |
| NG2 | Fine‑grained RBAC                                 | All users same role; admins via env flag              |

---

## 2 · Token Strategy

| Token       | Format            | TTL    | Location                             | Rotation                 |
| ----------- | ----------------- | ------ | ------------------------------------ | ------------------------ |
| **Access**  | JWT (HS512)       | 15 min | memory (Store)                       | not rotated              |
| **Refresh** | JWT (HS512) + jti | 7 days | httpOnly, Secure SameSite=Lax cookie | rotated on every refresh |

*Why JWT?* – Works across sub‑domains, no server session state; easy mobile integration.  *Laravel Sanctum’s token mode* issues opaque tokens → we wrap them as signed JWTs for stateless validation.

---

## 3 · High‑Level Flows

### 3.1 Email / Password Sign‑Up

```mermaid
sequenceDiagram
User->>Web: POST /register
Web->>API: /api/auth/register {email, pwd}
API->>DB: INSERT users, hashed pwd (Argon2id)
API-->>Web: 201 + {access, refresh}
Web->>Browser: store access; set-cookie(refresh)
```

### 3.2 OAuth Social Login (Google)

1. Web calls `/api/auth/oauth/google/url` → returns Google auth URL with state.
2. Browser redirects user to Google Consent.
3. Google redirects back to `/oauth/google/callback?code=…&state=…`.
4. API exchanges `code` for profile; upserts user; mints tokens.
5. API sets `refresh` cookie; returns `access` to Web.

### 3.3 Token Refresh

1. Web detects 401 or access expiry ⇒ POST `/api/auth/refresh` (no body).
2. API validates `refresh` cookie **and** jti not‑revoked; issues new pair, rotates refresh (blacklists old jti).

### 3.4 Logout (All Devices)

* Web POST `/api/auth/logout` → API deletes refresh jti row + sets `refresh=; Max‑Age=0` cookie.

---

## 4 · API Contract (OpenAPI excerpt)

| Endpoint                    | Verbs  | Body / Params               | Response                           |
| --------------------------- | ------ | --------------------------- | ---------------------------------- |
| `/auth/register`            | `POST` | `{email, password}`         | `{access}` + `Set‑Cookie: refresh` |
| `/auth/login`               | `POST` | `{email, password}`         | same                               |
| `/auth/oauth/:provider/url` | `GET`  | provider ∈ {google, github} | `{redirect_url}`                   |
| `/oauth/:provider/callback` | `GET`  | `code`, `state`             | 302 + tokens                       |
| `/auth/refresh`             | `POST` | – (cookie only)             | `{access}` + new `refresh` cookie  |
| `/auth/logout`              | `POST` | –                           | 204                                |

HTTP status: 200/201 success; 401 invalid token; 400 validation; 429 rate‑limited.

---

## 5 · Database Changes (api repo)

```sql
CREATE TABLE users (
  id UUID PK DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT, -- nullable for pure social
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE refresh_tokens (
  jti UUID PK,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  expires_at TIMESTAMPTZ,
  revoked BOOLEAN DEFAULT FALSE,
  created_ip INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
CREATE INDEX ON refresh_tokens(user_id);
```

---

## 6 · Front‑End Integration (web repo)

* Uses **Pinia Auth Store** holding `access` in memory (reset on tab close).
* Axios interceptor attaches `Authorization: Bearer {access}`.
* Second interceptor catches 401 ⇒ triggers silent refresh flow; queues pending requests.
* Login / Sign‑up forms under `/auth` route; Google / GitHub buttons call `getOAuthUrl`.
* Protect routes with `router.beforeEach` checking `authStore.isLoggedIn`.

> *Security note*: Avoid `localStorage`; access token lives only in JS memory; refresh stays in httpOnly cookie (XSS‑safe).

---

## 7 · Security Hardening

* Hash passwords with **Argon2id** (`password_hash()` cost=4, memory=64 MB).
* **Rate‑limit** `/auth/*` routes (Laravel Throttle: 5/min IP+email).
* **CSRF**: refresh cookie is SameSite=Lax; `/auth/refresh` requires CSRF header for browsers; mobile apps send header token.
* **Device Fingerprint**: optional hash of UA + IP stored with refresh jti to detect anomaly.
* **2FA**: TOTP planned Post‑MVP (add `user_totp_secrets`).

---

## 8 · Test Plan

| Layer       | Tool      | Cases                                     |
| ----------- | --------- | ----------------------------------------- |
| API unit    | PHPUnit   | token mint, hash verify, jti blacklist    |
| API feature | Pest      | /register, /login, /refresh, misuse cases |
| Web e2e     | Cypress   | happy flows, refresh rotation, logout     |
| Security    | OWASP ZAP | CSRF, cookie flags, header check          |

---

## 9 · Roll‑Out Milestones

| Sprint | Deliverable                                     |
| ------ | ----------------------------------------------- |
| 0      | DB migrations; Laravel Sanctum + JWT helper lib |
| 1      | Email/password routes + Vue forms               |
| 2      | OAuth Google & GitHub integration               |
| 3      | Refresh rotation & logout everywhere            |
| 4      | Hardening, rate‑limit, Cypress e2e              |

---

## 10 · Open Questions

1. Use **Laravel Passport** instead of custom JWT? (heavier)
2. Where to store revoked jti – Redis vs Postgres table? (table chosen for audit)
3. Need remember‑me cookie (>30 days) or rely on refresh rotation?

---

*End of doc – comments welcome via PR review.*
