# The Enterprise Software Construction Primer

## Building Global Platforms with AI-Assisted Engineering

---

> **"If you wouldn't expect a first-day junior developer to succeed without it, don't expect AI to succeed without it either."**

---

## Who This Primer Is For

This primer is for engineering teams building enterprise-grade software — platforms that must operate globally, handle real money, comply with regulations across jurisdictions, and scale under pressure. It is specifically written for teams that are using (or planning to use) AI coding assistants as part of their development workflow.

It is based on hard-won lessons from building the XFile platform: a multi-region financial services system with microservices, event-driven architecture, multi-currency support, and regulatory compliance requirements spanning multiple countries.

Everything in this document applies whether your "builder" is a human engineer, an AI coding assistant, or — as is increasingly common — both working together.

---

## The Construction Metaphor

We use the metaphor of building a house throughout this primer because the parallels are exact:

| Construction | Software Engineering |
|---|---|
| Surveying the lot | Discovery & requirements gathering |
| Blueprints & architecture plans | System architecture diagrams |
| Plumbing schematics | Data models & ERD diagrams |
| Electrical schematics | API contracts & integration specs |
| Building codes & permits | Compliance, security standards, ADRs |
| The construction crew | Engineering team + AI assistants |
| The job site binder | CLAUDE.md, context files, documentation |
| Foundation & framing | Core services, shared libraries, infrastructure |
| Inspections | Code review, testing, QA gates |
| Certificate of occupancy | Production deployment sign-off |
| Ongoing maintenance | Operations, monitoring, tech debt management |

The critical insight: **you would never hand a crew of contractors a pile of lumber and say "build me a house."** You would never do that even with the most experienced crew. Yet teams routinely hand AI tools (and junior engineers) vague instructions and expect production-quality results.

This primer exists so you don't make that mistake.

---

## Table of Contents

1. [Part 1: The Lot — Concept & Vision](#part-1-the-lot--concept--vision)
2. [Part 2: The Blueprints — Architecture](#part-2-the-blueprints--architecture)
3. [Part 3: The Permits — Design & Standards](#part-3-the-permits--design--standards)
4. [Part 4: The Crew — People, Roles & Tools](#part-4-the-crew--people-roles--tools)
5. [Part 5: The Build — Planning & Execution](#part-5-the-build--planning--execution)
6. [Part 6: The Inspection — Quality & Verification](#part-6-the-inspection--quality--verification)
7. [Part 7: The Handoff — Deployment & Operations](#part-7-the-handoff--deployment--operations)
8. [Appendix A: The Master Checklist](#appendix-a-the-master-checklist)
9. [Appendix B: AI-Specific Guidance](#appendix-b-ai-specific-guidance)

---

## Part 1: The Lot — Concept & Vision

*Before you break ground, you survey the land.*

### 1.1 The Principle

No construction project begins with a hammer. It begins with a question: **What are we building, and why?**

When you buy a lot, you assess the terrain, the soil composition, the zoning restrictions, the neighborhood, the climate. You determine what kind of structure the lot can support — a single-story ranch or a three-story colonial. You don't make that decision based on what's trendy; you make it based on what the land, the budget, and the homeowner's needs demand.

Software is identical. Before a single line of code is written, you must define:

- **The purpose** — What problem does this platform solve? For whom?
- **The constraints** — What regulations apply? What budget exists? What timeline?
- **The scale** — Is this a cottage or a skyscraper? Regional or global?
- **The environment** — What existing systems must it integrate with? What infrastructure exists?

### 1.2 What This Looks Like in Practice

**Business Case Document** — A clear, written articulation of why this platform exists. Not a slide deck. A document that answers:
- What business problem are we solving?
- Who are the users? What are their workflows?
- What does success look like? How will we measure it?
- What are the non-negotiable requirements (compliance, uptime, data residency)?

**Constraints & Requirements Matrix** — The "zoning laws" of your project:
- Regulatory requirements by region (GDPR, PCI-DSS, local financial regulations)
- Performance requirements (latency targets, throughput, availability SLAs)
- Integration requirements (existing systems, third-party services, payment networks)
- Budget and timeline constraints

**Stakeholder Alignment** — The homeowner, the HOA, and the city inspector all need to agree before a shovel hits dirt:
- Business stakeholders sign off on scope and priorities
- Engineering leadership signs off on technical feasibility
- Security and compliance teams review regulatory requirements
- Operations teams review infrastructure and support requirements

### 1.3 The AI Parallel

An AI coding assistant given the prompt "build me a fintech platform" will produce something. It might even compile. But it will reflect no understanding of your regulatory environment, your users, your scale requirements, or your business rules. It will be a house built without surveying the lot — and it will settle, crack, and eventually need to be torn down.

**The rule:** AI tools are force multipliers. They multiply whatever you give them. Give them vague requirements, you get confidently wrong code at scale. Give them precise, well-documented requirements, you get consistent, aligned implementations at speed.

### 1.4 Lessons from the Field: How Constraints Drive Decisions

The theory above sounds straightforward. In practice, the most important outcome of surveying the lot is discovering constraints that force you *away* from conventional wisdom. The decisions that matter most are the ones where your specific context overrides generic best practices.

**Example: Why we chose shared databases over database-per-service**

Microservices orthodoxy says every service should own its database. When we surveyed our lot — a multi-tenant financial services platform handling real money — we discovered constraints that made that approach dangerous:

- **ACID transactions are non-negotiable.** A payment that debits one account and credits another cannot tolerate eventual consistency. Distributed sagas add complexity that introduces failure modes regulators won't accept.
- **Regulatory compliance demands provable data integrity.** SOX and PCI-DSS auditors don't accept "the saga will eventually reconcile." They require atomic, verifiable transactions.
- **Reconciliation requires cross-domain queries.** Our reconciliation engine needs to JOIN transactions, accounts, and bank statements in a single query. Across dozens of separate databases, this becomes a distributed data warehouse problem.
- **Operational cost scales linearly.** One database to back up, monitor, patch, and tune — or dozens. For a platform that isn't yet at the scale where independent scaling per service is required, a database per service is pure overhead.

The result: we adopted a shared database with strict logical boundaries — services communicate through APIs, never through direct table access, but the underlying PostgreSQL instance is shared. We documented this decision, the alternatives we rejected, and the conditions under which we would revisit it (ADR-001). When those conditions change — when a single service genuinely needs independent scaling or a different storage engine — we have a clear migration path.

**The point is not that shared databases are always right.** The point is that surveying our specific lot — financial compliance, ACID requirements, operational reality — led to a decision that contradicts the default recommendation. Without that survey, we would have built a database per service because "that's what microservices do," and spent months fighting distributed consistency problems that didn't need to exist.

**Example: How regulatory requirements shaped our entire data architecture**

When you build a platform that handles real money across multiple countries, the "zoning laws" aren't metaphorical — they're literal:

| Constraint | Source | Architectural Impact |
|---|---|---|
| Tamper-evident audit trails | PSD2, SOX | Dedicated audit database with SHA-256 hash chains, RSA-2048 signatures, 7-year retention (ADR-006) |
| Data residency | GDPR | Natural sharding strategy — separate database instances per region, not per service (ADR-001) |
| Provable correctness | PCI-DSS, SOX | Integration-first testing against real databases, never mocked (ADR-002) |
| Consistent authentication | PCI-DSS, PSD2 | Centralized IAM as single source of truth — no local JWT validation (ADR-007) |
| Multi-provider support | Business/regulatory | Plugin/adapter pattern with auto-discovery for banking, payment networks, KYC providers (ADR-003) |

Each of these ADRs traces directly back to a constraint identified during our "lot survey." None of them are obvious from the prompt "build a fintech platform." All of them would have been missed — or discovered painfully during production incidents — without upfront requirements work.

**Example: Scale requirements that defined our service decomposition**

We didn't start with "let's build microservices." We started with a list of capabilities the platform needed to deliver: payment processing, account management, card issuance, reconciliation, compliance screening, multi-channel notifications, audit logging, and more. We mapped those capabilities against:

- **Independent scaling needs** — Transaction processing needs to scale differently than notification delivery
- **Regulatory isolation** — Audit logging must be isolated from business logic to prevent tampering
- **Team ownership** — Each service should be ownable by a small team
- **Deployment independence** — A change to fee calculation shouldn't require redeploying the card issuance service
- **Integration boundaries** — External provider integrations (banks, card networks, KYC providers) need abstraction layers that can swap providers without touching business logic

This analysis produced a phased service roadmap — not because a specific number of services was a goal, but because the requirements demanded that many distinct boundaries. We then sequenced them deliberately: foundation services first (IAM, event bus, shared libraries), then core business services (payments, accounts, transactions), then domain-specific services (cards, reconciliation, vIBAN), and finally advanced services (fraud detection, compliance, analytics). The sequence mirrors the construction metaphor exactly — you don't install fixtures before the plumbing is roughed in.

### 1.5 Checklist: Concept & Vision

- [ ] Business case document exists and is reviewed by stakeholders
- [ ] Target users and their primary workflows are documented
- [ ] Success metrics are defined and measurable
- [ ] Regulatory and compliance requirements are identified by region
- [ ] Performance and availability requirements are specified
- [ ] Integration requirements with existing systems are documented
- [ ] Budget and timeline constraints are established and realistic
- [ ] All key stakeholders have signed off on scope
- [ ] Constraints are documented in a format accessible to the engineering team (and AI tools)

---

## Part 2: The Blueprints — Architecture

*No contractor builds without plans. No engineer should code without architecture.*

### 2.1 The Principle

Architecture is the master blueprint. When an architect designs a house, they don't just draw the pretty exterior — they produce a complete set of documents:

- **The floor plan** — Room layout, flow between spaces, how people move through the structure
- **The structural plan** — Load-bearing walls, foundation type, framing specifications
- **The plumbing schematic** — Where water enters, where it drains, how it flows between fixtures
- **The electrical schematic** — Panel location, circuit layout, outlet placement, load calculations
- **The elevation drawings** — How it looks from every angle, how it sits on the lot

Each of these serves a different trade, but they all reference the same structure. The plumber reads the plumbing schematic but needs to know where the walls are. The electrician reads the electrical schematic but needs to know where the plumbing runs so they don't drill through a pipe.

Software architecture serves the same purpose: **it is the shared understanding that allows independent teams to build components that fit together.**

### 2.2 Architecture Diagrams — The Floor Plan

The system architecture diagram is your floor plan. It shows:

- **Service boundaries** — What are the distinct components of your system? (Microservices, modules, subsystems)
- **Communication patterns** — How do services talk to each other? (Synchronous REST/gRPC, asynchronous events, message queues)
- **External integrations** — What third-party systems does your platform connect to?
- **Data flow** — How does information move through the system from input to output?
- **Infrastructure topology** — Where do services run? How are they networked?

**What good looks like:**
```
A system architecture diagram should be readable by a new engineer
(or an AI assistant) and answer the question: "What are the major
pieces of this system, and how do they relate to each other?"
```

**What bad looks like:**
```
A single box labeled "Backend" with an arrow to a box labeled "Database"
and another arrow to a box labeled "Frontend." This tells no one anything.
```

**Real-world example:** On the XFile platform, our architecture diagrams define every microservice (IAM, Transaction Processing, Card Management, Ledger, Notification, Admin), the event bus connecting them, the API gateway fronting them, and the external integrations (payment networks, banking partners, KYC providers). A new engineer — or an AI assistant — can look at this diagram and understand where a new feature belongs before writing a line of code.

### 2.3 ERD Diagrams — The Plumbing Schematic

Your Entity Relationship Diagrams are the plumbing schematic. They define:

- **Entities** — What are the core data objects in your system? (Users, Accounts, Transactions, Cards)
- **Relationships** — How do entities relate? (A User has many Accounts. An Account has many Transactions.)
- **Attributes** — What data does each entity hold?
- **Constraints** — What rules govern the data? (Unique email, non-negative balance, required fields)

Just as a plumbing schematic ensures water flows correctly and doesn't back up, your ERD ensures data flows correctly and doesn't become inconsistent.

**Why this matters for AI:** An AI coding assistant asked to "add a transaction history endpoint" without an ERD will invent its own data model. It might create a flat table when you need a normalized schema. It might miss the relationship between transactions and ledger entries. It might not know that your system uses a specific ledger model. The ERD is the document that prevents this — it is the single source of truth for data structure.

### 2.4 API Contracts — The Electrical Schematic

API contracts are the electrical schematic of your system. They define:

- **Endpoints** — What operations are available?
- **Request/Response schemas** — What data goes in, what comes out?
- **Authentication & Authorization** — Who can call what?
- **Error handling** — What happens when things go wrong?
- **Versioning** — How do contracts evolve without breaking consumers?

Just as the electrical schematic ensures the right voltage reaches the right outlet, API contracts ensure the right data reaches the right service in the right format.

**Formats that work:**
- OpenAPI/Swagger specifications for REST APIs
- Protocol Buffer definitions for gRPC services
- AsyncAPI specifications for event-driven interfaces
- GraphQL schemas for query interfaces

**The key insight:** API contracts should be written *before* implementation, not generated *after*. You don't wire a house and then draw the electrical schematic to match — you design the schematic and then wire to spec. Contract-first development prevents integration headaches and ensures services can be built in parallel.

### 2.5 ADRs — The Permit File

Architecture Decision Records (ADRs) are your permit file. Every significant decision — *why* you chose this foundation type, *why* you ran plumbing through this wall instead of that one, *why* you specified 200-amp service instead of 100-amp — is recorded, with context and rationale.

ADRs answer the most dangerous question in software engineering: **"Why is it built this way?"**

Without ADRs, engineers (and AI tools) encounter architectural decisions and either:
1. **Assume they're arbitrary** and change them, breaking things that depended on the original reasoning
2. **Assume they're sacred** and work around them, even when the original reasoning no longer applies

**ADR format:**
```markdown
# ADR-XXX: [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
What is the issue or question that prompted this decision?

## Decision
What did we decide to do?

## Consequences
What are the positive and negative outcomes of this decision?

## Alternatives Considered
What other options were evaluated, and why were they rejected?
```

**Real-world example:** On XFile, ADR-001 establishes the microservices architecture pattern. ADR-003 defines the plugin/adapter pattern for multi-institution connectors. ADR-011 documents the ledger model for different financial institution types. When an AI assistant is tasked with adding a new banking integration, these ADRs tell it *how* integrations are structured and *why* — so it builds to pattern instead of inventing a new approach.

### 2.6 Infrastructure Architecture — The Foundation

Your infrastructure architecture defines the foundation your software sits on:

- **Cloud topology** — Regions, availability zones, account structure
- **Networking** — VPCs, subnets, load balancers, DNS
- **Compute** — Container orchestration (Kubernetes), serverless functions, VM specs
- **Storage** — Databases (type, replication, backup), object storage, caching layers
- **Security** — Network segmentation, encryption (at rest, in transit), key management

Just as a house's foundation must be designed for the soil conditions and the weight it will bear, your infrastructure must be designed for the traffic patterns, data volumes, and availability requirements of your platform.

### 2.7 The AI Parallel

AI coding tools are excellent draftsmen but terrible architects. They can draw walls wherever you point — but they won't know your family needs four bedrooms, not three.

**What AI does well with architecture:**
- Generating implementation code that follows documented architectural patterns
- Creating boilerplate for new services that match existing service structures
- Writing infrastructure-as-code when the target topology is clearly specified
- Producing API implementations from OpenAPI specs

**What AI does poorly without architecture:**
- Making system boundary decisions
- Choosing communication patterns (sync vs. async)
- Designing data models that account for business rules it doesn't know about
- Understanding why the system is built the way it is

**The rule:** Architecture is a human responsibility. Implementation is where AI accelerates. If your architecture documentation is thorough, AI can build to spec at remarkable speed. If it isn't, AI will build fast in the wrong direction.

### 2.8 Lessons from the Field: Architecture That Evolves

Architecture documents are not stone tablets. They are living blueprints that evolve as you build — and the mechanism for that evolution matters as much as the documents themselves.

**Example: How duplicated logic led to a three-layer architecture**

Our payment architecture didn't start at v3.0. In v2.1, domain services like the Incoming Payment Processor handled their own orchestration — transaction lifecycle, idempotency enforcement, fee calculation, event publishing, and audit logging. It worked. Then we built the second payment type and found ourselves duplicating the same orchestration logic. Then the third. By the time we were planning the fourth, the pattern was clear: orchestration logic was scattering across domain services, creating inconsistent behavior, duplicated business rules, and diverging audit trails.

The fix was architectural, not tactical. We established a three-layer pattern:

- **Layer 1 (Domain):** Network/protocol-specific logic only — VAN lookup, webhook validation, message transformation, network selection
- **Layer 2 (Orchestration):** Transaction Processing Service handles ALL six payment types — lifecycle, idempotency, fees, events, audit. Single source of truth.
- **Layer 3 (Data):** Account Management Service provides primitive operations only — credit, debit, hold, release. No business logic.

Every payment now flows Domain → Orchestration → Data with no layer skipping and no duplicate orchestration. This wasn't the original design — it was discovered through building, documented in an ADR, and enforced going forward. The lesson: your first architecture will be wrong in places. ADRs give you the mechanism to evolve it deliberately rather than letting it drift.

**Example: ADRs that caught real violations**

ADRs aren't just documentation — they're enforceable contracts. Two examples from our build:

*The API Gateway violation (ADR-004):* Our Admin Backend Service was designed as a Backend-for-Frontend gateway — all frontend requests route through it to the underlying microservices. During a review, we discovered the Admin Portal was making direct calls to the Analytics Service, the IAM Service, and the Fee Calculation Service, bypassing the gateway entirely. Because ADR-004 existed and was explicit, we could point to the documented decision, explain why direct calls were a security and maintainability problem (CORS sprawl, no centralized auth, no request aggregation), and execute a compliance refactor. Without the ADR, those direct calls would have been "just how it works" — and every new frontend feature would have deepened the violation.

*The authentication vulnerability (ADR-007):* Before centralizing authentication in the IAM Service, individual services validated JWTs locally. The problem: local validation only checked whether the token was *signed correctly*, not whether the user had the *right role* for that endpoint. A consumer token — valid JWT, valid signature — worked on admin endpoints. This was a critical security gap discovered in testing. ADR-007 now mandates that every service validates tokens through the IAM Service, which checks both authenticity and authorization. No local JWT validation, no exceptions. The ADR documents not just the decision but the vulnerability that prompted it, so no future engineer (or AI assistant) is tempted to "optimize" by reverting to local validation.

**Example: Data architecture separated by concern, not by service**

Our three databases exist for fundamentally different reasons:

| Database | Purpose | Why It's Separate |
|---|---|---|
| Payment DB | Core payment processing — shared across services with logical table ownership. ACID transactions across domains (accounts, transactions, ledger entries) require co-location. |
| IAM DB | Identity and access management — different security posture (password hashes, sessions, API keys), different access patterns, different team ownership. |
| Audit DB | Compliance audit trail — append-only with cryptographic hash chains and digital signatures. Different retention policy (7-year regulatory requirement), different backup strategy, must be isolated from business logic to prevent tampering. High write volume that must not impact transaction processing. |

This separation is driven by security boundaries, compliance requirements, and operational concerns — not by microservices dogma. The audit database in particular illustrates the point: it stores tamper-evident log entries with cryptographic hash chains, digital signatures, and sequential numbering for tamper detection. It needs its own backup frequency, its own retention policy (hot → warm → cold storage over 7 years), and its own access controls (most services can only write via the event bus, never query directly). Co-locating this with transactional data would compromise all of those properties.

**Example: Communication patterns — most operations use both**

Early in the build, we framed service communication as a choice: synchronous REST or asynchronous events. ADR-005 documents the realization that this is a false dichotomy. Most real business operations need *both*, simultaneously:

```
Timeline for a tenant suspension:

0ms:    Admin clicks "Suspend Tenant"
100ms:  Database commit (synchronous — admin needs confirmation)
110ms:  Event published to RabbitMQ (asynchronous — downstream services react)
120ms:  HTTP 200 returned to admin ← User sees success HERE
150ms:  Audit Service receives event, writes tamper-evident log
200ms:  Notification Service sends email to tenant
300ms:  Analytics Service updates tenant metrics
```

The synchronous path gives the user immediate feedback. The asynchronous path handles everything that doesn't need to block the user — audit logging, notifications, analytics. Forcing everything into one pattern creates either a slow user experience (everything synchronous) or an unreliable one (everything asynchronous with no immediate confirmation).

This insight shaped our event naming convention (`{resource}.{action}` — e.g., `tenant.updated`, `transaction.approved`), our EventBus configuration (RabbitMQ topic exchange on `xfile.events`), and our subscription patterns (the Audit Logging Service subscribes to `*.*` — every event on the platform — while other services subscribe only to their domain).

**The compound effect: 11 ADRs and counting**

At the time of writing, XFile has 11 Architecture Decision Records spanning:

| ADR | Decision | Why It Matters |
|---|---|---|
| ADR-001 | Microservices with shared database | Defines service boundaries and data strategy |
| ADR-002 | Integration-first TDD | Defines how correctness is proved |
| ADR-003 | Plugin/adapter pattern | Defines how external integrations are built |
| ADR-004 | Backend-for-Frontend gateway | Defines how frontends access backend services |
| ADR-005 | Dual communication patterns | Defines when to use sync vs. async (answer: both) |
| ADR-006 | Event-based audit logging | Defines how compliance audit trails work |
| ADR-007 | Centralized IAM authentication | Defines how security is enforced across all services |
| ADR-008 | Three-layer payment architecture | Defines payment flow structure (Domain → Orchestration → Data) |
| ADR-009 | Unified balance architecture | Defines the account model across payment instruments |
| ADR-010 | Fee calculation architecture | Defines the single source of truth for fee computation |
| ADR-011 | Multi-model ledger support | Defines how different financial institution models coexist |

No single ADR is the architecture. Together, they form a complete picture of *how* the system is built and *why*. A new engineer or AI assistant reading all 11 understands not just the current state but the reasoning behind it — which decisions are load-bearing walls that cannot be moved, and which are interior walls that could be reconfigured if requirements change.

### 2.9 Checklist: Architecture

- [ ] System architecture diagram exists showing all services, boundaries, and communication patterns
- [ ] ERD diagrams exist for all data domains
- [ ] API contracts are defined (OpenAPI, protobuf, AsyncAPI) before implementation begins
- [ ] ADRs exist for every significant architectural decision
- [ ] Infrastructure architecture is documented (cloud topology, networking, compute, storage)
- [ ] Architecture diagrams are versioned and kept current as the system evolves
- [ ] A new engineer (or AI assistant) can read the architecture docs and understand where a new feature belongs
- [ ] Architecture docs are referenced in project context files (CLAUDE.md or equivalent)
- [ ] Trade-offs and constraints are explicitly documented, not just decisions

---

## Part 3: The Permits — Design & Standards

*Before construction begins, you establish codes and standards.*

### 3.1 The Principle

Every municipality has building codes. These aren't suggestions — they're the baseline requirements that ensure a structure is safe, sound, and won't cause problems for its neighbors. You can build above code, but you cannot build below it.

In software, your standards serve the same purpose. They define the minimum acceptable quality, the patterns everyone follows, and the rules that keep the codebase consistent, secure, and maintainable. Without standards, every engineer (and every AI assistant) makes different choices, and the result is a house where every room was built by a different contractor with different materials and different conventions. The doors don't match, the outlets are at different heights, and the plumbing uses three different pipe sizes.

### 3.2 Coding Standards & Conventions — The Building Code

Your coding standards are the building code of your project. They include:

**Language & Framework Conventions:**
- Language version and target runtime
- Framework version and approved libraries
- Project structure and file organization conventions
- Naming conventions (classes, methods, variables, files, database tables)

**Code Style:**
- Formatting rules (enforced by automated tooling, not humans)
- Import ordering and grouping
- Maximum file length and function complexity guidelines
- Comment and documentation expectations

**Patterns & Practices:**
- Approved design patterns for common problems (repository pattern, adapter pattern, etc.)
- Error handling conventions (how errors are created, propagated, and logged)
- Logging standards (what to log, at what level, in what format)
- Configuration management (how services read config, environment variable naming)

**Why this matters more with AI:** A human developer works on a codebase for months and absorbs conventions through osmosis — code review feedback, pairing sessions, reading existing code. An AI assistant starts fresh every session. Without explicit, written conventions, it will default to generic best practices that may not match your project's patterns. The result is code that "works" but looks like it was written by someone who has never seen your codebase.

**The fix:** Your conventions must be written down, explicit, and referenced in your AI context files. Not "follow best practices" but "use the repository pattern as defined in `shared/base-repository.ts`, return `Result<T, AppError>` from all service methods, and log using the structured logger from `@xfile/logging`."

### 3.3 Security Standards — The Fire Code

Security standards are your fire code. Just as fire code dictates fire-resistant materials, sprinkler placement, and egress requirements, your security standards define:

**Authentication & Authorization:**
- Authentication mechanism (OAuth 2.0, JWT, session-based)
- Authorization model (RBAC, ABAC, policy-based)
- Token management (issuance, refresh, revocation, expiry)
- Service-to-service authentication

**Data Protection:**
- Encryption at rest (what, how, key management)
- Encryption in transit (TLS versions, certificate management)
- PII handling (what constitutes PII, how it's stored, how it's accessed)
- Data retention and deletion policies

**Input Validation & Output Encoding:**
- Input validation rules (server-side, always — client-side is cosmetic)
- SQL injection prevention (parameterized queries, ORM usage)
- XSS prevention (output encoding, CSP headers)
- CSRF protection

**Audit & Compliance:**
- What actions are audit-logged
- Audit log format and retention
- Compliance-specific requirements (PCI-DSS, SOC 2, GDPR)

**Why this matters critically with AI:** AI coding assistants will write functional code that is insecure by default unless explicitly instructed otherwise. They will:
- Concatenate user input into SQL queries if you don't specify parameterized queries
- Skip authorization checks if you don't specify the authorization model
- Log sensitive data if you don't specify what constitutes PII
- Use weak cryptographic defaults if you don't specify your encryption standards

**The rule:** Security is never implicit. Every security requirement must be explicit, documented, and enforced through automated tooling (linters, SAST scanners, pre-commit hooks). AI doesn't yet inherently know your security posture — it must be told.

### 3.4 Testing Strategy — The Inspection Schedule

Your testing strategy is your inspection schedule. Just as a house undergoes inspections at each phase of construction (foundation, framing, rough-in, final), your software must be tested at each level:

**Unit Tests — Individual Component Inspection:**
- Every public method on every service class
- Business logic validation
- Edge cases and error conditions
- Mocking strategy for external dependencies

**Integration Tests — System Assembly Inspection:**
- Service-to-database interactions
- Service-to-service communication
- Message queue publishing and consumption
- External API integration (with contract tests)

**End-to-End Tests — Final Walkthrough:**
- Critical user workflows from start to finish
- Cross-service transaction flows
- Authentication and authorization flows
- Failure and recovery scenarios

**Performance Tests — Load-Bearing Capacity:**
- Throughput under expected load
- Latency at P50, P95, P99
- Behavior under stress (2x, 5x, 10x expected load)
- Resource consumption (CPU, memory, connections)

**Test Standards:**
- Minimum coverage thresholds (not as a vanity metric, but as a safety net)
- Test naming conventions
- Test data management (fixtures, factories, seeds)
- CI pipeline integration (when tests run, what blocks a merge)

**The AI advantage with testing:** This is one area where AI genuinely excels. Given clear test standards and examples of existing tests, AI can generate comprehensive test suites rapidly. The key is providing it with:
- Your testing framework and conventions
- Examples of well-written tests in your codebase
- The business rules that need to be validated
- Your mocking and fixture patterns

### 3.5 Documentation Standards — The As-Built Drawings

As-built drawings are the documentation created *after* construction that reflects what was *actually* built (which often differs from the original plans). They are critical for anyone who maintains the building in the future — you need to know where the pipes actually run before you start renovating.

Your documentation standards define:

- **What gets documented** — API endpoints, business rules, deployment procedures, runbooks
- **Where documentation lives** — In-code comments, dedicated docs directory, wiki, API specs
- **How documentation stays current** — Review cadence, ownership, staleness detection
- **Documentation-as-code** — Documentation lives in the repo, is version-controlled, and is reviewed in PRs

**The most important documentation rule:** Documentation that is wrong is worse than no documentation. A plumber who follows an incorrect as-built drawing will cut into a live wire. An engineer (or AI assistant) who follows outdated documentation will build against an interface that no longer exists.

**How to keep documentation alive:**
- Documentation updates are part of the definition of done for every feature
- CI checks can validate that API documentation matches actual implementations
- Regular documentation audits (quarterly at minimum)
- Documentation ownership is explicit (every doc has an owner)

### 3.6 Design System & UX Standards — The Elevation Drawings

Elevation drawings show what the house looks like from the outside — the curb appeal, the proportions, the materials. Your design system serves the same purpose for your software:

- **Component library** — Reusable UI components with consistent behavior
- **Design tokens** — Colors, typography, spacing, shadows
- **Interaction patterns** — How users navigate, how errors are displayed, how loading states work
- **Accessibility standards** — WCAG compliance level, keyboard navigation, screen reader support
- **Responsive behavior** — How the interface adapts across devices and screen sizes

### 3.7 Lessons from the Field: Standards That Earn Their Keep

Standards only matter if they're enforced and if they solve real problems. The standards that survived on our project are the ones that prevented actual incidents — not the ones that sounded good in a planning meeting.

**Example: The inverted test pyramid — why we never mock the database**

The traditional test pyramid says: lots of unit tests at the bottom, fewer integration tests in the middle, fewest E2E tests at the top. For a financial services platform, we inverted it (ADR-002):

```
Traditional Pyramid:        XFile Inverted Pyramid:
      /\                          ___________
     /E2E\                             E2E (5%)
    /Int  \                       ___________
   /Unit   \                       Unit (5%)
  /__________\               ___________________
                              Integration (90%)
```

**~90% integration tests** with real PostgreSQL and seed data. **~5% unit tests** for pure business logic only. **~5% E2E tests** for full workflow validation.

The rationale is specific to financial services, not a generic preference:

1. **Regulators demand provable correctness of database transactions.** SOX and PCI-DSS auditors need evidence that payments, fees, and reconciliation work correctly against real database constraints — foreign keys, check constraints, triggers, ACID properties. A mocked database proves nothing to an auditor.

2. **Schema evolution breaks mocks silently.** When a database column changes, a mock still passes — it's testing against a fantasy. A real database test fails immediately, catching the break before it reaches production.

3. **Financial calculations depend on database constraints.** A payment that should fail due to insufficient balance, a fee that depends on a foreign key relationship to a fee structure, a reconciliation that JOINs across three tables — these can only be tested meaningfully against a real database.

4. **The performance trade-off is acceptable.** Real database tests run 20-50ms each vs. <1ms for mocked unit tests. For a platform where a single calculation error could mean lost funds, we'll take the slower tests.

**What this looks like in practice:**

Every service tests against the same PostgreSQL instance with maintained seed data across four databases (payment, IAM, audit, and test-state). The seed data includes real tenants, subaccounts, transactions, and reconciliation runs — not synthetic fixtures. Environment reset takes seconds via a single script.

We use two testing approaches:

- **Approach A (95% of services):** Test against the production database with seed data. This is the default for all internal business logic — account management, transaction processing, fee calculation, audit logging.
- **Approach B (External integrations only):** Test against a dedicated test-state database for simulating external provider responses — payment network webhooks, banking API callbacks, card processor authorizations. The schema is loaded centrally from a shared location.

The rule is simple: **if it touches the database, test it with a real database. If it's pure math or string manipulation, a unit test is fine. Never mock XFile's own infrastructure — database, EventBus, or internal services.**

Coverage thresholds are enforced in CI: >90% line coverage, >85% branch/function/statement coverage. Services below threshold cannot be marked "production ready." Every service maintains a standardized coverage report using a shared template. Real numbers from the build: our most complex service (financial reconciliation) achieved 89% coverage with nearly 700 passing tests across unit, integration, and performance suites. Other core services range from 84-90%+ coverage with hundreds of tests each.

**Example: Security as a standard, not a feature — centralized IAM**

Security standards on paper are worthless if individual services implement them differently. We learned this the hard way.

Before ADR-007, each service validated JWTs locally. The code looked correct:

```javascript
// Looks secure. Is not.
const payload = jwt.verify(token, process.env.JWT_SECRET);
req.user = payload;
```

The problem: local validation checked whether the token was *signed correctly* but not whether the user had the *right role* for that endpoint. A consumer's JWT — valid signature, valid expiry — was accepted by admin endpoints. This is a critical authorization bypass that compiles, passes tests, and looks reasonable in code review.

The fix wasn't "be more careful with JWT validation." The fix was architectural: **no service validates tokens locally. Every request is validated through the IAM Service, which checks both authenticity and authorization.** This is enforced through a shared client library that every service must use, and through a startup validation pattern that tests IAM connectivity before a service accepts traffic:

```javascript
async function startServer() {
  await db.testConnection();

  try {
    iamServiceClient.initialize();
    await iamServiceClient.getServiceToken();
    console.log('[Init] IAM authenticated');
  } catch (error) {
    console.error('[Init] IAM failed - DEGRADED mode');
  }

  app.listen(PORT);
}
```

If a service can't reach IAM at startup, it reports itself as `degraded` (HTTP 503 on the health endpoint), not `healthy`. This prevents the insidious failure mode where a service appears healthy but silently skips authentication. The health check standard is explicit: 200 means all dependencies operational, 503 means something is wrong.

To make this repeatable, we maintain an IAM Integration Guide (`docs/guides/IAM_INTEGRATION_GUIDE.md`) with copy-paste templates, step-by-step setup instructions, timing diagrams, a troubleshooting section, and a DO/DON'T checklist. When a new service is created — by a human or an AI assistant — the guide gets it to compliant IAM integration in under an hour. Without it, every service would reinvent authentication slightly differently, and we'd be back to the vulnerability that started this.

**Example: Documentation drift detection as an automated standard**

Documentation that drifts from reality is worse than no documentation. We enforce documentation accuracy with two automated tools built as AI assistant slash commands:

**`/check-docs`** scans the codebase for contradictions between documentation and implementation:
- ADR vs. implementation drift (e.g., a service doing direct database INSERT into `audit_logs` when ADR-006 requires event-based writes)
- Environment variable conflicts across `.env` files
- Path references in documentation that point to files that no longer exist
- Service communication anti-patterns (direct calls that should go through the gateway per ADR-004)
- Multiple documents claiming to be the "authoritative source" for the same thing

**`/context-check`** validates code against all ADR compliance rules:
- No cross-service database access (ADR-001)
- Test coverage >90% with real PostgreSQL, no database mocking (ADR-002)
- Plugin/adapter pattern for external integrations (ADR-003)
- All frontend calls through Admin Backend, not direct to microservices (ADR-004)
- Proper use of both REST and EventBus patterns (ADR-005)
- No direct INSERT into audit_logs (ADR-006)
- JWT validation via IAM Service only (ADR-007)
- Layer 1/Layer 2 separation in payment flows (ADR-008)
- Account Management as single source of truth for balance (ADR-009)

These aren't aspirational — they run against the actual codebase and produce actionable reports. When an AI assistant generates code that violates an ADR, the context-check catches it before it's merged.

We also maintain an environment sync script that compares `.env.example` files (committed) against `.env` files (gitignored, contain real secrets) across all services. After any pull, running the script reports exactly which environment variables are missing from your local config. This prevents the "works on my machine" class of failures that plague multi-service environments — when 20+ services each need a dozen environment variables, manual tracking is guaranteed to fail.

### 3.8 Checklist: Design & Standards

- [ ] Coding standards are documented and enforceable via linting/formatting tools
- [ ] Security standards are documented covering auth, data protection, input validation, and audit
- [ ] Testing strategy is defined with clear expectations at unit, integration, and E2E levels
- [ ] Test coverage thresholds are established and enforced in CI
- [ ] Documentation standards define what, where, and how docs are maintained
- [ ] Design system / UX standards exist for frontend development
- [ ] All standards are referenced in AI context files so assistants follow them
- [ ] Automated tooling enforces standards (linters, formatters, SAST, pre-commit hooks)
- [ ] Standards are reviewed and updated at least quarterly

---

## Part 4: The Crew — People, Roles & Tools

*You don't hand a plumber electrical wire.*

### 4.1 The Principle

A construction project doesn't succeed because you hired skilled people. It succeeds because you hired the right skilled people, gave them clear roles, equipped them with proper tools, and organized them so their work fits together.

You need a general contractor who oversees the whole project. You need an architect who designs the plans. You need specialists — electricians, plumbers, framers, roofers — who execute their domains with expertise. And you need inspectors who verify the work meets code.

You would never ask the framer to run electrical. You would never ask the electrician to design the floor plan. Specialization exists because quality requires focus.

Software teams work the same way — and now those teams include AI assistants as members of the crew.

### 4.2 Team Structure & Roles

**The General Contractor — Technical/Engineering Lead:**
- Oversees the entire project, ensures work stays aligned with the architecture
- Makes day-to-day technical decisions within the architectural framework
- Reviews work from all team members (human and AI)
- Manages dependencies between workstreams

**The Architect — System/Software Architect:**
- Designs the system architecture and maintains the blueprints
- Makes structural decisions (service boundaries, communication patterns, data models)
- Reviews architectural impact of proposed features
- Keeps ADRs current

**The Specialists — Engineers:**
- Backend engineers — the plumbers and electricians, building internal systems
- Frontend engineers — the finish carpenters and painters, building what users see and touch
- Infrastructure/DevOps engineers — the foundation and site-prep crew
- Security engineers — the fire marshal, ensuring code meets safety standards
- QA engineers — the inspectors at every phase

**The New Crew Member — AI Coding Assistants:**
- Treat them as your most productive junior developer
- Fast, tireless, and eager, but they need supervision
- They will confidently do the wrong thing if instructions are ambiguous
- They don't retain context between sessions unless you provide it explicitly
- They require the same onboarding materials you'd give a new hire

### 4.3 AI as a Crew Member — The Junior Developer Model

This is the central insight of this primer, so it's worth being explicit:

**AI coding assistants behave identically to enthusiastic junior developers.**

| Junior Developer Behavior | AI Assistant Behavior |
|---|---|
| Asks clarifying questions when confused (sometimes) | Assumes and proceeds when confused (always) |
| Follows patterns they've seen in the codebase | Follows patterns in the context window |
| Writes code that works but may not fit the architecture | Writes code that compiles but may not fit the architecture |
| Needs code review before merging | Needs code review before merging |
| Benefits enormously from clear specifications | Requires clear specifications to produce good output |
| Learns your codebase over weeks of immersion | Learns your codebase in minutes — but forgets it next session |
| Can go down a rabbit hole if unsupervised | Will go down a rabbit hole if context is incomplete |

The implication is powerful: **every process you already have for onboarding and managing junior developers applies directly to managing AI assistants.** You don't need a new process — you need to apply your existing process more rigorously and more explicitly.

### 4.4 The Job Site Binder — Context Files

On every construction site, there's a binder (or increasingly, a tablet) that contains the plans, permits, code references, and special instructions for the project. Every worker checks it before starting their shift. It's the single source of truth on-site.

In AI-assisted development, this binder is your **context file** (CLAUDE.md, .cursorrules, or equivalent). It is the most important document in your AI workflow because it is the document the AI reads at the start of every session.

**What belongs in the context file:**
```
1. Project overview — What is this system? (one paragraph)
2. Architecture summary — Key services, their responsibilities, how they communicate
3. Tech stack — Languages, frameworks, versions, key libraries
4. Project structure — Directory layout, where things live
5. Coding conventions — Naming, patterns, error handling, logging
6. Testing conventions — Framework, patterns, where tests live, how to run them
7. Build & run instructions — How to build, test, and run locally
8. Key references — Links to architecture docs, ERDs, API specs, ADRs
9. Common pitfalls — Things that frequently go wrong and how to avoid them
10. Current status — What phase the project is in, what's in progress
```

**What does NOT belong in the context file:**
- Every detail of every service (it should reference detailed docs, not duplicate them)
- Outdated information (this document must be kept current)
- Opinions or philosophies without actionable guidance

**The test:** If a new engineer could read your context file and produce their first PR within a day without asking basic setup or "where does this go?" questions, your context file is good. If an AI assistant can produce code that follows your patterns without correction, your context file is excellent.

### 4.5 Tool Selection

Just as a contractor selects tools for the job — a framing nailer for framing, a finish nailer for trim, a specific saw for specific cuts — your team needs the right tools:

**Development Tools:**
- IDE / editor (standardized across the team for consistency)
- AI coding assistant(s) — selected for your language/framework and workflow
- Version control (Git) and collaboration platform (GitHub, GitLab)
- CI/CD pipeline tooling

**Architecture & Design Tools:**
- Diagramming tools (for architecture, ERD, sequence diagrams)
- API design tools (Swagger/OpenAPI editor, Postman)
- Design tools (Figma, etc. for UI/UX)

**Project Management:**
- Issue tracking (Jira, Linear, GitHub Issues)
- Documentation platform (in-repo Markdown, Confluence, Notion)
- Communication (Slack, Teams)

**Quality & Security:**
- Linters and formatters (language-specific, enforced in CI)
- SAST/DAST scanners
- Dependency vulnerability scanning
- Test coverage reporting

### 4.6 Lessons from the Field: Building the Job Site Binder

The job site binder metaphor sounds simple. In practice, building one that actually works — that an AI assistant reads and follows — required iterating through several failures.

**Example: Tiered context loading — because one size doesn't fit all**

Our first attempt at a context file was a single, massive document. The problem: for a quick bug fix, loading 11 ADRs and 4 architecture documents was overkill and consumed context window space that could be used for the actual task. For complex architectural work, a summary wasn't enough — the AI needed the full ADR text to understand constraints and trade-offs.

The solution was a tiered approach:

- **Tier 1 (Automatic — every session):** `.claude/instructions.md` loads automatically. It's ~700 lines containing core ADR summaries, critical rules with code examples, anti-patterns, environment variable standards, path resolution patterns, port assignments, and a documentation hierarchy. An AI assistant reading this file can handle 80% of daily development tasks correctly.

- **Tier 2 (On-demand — complex work):** The `/onboard` slash command loads the full text of all 11 ADRs plus the core architecture documentation (Architecture Overview, Microservices Structure, Payment Network Architecture). This is used when starting work on a new feature that spans multiple services, making architectural decisions, or refactoring existing code.

- **Tier 3 (Targeted — compliance):** The `/context-check` and `/check-docs` commands validate work against standards without loading everything. They're used during and after development, not before.

The key insight: **the context file isn't a single document — it's a system.** The automatic tier handles daily work. The on-demand tier provides deep context when needed. The compliance tier catches what the other two missed.

**Example: The agent context gap — a real lesson learned**

AI coding assistants can spawn sub-agents for parallel work. We discovered the hard way that these sub-agents don't inherit the parent's context. An agent launched to "write integration tests for the account service" will write tests based on generic software testing knowledge — which means mocking the database, writing unit tests instead of integration tests, guessing at field names in camelCase instead of checking the actual PostgreSQL schema for snake_case columns, and inventing test data instead of using seed data. Every one of these violates ADR-002.

Our instructions.md now includes explicit guidance for this scenario: when launching a sub-agent for testing work, the prompt must include specific file paths to read first (ADR-002, the testing strategy guide, the instructions.md itself), non-negotiable requirements (90% integration tests, real PostgreSQL, no database mocking), and a verification checklist for after the agent completes. We also include explicit anti-patterns:

```
NEVER do these in agent-generated tests:
- jest.mock('../../config/database')
- Field names like reversedAmount (should be reversed_amount)
- Enum values like 'enterprise' (check schema for valid values)
- Invented UUIDs like '00000000-0000-0000-0000-000000000000'
```

This isn't theoretical guidance — it's the result of receiving agent output that did all of these things and having to reject and redo it. The explicit anti-pattern list was built from actual failures.

**Example: Schema-first testing rules — telling AI what to check before writing code**

One of the most persistent problems with AI-generated code is inventing data structures instead of checking what actually exists. For a platform with dozens of tables across multiple databases, this leads to tests that reference columns that don't exist, use enum values that aren't valid, and create test data with field names that don't match the schema.

Our instructions.md includes schema-first testing rules that require — before writing any test — checking the actual database:

```bash
# Check table structure for field names
psql -d your_database -c "\d table_name"

# Check enum values
psql -d your_database -c "\dT+ enum_name"

# Get actual seed data IDs (not invented ones)
psql -d your_database -c "SELECT id FROM tenants LIMIT 5;"
```

This is explicitly called out because AI assistants will confidently generate plausible-looking field names (`reversedAmount`, `processingFee`, `pendingBalance`) that don't match the actual schema (`reversed_amount`, `processing_fee`, `pending_balance`). The tests compile, they look reasonable in review, and they fail at runtime with confusing errors. The fix isn't "be more careful" — it's "always check the schema first, every time, no exceptions."

**Example: The documentation hierarchy — resolving conflicting sources**

With 11 ADRs, multiple architecture documents, service-specific READMEs, guides, and tutorials, conflicting information is inevitable. Rather than hoping it doesn't happen, we established an explicit precedence order:

1. **Architecture Decision Records (ADRs)** — Authoritative for all architectural decisions, no exceptions
2. **Architecture Documentation** — Authoritative for system design
3. **Service-Specific Documentation** — Authoritative for individual service implementation
4. **Standards Documentation** — Authoritative for coding standards and patterns
5. **Guides and Tutorials** — Informational only, not authoritative

When an AI assistant encounters a conflict — a guide says to do something one way, but an ADR says otherwise — the precedence order resolves it without requiring a human to arbitrate. This is referenced in the instructions.md so it's available in every session.

### 4.7 Checklist: People, Roles & Tools

- [ ] Team roles are defined with clear responsibilities
- [ ] AI assistants are treated as team members with defined scope and review requirements
- [ ] A context file (CLAUDE.md or equivalent) exists and is kept current
- [ ] The context file references all key architecture docs, standards, and conventions
- [ ] A new team member (human or AI) can get productive within one day using onboarding docs
- [ ] Tool selection is deliberate and standardized across the team
- [ ] Code review is required for all contributions — human and AI alike
- [ ] Human-in-the-loop checkpoints are defined for architectural decisions, security-sensitive code, and production deployments

---

## Part 5: The Build — Planning & Execution

*Now you build — in phases, with inspections at every stage.*

### 5.1 The Principle

A house isn't built all at once. Construction follows a strict sequence dictated by physical dependencies:

1. **Site preparation and foundation** — You can't frame before the foundation cures
2. **Framing** — You can't run plumbing or electrical without walls
3. **Rough-in (plumbing, electrical, HVAC)** — You can't install drywall until inspectors approve the rough-in
4. **Insulation and drywall** — You can't paint bare framing
5. **Finish work (trim, fixtures, paint)** — You can't install countertops without cabinets
6. **Final inspection and punch list** — You don't hand over the keys until everything is right

Violating this sequence creates rework. Hanging drywall before the plumbing inspection means tearing it out if the inspector finds a problem. Pouring the foundation before the site survey means discovering the hard way that the lot slopes wrong.

Software has the same dependencies. Building UI before the API exists means building against assumptions. Building services before the data model is defined means refactoring when the model changes. Deploying before testing means fixing bugs in production.

### 5.2 Phase Planning — The Construction Schedule

Enterprise software should be built in phases, with each phase delivering a complete, testable increment. This is not waterfall — it is sequencing work based on real dependencies.

**Phase structure:**
```
Phase 1: Foundation
- Infrastructure setup (cloud accounts, networking, CI/CD pipeline)
- Core shared libraries (logging, configuration, error handling)
- Database schemas for core entities
- Authentication/authorization service
- Development environment and local setup tooling

Phase 2: Framing
- Core business services (the primary domain services)
- API gateway and routing
- Inter-service communication (event bus, message queues)
- Initial test suites at all levels

Phase 3: Rough-In
- Integration with external services (payment networks, banking partners, etc.)
- Business logic implementation (the "plumbing and wiring")
- Admin interfaces
- Monitoring and observability setup

Phase 4: Finish Work
- Frontend implementation
- Documentation completion
- Performance optimization
- Security hardening and audit

Phase 5: Inspection & Punch List
- End-to-end testing across all workflows
- Load and stress testing
- Security penetration testing
- Documentation review and final sign-off
```

Each phase has **entry criteria** (what must be complete before it begins) and **exit criteria** (what must be verified before moving to the next phase).

### 5.3 Task Decomposition — Work Orders

A general contractor doesn't tell the electrician "wire the house." They issue specific work orders: "Run a 20-amp circuit from the panel to the kitchen island with four outlets spaced 24 inches apart, GFCI protected."

The same precision is required when assigning tasks to engineers and AI assistants. The quality of the output is directly proportional to the quality of the specification.

**Bad work order (for human or AI):**
```
"Add transaction processing to the platform."
```

**Good work order:**
```
"Create the TransactionService in the transaction-processing microservice.

It should:
- Accept a CreateTransactionRequest (see API spec in api-specs/transaction.yaml)
- Validate the request against business rules:
  - Source account must exist and be active
  - Source account must have sufficient balance (check via LedgerService)
  - Transaction amount must be positive and within daily limits
  - Currency must be supported for the account type
- Create a transaction record in the transactions table (see ERD)
- Publish a TransactionCreated event to the event bus (see event schema)
- Return a TransactionResponse with the transaction ID and status

Follow the existing service pattern in AccountService for structure.
Use the existing repository pattern for database access.
Use the existing event publisher for event publishing.

Write unit tests covering:
- Successful transaction creation
- Insufficient balance rejection
- Invalid account rejection
- Daily limit exceeded rejection

Reference: Relevant ADRs (ledger model, fee structure), API spec"
```

**The difference:** The bad work order requires the implementer to make dozens of decisions. A senior engineer might make them well. A junior engineer or AI assistant will make them — but not necessarily in alignment with your architecture, your patterns, or your business rules.

### 5.4 Status Tracking — The Job Board

Every construction site has a visual status board. What's in progress, what's blocked, what's complete, what failed inspection.

Your project needs the same:

- **Current phase** — Where are we in the overall build?
- **Active workstreams** — What's being built right now, and by whom?
- **Blockers** — What's stuck and why?
- **Completed work** — What's been built, tested, and approved?
- **Upcoming work** — What's next, and what are its dependencies?

Status tracking should be:
- **Updated daily** — Stale status boards are useless
- **Visible to everyone** — Including AI assistants (reference current status in context files)
- **Honest** — A task that's "90% done" but hasn't been tested is not 90% done
- **Action-oriented** — Every blocker should have an owner and a resolution plan

### 5.5 PR & Code Review — The Inspection

Code review is your inspection process. Just as an inspector verifies that work meets code before it's covered by drywall, a code reviewer verifies that work meets standards before it's merged into the main branch.

**Code review must apply to ALL code, regardless of who (or what) wrote it.**

AI-generated code requires *more* scrutiny, not less, because:
- It may follow the letter of the specification but miss the spirit
- It may introduce subtle issues that compile and pass tests but violate architectural patterns
- It may duplicate logic that already exists elsewhere in the codebase
- It may use deprecated patterns or libraries
- It tends to over-engineer or add unnecessary abstractions

**Code review checklist:**
- [ ] Does this change align with the architecture?
- [ ] Does it follow project coding conventions?
- [ ] Is the business logic correct and complete?
- [ ] Are there sufficient tests?
- [ ] Are there security concerns?
- [ ] Does it introduce unnecessary complexity?
- [ ] Is the error handling appropriate?
- [ ] Will this be maintainable by someone who didn't write it?

### 5.6 CI/CD Pipeline — The Assembly Line

Your CI/CD pipeline is the quality assurance assembly line that every change must pass through:

```
Code Commit
  |
  v
Automated Formatting Check ---- FAIL --> Block merge, fix formatting
  |
  PASS
  v
Linting & Static Analysis ----- FAIL --> Block merge, fix violations
  |
  PASS
  v
Unit Tests --------------------- FAIL --> Block merge, fix tests
  |
  PASS
  v
Integration Tests -------------- FAIL --> Block merge, fix tests
  |
  PASS
  v
Security Scan (SAST) ----------- FAIL --> Block merge, fix vulnerabilities
  |
  PASS
  v
Build Artifact
  |
  v
Deploy to Staging
  |
  v
E2E Tests ---------------------- FAIL --> Block promotion, investigate
  |
  PASS
  v
Manual Review Gate
  |
  APPROVED
  v
Deploy to Production
```

**The rule:** No code reaches production without passing every gate. No exceptions. This is the equivalent of never issuing a certificate of occupancy without a final inspection.

### 5.7 Lessons from the Field: How the Build Actually Went

Plans look clean on paper. The build reveals the real dependencies — the ones you anticipated and the ones you didn't.

**Example: Deliberate sequencing — learn the pattern before building the hard thing**

Our phase plan wasn't arbitrary. We started with the simplest services and worked toward the most complex, for a specific reason: each phase taught us something we needed for the next.

```
Phase 1 (Weeks 1-4): Low-Risk Services — Learn the Pattern
  ✅ Fee Calculation Service    → Stateless service extraction
  ✅ Notification Service       → Async event-driven patterns
  ✅ Audit Logging Service      → Database-backed event consumer
  ✅ Analytics Service          → Read-only event subscriber
  ✅ Event-Driven Architecture  → Message broker + shared event bus library

Phase 2 (Weeks 5-10): IAM & Applications — Build Auth Before Everything Needs It
  ✅ IAM Service                → Complex multi-tenant authentication
  ✅ Admin Portal               → Frontend with full auth integration
  ✅ Consumer Wallet            → Consumer-facing application
  ✅ Merchant POS               → Merchant-facing application
  ✅ CI/CD Pipeline             → GitHub Actions with automated testing

Phase 3 (Weeks 11-13): High-Risk Core — Built 8 Weeks Ahead of Schedule
  ✅ Transaction Processing     → Payment orchestration, cross-tenant support
  ✅ Account Management         → Balance management, account lifecycle

Phase 4 (Ongoing): Domain Services — Built on Solid Foundation
  ✅ Banking Integration        → External provider abstraction (ADR-003)
  ✅ Reconciliation Engine      → Three-way financial matching
  ✅ Virtual Account Management → Virtual account allocation
  ✅ Card Platform (5 services) → Issuing, transactions, disputes, ATM, program mgmt
```

The sequencing logic: Fee Calculation is stateless — no database, no auth, no events. It's the simplest possible microservice, and building it first taught us the project structure, the build system, and the deployment pipeline. Notification Service added event consumption. Audit Logging added database persistence. By the time we reached IAM (the most complex foundational service), we had three successful services teaching us the patterns.

Building IAM before the core business services (Transaction Processing, Account Management) was critical. Without IAM in place, every service would have used ad-hoc authentication — and we'd have spent weeks retrofitting centralized auth later. Because IAM was built in Phase 2, every service built after it used centralized authentication from day one.

The result: high-risk core services were completed 8 weeks ahead of schedule. Not because we rushed — because the foundation was solid. When Transaction Processing Service was being built, the event bus was battle-tested, IAM integration was a solved pattern, and the testing infrastructure was mature.

**Example: Startup order — dependency chains are real at runtime too**

Dependencies don't just matter during the build phase — they matter every time you start the platform. Our startup script launches 20+ services in a tmux session, and the order is deliberate:

```bash
# 1. Audit Logging FIRST — must be running to capture all events
tmux new-session -d -s xfile -n "audit" "cd audit-logging-service && npm start"

# 2. IAM Service SECOND — authentication provider for everything
tmux new-window -t xfile -n "iam" "cd iam-service && npm start"

# 3. Then core services, then domain services, then frontends...
```

Audit Logging starts first because it subscribes to every event on the platform (`*.*`). If it starts after other services, events published during startup are lost — and for a compliance-critical audit trail, lost events are a regulatory violation. IAM starts second because every other service validates tokens through it at startup (the fail-fast pattern from ADR-007). If IAM isn't ready when services try to authenticate, they enter degraded mode unnecessarily.

This startup order was learned, not designed. The first time we started all services simultaneously, we discovered race conditions: services trying to authenticate before IAM was ready, events published before the audit subscriber was listening, health checks failing because dependencies weren't available yet. The script now builds in wait times and health checks between phases.

**Example: Operational tooling — the scripts that make the build repeatable**

A construction site doesn't function on blueprints alone — it needs tools, jigs, and fixtures that make work repeatable. For a platform with 20+ microservices, operational scripts are that tooling. We maintain a library of shell scripts that handle:

- **Environment setup:** Our reset script drops and recreates all databases, loads schemas, seeds data from our banking partner's sandbox, creates test users, and verifies the installation. It handles edge cases like circular foreign keys in the transactions table (where `parent_transaction_id` references the same table) by disabling triggers during seed data loading:

  ```bash
  ALTER TABLE transactions DISABLE TRIGGER ALL;
  -- Load seed data with parent references
  ALTER TABLE transactions ENABLE TRIGGER ALL;
  ```

- **Service orchestration:** The startup script launches all services with correct environment variables, IAM credentials, database connections, and EventBus configuration per service. Each service gets its own tmux window for isolated log viewing.

- **Environment sync:** An env-sync script compares `.env.example` (committed) against `.env` (gitignored) across all services and reports exactly which variables are missing after a pull. For a platform where each service needs a dozen+ environment variables, this prevents configuration-related startup failures.

- **Health verification:** A health-check script pings all service health endpoints, reports status, and can auto-restart failed services.

None of these scripts are glamorous. All of them prevent the kind of "it works on my machine" failures that slow teams down. When a new developer (or an AI assistant operating in a fresh environment) can run two scripts and have a fully operational multi-service platform with seeded data, the barrier to productive work drops dramatically.

### 5.8 Checklist: Planning & Execution

- [ ] Project is divided into phases with clear dependencies and milestones
- [ ] Each phase has defined entry and exit criteria
- [ ] Tasks are decomposed into specific, well-defined work orders
- [ ] Work orders include acceptance criteria, references to architecture docs, and pattern examples
- [ ] Status tracking is maintained and updated daily
- [ ] Code review is required for all merges, with a defined checklist
- [ ] CI/CD pipeline enforces all quality gates automatically
- [ ] No code reaches production without passing all automated checks and human review
- [ ] AI-generated code receives the same (or higher) review scrutiny as human-written code

---

## Part 6: The Inspection — Quality & Verification

*No house passes without final inspection.*

### 6.1 The Principle

Throughout construction, inspections happen at critical junctures — not at the end. The foundation is inspected before framing begins. The rough-in plumbing and electrical are inspected before drywall covers them. The final inspection verifies everything before the homeowner gets the keys.

The cost of fixing a problem increases exponentially the later it is discovered:
- Foundation crack found during the pour: **minor repair**
- Foundation crack found after framing: **major structural repair**
- Foundation crack found after the family moves in: **catastrophic and expensive**

Software quality follows the exact same curve. A bug caught in a unit test costs minutes. A bug caught in staging costs hours. A bug caught in production costs days, reputation, and potentially money.

### 6.2 Testing at Every Level

**Unit Tests — Component Inspection (Foundation, Framing)**
```
Every load-bearing beam is checked individually.
Every circuit is tested before it's connected to the panel.

- Test individual methods and functions in isolation
- Test business logic with all edge cases
- Test error handling paths
- Fast, numerous, and cheap to run
- Target: Every public method on every service
```

**Integration Tests — System Assembly Inspection (Rough-In)**
```
The plumbing is pressurized to check for leaks at every joint.
The electrical panel is energized to verify all circuits.

- Test service-to-database interactions with real databases
- Test service-to-service communication
- Test message publishing and consumption
- Test external service integrations (with contract tests)
- Slower, fewer, but critical for catching wiring errors
```

**End-to-End Tests — Final Walkthrough**
```
Turn on every faucet. Flip every switch. Open every door.
Walk the entire house as if you're the homeowner moving in.

- Test critical user workflows from login to completion
- Test cross-service transaction flows
- Test failure scenarios and recovery
- Slowest, fewest, but validate the complete system
```

**Performance Tests — Load-Bearing Capacity**
```
Will the deck support a full party? Will the foundation hold
through an earthquake? Will the roof withstand a hurricane?

- Load testing at expected peak traffic
- Stress testing at multiples of expected traffic
- Endurance testing over extended periods
- Identify bottlenecks before users do
```

### 6.3 Security Verification

**Static Analysis — Blueprint Review**
- Automated scanning for known vulnerability patterns (SAST)
- Dependency vulnerability scanning
- Secret detection (no credentials in code)
- License compliance checking

**Dynamic Analysis — Penetration Testing**
- Automated security testing against running application (DAST)
- Manual penetration testing for critical systems
- Authentication and authorization boundary testing
- Input fuzzing

**Compliance Verification**
- Regulatory requirement traceability (requirement -> implementation -> test)
- Audit log completeness verification
- Data handling verification (encryption, retention, access controls)
- Third-party security assessment (SOC 2, PCI-DSS audits)

### 6.4 Documentation Verification

Before handing over the keys, verify that your as-built drawings match reality:

- [ ] API documentation matches actual API behavior
- [ ] Architecture diagrams reflect the current system state
- [ ] Runbooks are accurate and have been tested
- [ ] Deployment procedures are documented and reproducible
- [ ] All ADRs are current (none reference deprecated decisions without noting it)

### 6.5 Lessons from the Field: Inspections That Actually Catch Things

Inspection processes only work if they check the right things and are easy enough to run that people actually use them. We built ours around a simple principle: if a violation has happened before, there should be an automated check that prevents it from happening again.

**Example: The `/context-check` command as an automated building inspector**

After the ADR-004 violation (Admin Portal calling microservices directly) and the ADR-007 vulnerability (local JWT validation bypassing role checks), we realized that code review alone wasn't catching architectural violations reliably. A human reviewer might miss that a new endpoint validates tokens locally instead of through IAM — the code looks correct, it works, and the reviewer has 15 other files to look at.

The `/context-check` slash command was built to be the automated inspector. It runs against the codebase and checks every ADR compliance rule:

- **ADR-001:** Are any services accessing another service's database tables directly?
- **ADR-002:** Is test coverage >90%? Are integration tests using real PostgreSQL? Is there any database mocking?
- **ADR-003:** Do external integrations use the plugin/adapter pattern with connector manifests?
- **ADR-004:** Are all frontend API calls going through the Admin Backend gateway, not direct to microservices?
- **ADR-005:** Are operations using both synchronous and asynchronous patterns where appropriate?
- **ADR-006:** Is anyone doing direct INSERT into `audit_logs` instead of publishing events?
- **ADR-007:** Is JWT validation happening through IAM Service, or is anyone validating locally?
- **ADR-008:** Is there clean separation between Layer 1 (domain) and Layer 2 (orchestration)?
- **ADR-009:** Is Account Management Service the sole owner of balance state?

The output is actionable: compliant, warning, or violation — with specific file locations and descriptions of what's wrong. This runs before code review, catching structural violations that are hard for humans to spot in large PRs.

**Example: Real coverage numbers — what inspection results actually look like**

Coverage thresholds are meaningless without enforcement and visibility. Every service maintains a standardized `TEST_COVERAGE_REPORT.md` using a shared template that reports line, branch, function, and statement coverage. Here are real numbers from the build:

| Service Category | Test Count Range | Coverage Range | Key Testing Focus |
|---|---|---|---|
| Financial Reconciliation | 600+ | 85-90% | Three-way matching, discrepancy detection, resolution workflows |
| Payment Orchestration | 250-300 | 85-90% | Payment lifecycle, cross-tenant settlement, idempotency |
| External Integration | 250-300 | 85-90% | Provider abstraction, webhook processing, account sync |
| Compliance & Audit | 250-300 | 85-90% | Event subscription, integrity verification, tamper detection |
| Authentication | 200-300 | 80-85% | Authentication flows, RBAC, service-to-service auth |
| Virtual Accounts | 150-200 | 85-90% | Account lifecycle, payment attribution |

These numbers are published, reviewed, and gated in CI. A service can't ship below the coverage threshold. The numbers aren't vanity metrics — they represent real confidence that financial calculations, compliance features, and security boundaries work correctly. When the reconciliation service has 600+ tests covering three-way matching between internal records, bank statements, and payment networks, that's provable correctness that an auditor can point to.

**Example: The `/check-docs` command — catching documentation drift before it misleads**

Documentation drift is a special category of quality failure. Unlike a bug that crashes at runtime, stale documentation fails silently — someone (human or AI) reads it, follows it, and builds something wrong. The failure only surfaces later when the implementation doesn't match the rest of the system.

Our `/check-docs` command scans for specific drift patterns:

- **ADR vs. implementation:** Does the code actually follow what the ADR says? If ADR-006 says "never insert directly into audit_logs," are there any `INSERT INTO audit_logs` statements in the codebase?
- **Path references:** Do file paths mentioned in documentation actually exist? After a refactor, docs often reference moved or deleted files.
- **Environment conflicts:** Do `.env` files across services have conflicting values for the same variable?
- **Competing authority:** Are multiple documents claiming to be the "authoritative source" for the same information? (This is a real problem when architecture docs and service READMEs evolve independently.)

This isn't a substitute for human documentation review — it's the automated first pass that catches the mechanical failures so human reviewers can focus on whether the documentation is *complete and clear*, not whether it's *technically accurate*.

**Example: Audit log integrity verification — inspecting the inspector**

For a platform with regulatory compliance requirements, the audit trail itself needs inspection. Our audit database uses SHA-256 hash chains (each log entry includes the hash of the previous entry), RSA-2048 digital signatures, and sequential numbering. But these integrity features are useless if they aren't verified.

The Audit Logging Service runs periodic integrity verification (hourly) that checks:
- Hash chain continuity — does each entry's `previous_log_hash` match the actual hash of the preceding entry?
- Signature validity — can each entry's digital signature be verified against the signing key?
- Sequence completeness — are there any gaps in the sequential numbering?
- Anomaly detection — unusual patterns like failed login spikes, geographic anomalies, or activity outside business hours

This is the meta-inspection: inspecting the inspection system itself. If someone (or something) tampers with the audit trail, the hash chain breaks and the integrity check fails. This is the regulatory equivalent of a sealed, tamper-evident inspection record.

### 6.6 Checklist: Quality & Verification

- [ ] Unit tests exist for all business logic with adequate coverage
- [ ] Integration tests verify all service boundaries and data flows
- [ ] E2E tests cover all critical user workflows
- [ ] Performance tests validate behavior under expected and peak load
- [ ] Security scanning (SAST, DAST) is integrated into CI/CD
- [ ] Dependency vulnerabilities are monitored and remediated
- [ ] No secrets exist in the codebase
- [ ] Compliance requirements are traceable to implementations and tests
- [ ] Documentation is verified against actual system behavior
- [ ] All quality gates pass before production deployment

---

## Part 7: The Handoff — Deployment & Operations

*Certificate of occupancy. Someone moves in.*

### 7.1 The Principle

The house is built. Inspections are passed. But the job isn't done when the last nail is driven — it's done when the homeowner has the keys, the utilities are connected, the systems are explained, and a maintenance plan is in place.

No builder hands over a house and disappears. They provide:
- A walkthrough of all systems (HVAC, electrical panel, water shutoff, security system)
- Warranty documentation
- Emergency contacts (who to call when the furnace breaks at 2 AM)
- A maintenance schedule (change HVAC filters quarterly, flush the water heater annually)

Your production deployment is the same handoff.

### 7.2 Deployment Strategy — Moving Day

**Blue/Green Deployment — Two Houses:**
- Build the new version alongside the old one
- Switch traffic when the new version is verified
- Keep the old version running as a rollback option
- Zero downtime, instant rollback capability

**Canary Deployment — One Room at a Time:**
- Deploy the new version to a small percentage of traffic
- Monitor for errors, latency, and business metric anomalies
- Gradually increase traffic if everything looks good
- Roll back immediately if problems appear

**Regional Rollout — Neighborhood by Neighborhood:**
- Deploy to one region first (preferably lowest-traffic)
- Verify behavior in production conditions
- Roll out to additional regions progressively
- Critical for globally deployed platforms with regulatory differences

### 7.3 Monitoring & Observability — Smoke Detectors and Security Cameras

You wouldn't move into a house without smoke detectors, carbon monoxide detectors, and a security system. Your production system needs equivalent monitoring:

**Application Monitoring:**
- Request rate, error rate, latency (RED metrics)
- Business metric dashboards (transactions processed, users active)
- Error tracking with full context (stack traces, request data, user context)

**Infrastructure Monitoring:**
- CPU, memory, disk, network utilization
- Container health and restart counts
- Database connection pool utilization, query performance
- Message queue depth and consumer lag

**Alerting:**
- Critical alerts (pages) — Service down, data loss risk, security breach
- Warning alerts (notifications) — Elevated error rates, approaching capacity
- Informational alerts — Deployment completed, scaling events
- Alert fatigue management — Every alert must be actionable

**Distributed Tracing:**
- Trace requests across service boundaries
- Identify latency bottlenecks
- Debug failures in multi-service workflows

### 7.4 Runbooks & Incident Response — When the Pipes Burst at 2 AM

**Runbooks** are the emergency procedures for your production system:
- Step-by-step instructions for common operational tasks
- Troubleshooting guides for known failure modes
- Escalation procedures (who to call, when, and how)
- Recovery procedures (how to restore from backup, how to roll back a deployment)

**Incident Response Process:**
```
1. DETECT  — Monitoring alerts fire
2. TRIAGE  — Assess severity and impact
3. RESPOND — Execute runbook, mitigate impact
4. RESOLVE — Fix the root cause
5. REVIEW  — Post-incident review (blameless)
6. IMPROVE — Update runbooks, add monitoring, fix systemic issues
```

### 7.5 Maintenance Plan — Every House Needs Upkeep

Production software requires ongoing maintenance:

- **Dependency Updates** — Keep libraries and frameworks current (security patches, bug fixes)
- **Technical Debt Management** — Schedule regular debt repayment, don't let it accumulate
- **Capacity Planning** — Review growth trends, scale proactively not reactively
- **Backup Verification** — Regularly test that backups can actually be restored
- **Disaster Recovery Testing** — Periodically simulate failures and verify recovery procedures
- **Documentation Maintenance** — Keep docs current as the system evolves

### 7.6 Lessons from the Field: Monitoring a 25-Service Platform

Deploying a single application is straightforward. Deploying and monitoring 25 interconnected services — where a failure in one can cascade to others — requires infrastructure that's built alongside the services, not bolted on afterward.

**Example: Health checks that tell the truth**

The most dangerous service state isn't "down" — it's "up but broken." A service that returns HTTP 200 on its health endpoint while silently failing to authenticate with IAM, connect to the database, or reach the message broker will pass basic monitoring checks while producing incorrect results.

Our health check standard (established in ADR-007 and applied across all services) uses three-tier health reporting:

```javascript
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    checks: {
      database: 'unknown',
      iam: 'unknown',
      eventBus: 'unknown'
    }
  };

  // Check each dependency
  const dbHealth = await testDatabaseConnection();
  const iamHealth = await iamServiceClient.healthCheck();

  health.checks.database = dbHealth ? 'connected' : 'failed';
  health.checks.iam = iamHealth.healthy ? 'authenticated' : 'failed';

  // Determine overall status
  if (!dbHealth) health.status = 'unhealthy';       // Critical failure
  else if (!iamHealth.healthy) health.status = 'degraded';  // Partial failure

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});
```

- **200 (healthy):** All dependencies operational. Safe to route traffic.
- **503 (degraded):** Service running but a dependency is down. May produce incomplete results.
- **503 (unhealthy):** Critical failure. Do not route traffic.

The startup script runs a health check sweep after launch and reports the status of all services — backend, frontend, and infrastructure — in a single summary. A recent startup reported the vast majority of backend services healthy, all frontends healthy, and all infrastructure components operational. The services that weren't fully healthy were stub implementations that don't yet have full health endpoints — and the report told us that explicitly, rather than hiding it behind a green status.

**Example: Monitoring and observability built in, not bolted on**

Our Monitoring & Observability Service collects metrics from all services via Prometheus and exposes them through Grafana dashboards. This was built as one of the core services, not as a Phase 4 afterthought, because we needed visibility into service behavior from the first multi-service integration test.

The monitoring stack includes five purpose-built dashboards:

- **Service Overview** — Health status, uptime, and availability across all services with tier classification (Tier 1 critical, Tier 2 important, Tier 3 standard)
- **Request Metrics** — HTTP request rates, latency distributions, and error rates per service
- **Error Tracking** — Error rates, failure patterns, and service-level error analysis
- **Business Metrics** — Payment volumes, transaction success rates, revenue tracking
- **Infrastructure** — Database connections, query performance, message queue depth, system resources

Every service exposes a `/metrics` endpoint in Prometheus format. This isn't optional — it's part of the service template. When a new service is created, metrics exposition is included from the start, which means the monitoring stack picks it up automatically without configuration changes.

**Example: Natural sharding as a deployment strategy**

For a multi-region financial platform, deployment and data residency are inseparable. Our architecture (ADR-001) uses natural sharding by region and institution — each region deploys its own database instances, its own service stack, and its own Admin Backend. This isn't just a scaling strategy; it's a regulatory compliance strategy:

- **EU data stays in EU** — GDPR data residency requirements satisfied by architecture, not by policy
- **Regional deployments are independent** — A UK deployment can be updated without touching the EU deployment
- **Per-institution isolation** — Different banking partners per region are configured per deployment, not globally
- **Regulatory alignment** — Country-specific compliance rules (PSD2 in EU, state regulations in US) are enforced at the deployment level

This means "deploying to production" isn't one event — it's a series of regional deployments, each verified independently. The Admin Backend Service aggregates data from regional deployments for super-admin visibility, but each region operates autonomously. A failure in one region doesn't cascade to others.

### 7.7 Checklist: Deployment & Operations

- [ ] Deployment strategy is defined (blue/green, canary, regional rollout)
- [ ] Rollback procedures are documented and tested
- [ ] Application monitoring covers RED metrics and business KPIs
- [ ] Infrastructure monitoring covers all critical resources
- [ ] Alerting is configured with appropriate severity levels and escalation paths
- [ ] Distributed tracing is implemented across all services
- [ ] Runbooks exist for common operational tasks and known failure modes
- [ ] Incident response process is defined and practiced
- [ ] On-call rotation is established with clear escalation procedures
- [ ] Dependency update process is defined and scheduled
- [ ] Backup and restore procedures are documented and regularly tested
- [ ] Disaster recovery plan exists and is tested at least annually

---

## Appendix A: The Master Checklist

This is the consolidated checklist. Use it as a gate review at each phase of your project.

### A.1 Concept & Vision
- [ ] Business case document exists and is reviewed
- [ ] Target users and workflows are documented
- [ ] Success metrics are defined
- [ ] Regulatory requirements are identified by region
- [ ] Performance and availability requirements are specified
- [ ] Integration requirements are documented
- [ ] Budget and timeline are established
- [ ] Stakeholders have signed off

### A.2 Architecture
- [ ] System architecture diagram exists
- [ ] ERD diagrams exist for all data domains
- [ ] API contracts are defined before implementation
- [ ] ADRs exist for all significant decisions
- [ ] Infrastructure architecture is documented
- [ ] Architecture docs are versioned and current
- [ ] A new team member can understand the system from architecture docs alone

### A.3 Design & Standards
- [ ] Coding standards are documented and enforced via tooling
- [ ] Security standards are documented
- [ ] Testing strategy is defined at all levels
- [ ] Documentation standards are defined
- [ ] Design system / UX standards exist
- [ ] Standards are referenced in AI context files
- [ ] Automated enforcement is in place

### A.4 People, Roles & Tools
- [ ] Team roles are defined with clear responsibilities
- [ ] AI assistants have defined scope and review requirements
- [ ] Context file (CLAUDE.md) exists and is current
- [ ] Tools are selected and standardized
- [ ] Code review is required for all contributions
- [ ] Human-in-the-loop checkpoints are defined

### A.5 Planning & Execution
- [ ] Project is phased with milestones
- [ ] Tasks are decomposed into specific work orders
- [ ] Status tracking is active and current
- [ ] CI/CD pipeline enforces all quality gates
- [ ] Code review applies to all code equally

### A.6 Quality & Verification
- [ ] Tests exist at unit, integration, and E2E levels
- [ ] Performance tests validate capacity
- [ ] Security scanning is automated
- [ ] Compliance is traceable
- [ ] Documentation matches reality

### A.7 Deployment & Operations
- [ ] Deployment strategy is defined with rollback capability
- [ ] Monitoring and alerting are comprehensive
- [ ] Runbooks and incident response are documented
- [ ] Maintenance plan is active
- [ ] Disaster recovery is tested

---

## Appendix B: AI-Specific Guidance

### B.1 The Golden Rules

1. **AI is a power tool, not yet an architect.** It multiplies the quality of your specifications. Good specs in, good code out. Bad specs in, confidently bad code out — at scale.

2. **Never let AI make architectural decisions.** Service boundaries, data models, communication patterns, security architecture — these are human decisions that require business context AI doesn't yet reliably hold across sessions.

3. **Context is everything.** An AI assistant with a well-written CLAUDE.md, architecture docs, and clear task specifications will outperform one with just a prompt. Invest in context the way you invest in onboarding.

4. **Review AI output the way you review junior developer output.** It compiles. It probably passes tests. But does it follow your patterns? Does it handle edge cases? Does it fit the architecture? Would you approve this PR from a first-week engineer without scrutiny?

5. **AI doesn't yet remember across sessions.** Every session currently starts fresh. Your context files, your documentation, your specifications — these are the institutional memory that AI still lacks. If it's not written down, it doesn't exist for AI.

6. **Specify patterns by example.** Don't say "follow best practices." Say "follow the pattern in `AccountService.java` — constructor injection, repository pattern, Result return types, structured logging." Show, don't tell.

7. **Decompose ruthlessly.** The smaller and more specific the task, the better the AI output. "Build the payment system" will fail. "Implement the `validateTransaction` method per the spec in `transaction.yaml` following the pattern in `AccountValidator`" will succeed.

8. **Trust but verify.** AI-generated code should go through the same CI pipeline, the same code review, the same testing standards as human-written code. No shortcuts.

### B.2 Context File Template

```markdown
# [Project Name] - AI Context

## Project Overview
[One paragraph: what this system does, who it serves, why it exists]

## Architecture
[Brief summary of system architecture with reference to detailed docs]
- Key services and their responsibilities
- Communication patterns (REST, events, etc.)
- Key architectural decisions (reference ADRs)

## Tech Stack
- Language: [version]
- Framework: [version]
- Database: [type and version]
- Message Broker: [type]
- Infrastructure: [cloud provider, orchestration]

## Project Structure
[Directory layout with explanations]

## Coding Conventions
- Naming: [conventions]
- Patterns: [repository, service, adapter, etc.]
- Error Handling: [approach]
- Logging: [format and levels]
- Testing: [framework, patterns, location]

## Build & Run
[How to build, test, and run locally]

## Key References
- Architecture: [path to architecture docs]
- ERD: [path to ERD]
- API Specs: [path to specs]
- ADRs: [path to ADRs]
- Standards: [path to standards docs]

## Current Status
[What phase the project is in, what's active, what's blocked]

## Common Pitfalls
[Things that frequently go wrong, patterns to avoid]
```

### B.3 Anti-Patterns to Avoid

**"Just let the AI figure it out"**
- Giving vague, high-level instructions and expecting architectural coherence
- This is the equivalent of telling a junior developer "build something good"
- *Real example:* An AI agent told to "write integration tests for the account service" will mock the database, use camelCase field names, invent test data, and write unit tests — violating ADR-002 in every way. The fix: explicit prompts that include file paths to read, non-negotiable rules, and anti-pattern examples (see Section 4.6).

**"AI wrote it, ship it"**
- Merging AI-generated code without thorough review
- AI code compiles and often passes tests — but may violate patterns, duplicate logic, or introduce subtle bugs
- *Real example:* AI-generated code that validates JWTs locally looks correct and works — but bypasses role-based authorization, creating a critical security vulnerability (see Section 3.7, ADR-007). The code passed tests because the tests didn't check authorization boundaries.

**"We don't need docs, the AI can read the code"**
- AI reads what's in its context window, not your entire codebase
- Without documentation, AI will make assumptions that may not match your reality
- *Real example:* Without ADR-001 in context, an AI assistant will default to "database per service" — the industry standard that's wrong for our financial compliance requirements. Without ADR-006, it will INSERT directly into audit_logs instead of using the event bus. The AI isn't wrong in general; it's wrong for *our* context.

**"AI replaces the need for architecture"**
- AI can generate code faster, but speed without direction is just faster wrong
- Architecture is more important with AI, not less, because AI amplifies both good and bad patterns
- *Real example:* Before the three-layer payment architecture (ADR-008), each payment type's domain service implemented its own orchestration logic. AI accelerated the duplication — it built each new payment type by copying the pattern from the last, compounding the inconsistency. The architectural refactor (Domain → Orchestration → Data) was a human decision that no AI tool would have surfaced at the time.

**"One prompt to rule them all"**
- Trying to accomplish too much in a single AI interaction
- Decompose into focused, specific tasks with clear acceptance criteria
- *Real example:* "Build the reconciliation engine" is a prompt that will produce something. "Implement the three-tier matching algorithm in ReconciliationMatcher: first blockchain hash match, then exact match on amount/reference/date, then fuzzy match with 80% threshold — following the pattern in TransactionMatcher and testing against seed data in reconciliation_reports table" is a prompt that will produce the right thing.

### B.4 The Slash Command Toolkit — A Real Implementation

The guidance in this appendix is abstract without tooling to make it concrete. On the XFile project, we implemented three AI assistant slash commands that operationalize the golden rules:

**`/onboard` — Full Context Loading (Golden Rule #3: Context is everything)**

Reads all 11 ADRs and core architecture documentation into the AI's context window. Used at the start of complex work sessions — new features, architectural changes, multi-service refactoring. This is the equivalent of having a new crew member read the complete job site binder before picking up a tool.

When to use: Starting work on a new feature that spans multiple services, making architectural decisions, before proposing significant changes.

**`/context-check` — ADR Compliance Verification (Golden Rule #8: Trust but verify)**

Validates the current codebase against all ADR compliance rules and produces an actionable report (compliant / warning / violation). This is the automated inspection that catches architectural drift — direct database access across service boundaries, local JWT validation, direct audit log inserts, frontend calls bypassing the API gateway.

When to use: Before submitting code for review, after completing a feature, periodically during development to catch drift early.

**`/check-docs` — Documentation Drift Detection (Golden Rule #5: AI doesn't yet remember across sessions)**

Scans for contradictions between documentation and implementation — ADR violations in code, broken file path references, environment variable conflicts, competing authoritative sources. Because AI assistants rely on documentation for context, stale documentation leads directly to incorrect implementations.

When to use: After major refactors, before onboarding new team members, as part of quarterly documentation reviews.

These three commands form a cycle: load context → do work → verify compliance → check documentation. They're not a substitute for human judgment, but they automate the mechanical verification that humans skip when under pressure.

---

## Final Word: Build It Right

The tools have changed. The speed has increased. But the fundamentals of building quality software haven't changed at all.

You still need architecture before implementation. You still need standards before coding. You still need review before shipping. You still need monitoring before sleeping soundly.

AI coding assistants are the most powerful tools to enter the software engineering workshop in decades. Like any powerful tool, they are extraordinary in skilled hands and dangerous in unskilled ones. A nail gun in the hands of an experienced framer builds houses fast. A nail gun in the hands of someone who skipped the blueprints just puts holes in things faster.

Build the blueprints. Write the specs. Define the standards. Set up the inspections. Then hand the AI the nail gun.

**Build it right, or build it twice.**

---

*This primer is a living document. It should be updated as tools evolve, lessons are learned, and processes are refined. The best construction manuals are the ones with mud on the pages — they've been used on the job site, not just shelved in the office.*
