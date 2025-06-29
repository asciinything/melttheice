### Community Alert – Application Specification (v1.4)

**Status:** *Draft – updated 24 Jun 2025 to add open-source governance guidance*

# 1. Project Overview & Mission

## 1.1 Project Name
Community Alert (working title)

## 1.2 Mission Statement
Empower communities by sharing real-time, location-based information about immigration-enforcement activity so individuals can make informed safety decisions.

## 1.3 Guiding Principles
*   **User Anonymity & Privacy-by-Design** – minimise data collected, sever linkability wherever possible, and apply strong encryption in transit and at rest.
*   **Open Source & Community Collaboration** – core code licensed under an OSI-approved licence; transparent governance encourages broad contributor base.
*   **Operational Independence** – self-hosted infrastructure ensures predictable costs and avoids vendor lock-in.
*   **Legal & Ethical Compliance** – practices reviewed for CFAA/CISA compliance, plus a clear moderation policy to prevent harassment or misinformation.

# 2. Threat Model & Privacy Architecture

| Goal | Design Choice |
| :--- | :--- |
| **Prevent deanonymisation by attackers, operators, or governments** | • No accounts; use salted hash of device-ID for vote deduplication.<br>• Encrypt location & push-token columns at rest using `pgcrypto` AES. |
| **Limit retained sensitive data** | • Auto-delete reports & location rows after 24 h via nightly partition drop. |
| **Transport security** | • Enforce TLS 1.3 + HSTS; Let’s Encrypt auto-renew. |
| **Abuse & spam mitigation** | • Nginx `limit_req_zone` on `/reports` & `/vote`.<br>• Minimum credibility score before push dispatch. |

# 3. System Architecture & Technology Stack

## 3.1 Mobile Application (Frontend)
| Item | Choice | Rationale |
| :--- | :--- | :--- |
| **Framework** | React Native 0.73 (Bare) | Full native control; large talent pool. |
| **Key libs** | `react-native-maps`, `@react-native-community/geolocation`, CodePush (HoodieHQ fork), `@react-native-push-notification`. | |
| **OS policies** | Android 13+: request approximate location first; iOS 17+: background-location purpose string. | |

## 3.2 Backend Services
| Service | Tech | Notes |
| :--- | :--- | :--- |
| **API** | ASP.NET Core 8 in Docker | Leverages maintainer C# expertise. |
| **Queue** | Redis Streams | Decouples inserts from push dispatch; horizontal scaling. |
| **Push workers** | In-house C# interfacing with APNS (JWT) & FCM v1 (OAuth 2); prune invalid tokens. | |
| **Map tiles (optional)** | `openmaptiles/openmaptiles` container | Zero external calls. |

## 3.3 Data Layer
| Element | Choice | Operational Plan |
| :--- | :--- | :--- |
| **Primary DB** | PostgreSQL 16 + PostGIS | GIST index on `geom`; date-partitioned tables. |
| **Backups** | `pg_dump` nightly + WAL to Backblaze B2 | Weekly automated restore test via GH Actions. |

## 3.4 Over-the-Air Updates
| Item | Choice |
| :--- | :--- |
| **Server** | HoodieHQ CodePush Server in Docker; TLS via Caddy |
| **Workflow** | `appcenter cli` → CodePush -> Staging → Prod with signed bundles |

## 3.5 Hosting & IaC
*   **Provider:** Hetzner CX21 (2 vCPU / 4 GiB) primary; CX11 standby for DNS fail-over.
*   **IaC:** Terraform + Ansible rebuilds in < 30 min.
*   **Observability:** Prometheus + Grafana; Uptime Kuma external pings.

# 4. Feature Specification

### 4.1 Report a Sighting
*   **Endpoint:** `POST /reports` (HMAC-auth header).
*   **DB:** Insert into date-partitioned `reports`; push `report_id` to Redis Stream `sightings`.

### 4.2 Community Confirmation (Waze-like)
*   `POST /reports/{id}/vote` adjusts `credibility_score` ±1.
*   Score ≤ 0 → soft-delete + cancel queued pushes.

### 4.3 Proximity Alerts
*   Worker pulls `report_id` from Redis.
*   PostGIS `ST_DWithin` lookup for users within radius.
*   Batch-send via APNS/FCM (max 10 notifications per user / 24 h).

### 4.4 Over-the-Air Updates
*   Client checks CodePush on app launch/resume.
*   Bundles are code-signed & checksum-verified; auto-rollback after > 3 crashes.

# 5. Operational Run-Books

| Domain | Task | Frequency |
| :--- | :--- | :--- |
| **Server** | OS security patches (unattended-upgrades) | weekly |
| | Docker image rebuild via Watchtower | daily |
| **Database** | Verify restore from backups | weekly |
| | `VACUUM` & partition maintenance | daily |
| **On-call** | Alert if API p95 > 2 s or 5xx > 1 % | 24 × 7 |

# 6. Community Governance & Anti-Abuse
*   **Moderation Policy** – open document defining prohibited content (doxxing, hate speech, hoaxes).
*   **Transparency Report** – quarterly aggregate stats on report removals & authority requests.
*   **User Controls** – in-app toggle for alerts; GDPR-style data-deletion request.
*   **Rate-Limiting & Scoring** – endpoints guarded to throttle spam; credibility score gate for push notifications.

# 7. Open Source Project Governance & Contributor On-Ramp (new)

## 7.1 Licence
*   **Primary:** MIT for client & server code (maximises adoption).
*   **Exception:** Any third-party code included under its own OSI-approved licence must be preserved.

## 7.2 Governance Model
| Role | Responsibilities | Election / Appointment |
| :--- | :--- | :--- |
| **Core Maintainers (2–4 ppl)** | Roadmap, code review, security response, releases. | BDFL-style for now; evolve to Maintainer Committee chosen by majority of active contributors (> 5 merged PRs in last 12 m). |
| **Contributors** | PRs, issue triage, docs. | Anyone passing CLA & CoC. |
| **Steering Advisors** | Legal/privacy guidance, community reps. | Invited annually by core team. |

## 7.3 Repository Setup (GitHub)
*   `main` – always deployable; protected branch.
*   `dev` – integration branch; squash-merge PRs.
*   Release tags `vX.Y.Z`; semantic versioning.
*   GitHub Actions: test → build Docker → push to GHCR; raise draft release.

## 7.4 Contribution Workflow
1.  Fork → feature branch → PR against `dev`.
2.  CI runs lint, unit tests, integration tests (PostGIS + API).
3.  At least 1 maintainer review + green CI → squash merge.
4.  `dev` auto-deploys to staging VPS via watchtower; feedback cycle.
5.  Monthly or on demand, merge `dev` → `main`, cut release, publish changelog.

## 7.5 Code of Conduct & Community Health
*   Adopt Contributor Covenant v2.1 (`CODE_OF_CONDUCT.md`).
*   Use Issue/PR templates for bug, feature, docs.
*   Discuss design changes in GitHub Discussions → tag `RFC`.
*   Monthly community call (Jitsi) with meeting notes in `/docs/meeting-minutes`.

## 7.6 Security Policy
*   `/SECURITY.md` with contact (`security@domain`) + PGP key.
*   90-day public disclosure window unless earlier patch available.

## 7.7 Funding & Sustainability
*   OpenCollective for donations → cover domain, VPS, CI costs.
*   GitHub Sponsors tier for recurring support.
*   Bounties on critical issues to encourage outside contribution.

# 8. Roadmap (6–12 months)

| Quarter | Milestone |
| :--- | :--- |
| **Q3 2025** | MVP launch on TestFlight / Internal App Sharing; live pilot in one city |
| **Q4 2025** | SMS fallback alerts via SimpleSMS; heat-map clustering UI |
| **Q1 2026** | Federation prototype (ActivityPub); secondary read replica |

# 9. Appendices

## 9.1 Glossary
*   **APNS** – Apple Push Notification Service
*   **FCM** – Firebase Cloud Messaging
*   **GIST** – Generalised Search Tree index (Postgres)
*   **HSTS** – HTTP Strict-Transport-Security

## 9.2 Reference Links
*   [HoodieHQ CodePush fork](https://github.com/hoodiehq/react-native-code-push-server)
*   [React Native OTA RFC](https://github.com/react-native-community/discussions-and-proposals/pulls)
