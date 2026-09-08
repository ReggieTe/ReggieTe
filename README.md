# Hi, I'm Regis Tembachako 👋

**Full stack engineer — PHP 8 / Symfony 7/Laravel · Adobe Commerce / Magento 2 · Flutter & Dart · Python**
South Africa · open to relocation (UK Skilled Worker · AU 482 · NZ AEWV)

8 years building and *operating* production systems — backend, mobile and data. I design the schema, write the domain services, build the web and mobile clients on top, and run the thing in production afterwards. I care most about how a system behaves when a dependency dies, not about the happy path.

[![Website](https://img.shields.io/badge/rtembachako.com-1f6feb?style=flat-square&logo=googlechrome&logoColor=white)](https://rtembachako.com)
[![EasyApply](https://img.shields.io/badge/easyapply.co.za-0b7285?style=flat-square&logo=googlechrome&logoColor=white)](https://easyapply.co.za)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0a66c2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/registembachako)
[![Email](https://img.shields.io/badge/tembachakoregis@gmail.com-24292f?style=flat-square&logo=gmail&logoColor=white)](mailto:tembachakoregis@gmail.com)

---

## 🚀 What I'm building — EasyApply

A 14-service commerce and services platform I architect and operate solo. It's where most of the work below actually runs.

| | |
|---|---|
| **Backend** | Symfony 7 modular monolith — 3,100+ classes, 530+ entities, 115 scheduled and CLI workers |
| **Web** | 15 branded storefronts from one Twig component system and one theme layer |
| **Mobile** | 14 branded Flutter apps from one codebase — 1,900+ Dart files, 595 screens, 29 locales |
| **Stores** | Live on Google Play · App Store · Huawei AppGallery |
| **Ops** | MariaDB, Redis, Docker, Nginx on a self-managed VPS fleet behind Cloudflare, CI/CD via GitHub Actions |

Because there's no one else on it, I've had to be good at the parts that are usually somebody else's job: migrations, queue architecture, payments and reconciliation, webhook signing, health checks, circuit breakers, cost control.

---

## 🧱 Selected engineering work

<details open>
<summary><b>Platform architecture — 14 services, one codebase, one deployment</b></summary>

- Modular monolith carrying 14 distinct product verticals with **shared domain code** rather than 14 near-copies — one billing rail, one payment router, one notification layer, one moderation path.
- Extension is a matter of adding a class: interfaces plus DI auto-tagging mean a new health check, automation capability or payment provider is one implementation and one service definition — never an edited switch.
- Business rules live in domain services consumed by both the web front end and the mobile API, so a calculation can't drift between clients — a rule I enforce in review *because it had already drifted once*.
- 115 scheduled workers for billing, settlement, reminders, reconciliation and indexing, with a run log that makes a job which silently did nothing diagnosable rather than invisible.

</details>

<details>
<summary><b>Correctness where it costs money — payments, refunds and settlement</b></summary>

- The full money path: checkout across several payment providers, an internal wallet ledger, commission splits, release of held funds to the merchant on fulfilment, and refunds — automatic where the provider supports it, queued for manual handling where it doesn't, **never silently dropped**.
- One invoice and receipt service rendering HTML and PDF and emailing asynchronously, wired into every payment path instead of reimplemented per flow. Recurring merchant fees ride the same rail, so there's a single place where an amount owed becomes a document.
- Fixed a real gap where funds for delivered orders stayed platform-side because nothing released them, and a credit-consumption bug spending future bookings against today's bill.
- Multi-currency resolved per country at persist time after an audit found one currency being stamped onto every market.
- Merchant-facing detail that decides whether commerce software is usable: per-venue payment methods and credentials with a fallback chain, VAT-inclusive pricing as a per-merchant setting, split bills settled by several people from their own devices.

</details>

<details>
<summary><b>Mobile — 14 apps, three stores, one codebase</b></summary>

- Gradle product flavours on a single `service` dimension: each app gets its own application id, name, icons, splash and feature set — instead of 14 near-copies that would have drifted within a release.
- **Branch-driven releases**: pushing `release/android/<flavour>` is the only input that selects what ships. Three Fastlane lanes (Play Internal Testing, TestFlight, AppGallery); production is a manually dispatched workflow bound to a per-flavour GitHub Environment with required reviewers, at a 10% staged rollout.
- Store policy encoded rather than remembered: the loans app is stripped from the AppGallery build where financial-services content is barred by review rule 1.14, and that build regenerates its manifest to drop the `REQUEST_INSTALL_PACKAGES` permission Play requires be removed — one codebase satisfying two contradictory review rules.
- **Offline-first** sqflite layer behind a typed table/model abstraction with 20 domain sync units, plus a caching API client with TTL, stale-while-revalidate, and invalidation keyed to the mutations that would make an entry wrong.
- Mercure SSE realtime through a connection manager, and FCM + Huawei HMS push behind one `PushTransport` interface, so app code never branches on vendor.

</details>

<details>
<summary><b>Adobe Commerce / Magento 2</b></summary>

- Custom modules across enterprise B2C storefronts — catalogue, checkout and admin-side behaviour — worked from functional specifications and architectural constraints.
- Customised through the platform's extension points: plugins and interceptors, observers, DI preferences, service contracts. Upgrades stay possible; nothing lands as a core edit.
- Multi-store and multi-website configuration with the scope-aware config, attribute and content differences that come with it.
- Performance as a *measured* change: Redis for session and cache backends, Varnish as full-page cache, indexer and cron hygiene, query and index work — profile, fix, re-measure.
- Bespoke ERP/CRM connectors where no supported extension existed; payment and shipping integrations designed for the failure case first.

</details>

<details>
<summary><b>Security engineering</b></summary>

- Global **Content Security Policy with per-request nonces** enforced in one event subscriber, so an injected inline script doesn't execute regardless of which template rendered it.
- Outbound webhooks signed with HMAC-SHA256 over `timestamp.body` under a bounded replay window; inbound verified in constant time.
- Refused to give a self-hosted automation engine database credentials — everything it may ask the app to do is an explicit, reviewable capability class behind one interface, so it can't bypass validation or domain listeners.
- User-selectable 2FA (TOTP plus a custom email/SMS one-time-code provider) and express passwordless sign-in for the customer-facing ordering flow.
- Certificate pinning on the mobile HTTP client, with the auth token kept out of headers and off disk in plain form.
- OWASP Top 10 hardening on a production PHP platform, with zero post-deployment security incidents.

</details>

<details>
<summary><b>Reliability, observability and graceful degradation</b></summary>

- Pluggable health-check framework — 31 checks over 6 tiers — as both a CLI command and an admin dashboard, where adding a dependency means adding one check class.
- Dependency failure made survivable: object storage degrades to local disk behind a **circuit breaker** and reconciles afterwards, instead of turning a suspended bucket into a login outage.
- A single-source spend brake for metered external APIs: kill switch, per-operation daily caps derived from a monthly ceiling, per-IP limits — splitting the "may we call?" read from the "record the spend" write so cache hits never consume budget.
- Exception triage with back-pressure: reports deduplicated by fingerprint, stamped with the release SHA, mapped to the ticket already open for that fingerprint — one outage is one ticket, not thousands of events.

</details>

<details>
<summary><b>Data & Python</b></summary>

- 530+ entities of hand-owned schema, including hand-written migrations where the generator couldn't be trusted against a drifted database, and collation-aware foreign keys after mismatched collations made constraints fail.
- Cut page load times by up to **35%** on a large citizen-facing platform through query optimisation, indexing and server-side caching.
- A Python ETL ingesting public government sanctions publications, normalising them into FollowTheMoney entities with alias enrichment and emitting a versioned manifest that triggers a reindex — replacing a per-call commercial API with an owned, auditable instance behind Elasticsearch.
- Screening **fails closed**: an unavailable check is never recorded as a clean result.

</details>

<details>
<summary><b>Front end</b></summary>

- A shared themed front-end system across 15 branded storefronts — one set of partials, one theme layer keyed per brand — so a design change lands everywhere at once.
- A full customer-facing ordering interface (menu browsing, cart, checkout, reservations, reviews, order tracking) in Twig and JavaScript, mirrored feature-for-feature by a Flutter client against the same API.
- Real Core Web Vitals damage measured and fixed: an unsized lazy hero image causing a **0.97 CLS**, render-blocking fonts and icons moved to async loading, and an auto-generated icon subset instead of a full icon font.
- ACE-certified on the Adobe Commerce front-end track — theme and layout customisation, LESS/CSS, Hyvä.

</details>

<details>
<summary><b>LLM features in production — with the guard rails</b></summary>

- AI shipped into revenue-bearing paths rather than prototypes: per-recipient outbound personalisation, content moderation for toxicity/fraud/PII exposure, and PDF-to-domain-object extraction — every one pinned to a JSON schema so nothing downstream parses free text.
- Failure behaviour designed deliberately: a refusal degrades to a hand-authored template, a transport error marks the record for retry, moderation thresholds may escalate a verdict but never soften one, and extracted structure is validated in typed code before anything persists.
- Scraped web content treated as attacker-controlled and model output sanitised before reaching an HTML sink, closing the prompt-injection-to-markup path.
- AI-assisted development in daily delivery — with human review as the final gate on everything merged.

</details>

---

## 🛠️ Tech

**Languages** &nbsp;`PHP 8.x` `Python` `Dart` `JavaScript` `SQL` `HTML5` `CSS3/LESS` `XML`

**Frameworks** &nbsp;`Symfony 7` `Doctrine ORM` `Twig` `Laravel` `Flutter` `Vue.js` `Magento 2 / Adobe Commerce`

**Architecture** &nbsp;domain-driven service design · DI and auto-tagged extension points · Symfony Messenger queues · cron and worker fleets · REST & GraphQL API design · Mercure realtime · HMAC-signed webhooks with replay protection · circuit breakers and graceful degradation

**Data** &nbsp;`MySQL / MariaDB` `SQLite` `Redis` `Varnish` `Elasticsearch / OpenSearch` — schema design, query optimisation, indexing, hand-written migrations, ETL, entity resolution

**Mobile** &nbsp;Android (Gradle flavours, manifest merging, foreground services, permissions) · iOS (Xcode, App Store Connect, TestFlight) · Huawei HMS / AppGallery · Provider state management · offline-first sync · deep linking and notification routing · i18n across 29 locales

**Cloud & DevOps** &nbsp;`Docker` `GitHub Actions` `Fastlane` `Nginx` `Cloudflare` `AWS (EC2, S3, CodePipeline)` `GCP (Vertex AI, Storage, Maps, Firebase)` · self-managed Linux VPS · CI/CD · observability and health-check frameworks

**Security** &nbsp;OWASP Top 10 · CSP with nonces · CSRF/XSS/SQLi defence · 2FA · KYC/AML integration · certificate pinning · least-privilege capability design · threat modelling

**AI & automation** &nbsp;Anthropic Messages API (schema-constrained output, document blocks, Batch API) · Google Gemini on Vertex AI · multi-agent orchestration over MCP · self-hosted n8n · Claude Code and Cursor

---

## 🎓 Certifications

**7× Adobe Certified** — three at Expert level (ACE)

| Certification | Level | Date |
|---|---|---|
| Adobe Commerce Developer | **ACE** | Aug 2025 (exp. 2027) |
| Adobe Commerce Business Practitioner | **ACE** | Jul 2025 |
| Adobe Commerce Front-End Developer | **ACE** | Jun 2025 |
| Adobe Commerce Developer | Professional | Jul 2024 |
| Adobe Commerce Business Practitioner | Professional | May 2024 |
| Adobe Commerce Front-End Developer | Professional | Apr 2024 |
| Professional Developer Adobe Commerce (AD0-E717) · SwiftOtter | — | Aug 2024 |

**BSc Honours in Information Technology, Software Engineering** — Chinhoyi University of Technology, Zimbabwe

---

## 📊 GitHub

<p>
  <img src="https://github-readme-stats.vercel.app/api?username=ReggieTe&show_icons=true&hide_border=true&count_private=true&theme=transparent" height="150" alt="GitHub stats" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=ReggieTe&layout=compact&hide_border=true&theme=transparent" height="150" alt="Top languages" />
</p>

---

## 📬 Get in touch

Open to senior backend, full stack, mobile and platform roles — remote, or relocation with sponsorship.

📧 **tembachakoregis@gmail.com** · 🌐 [rtembachako.com](https://rtembachako.com) · 💼 [LinkedIn](https://linkedin.com/in/registembachako)
