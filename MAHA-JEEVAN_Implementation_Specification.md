# MAHA-JEEVAN — Implementation specification for Claude Code / Codex

**Version:** 1.0 | Proposed Maharashtra drought, farmer and livestock relief management platform

**Status:** Engineering requirements, not an approved government policy or production authorization. All government integrations, eligibility rules, ration schedules, financial delegations and retention periods require departmental confirmation. Use synthetic data in demos.

## 1. Instructions to the coding agent

Act as a senior full-stack engineer and security-conscious solution architect. Build a runnable, tested application, not merely a design or static screens. Read this entire specification before coding. Inspect any existing repository first; preserve existing conventions and working functionality. If no repository exists, use the defaults below. Create a phased implementation plan and a requirements traceability checklist in `docs/implementation-status.md`. Implement vertical slices with database migrations, backend APIs, frontend UI, authorization, validation, tests and seed data. Run tests and builds after each slice. Do not claim a feature works unless it was tested. Do not invent external API endpoints or government credentials. Where approval is absent, use clearly labeled mock adapters. Document assumptions, unresolved policy decisions and remaining work. Never expose secrets, real Aadhaar information or real farmer records. Do not deploy to a live government environment without explicit authorization.

### Deliverables
- Runnable monorepo, Docker Compose development setup, `.env.example`, database migrations and seeded demo accounts; no production passwords.
- Responsive role-based web application, installable offline-capable PWA for field users, Marathi-first farmer interface with English/Hindi localization.
- OpenAPI specification, ER diagram, role-permission matrix, workflow state diagrams, test suite, sample reports and README with setup commands.
- Working end-to-end demo covering application, camp approval, vet admission, attendance, fodder and water delivery, farmer dispute, inspection, claim approval and district/state monitoring.

## 2. Architecture and engineering defaults

Use Java 21, Spring Boot 3.x compatible release, Spring Security, Maven, PostgreSQL with PostGIS, Flyway, React + TypeScript + Vite, Tailwind CSS, TanStack Query, React Router and i18next. Use a modular monolith initially; split services only when justified. Use Redis optionally for cache/rate limits; S3-compatible private object storage for evidence; background job/outbox worker for notifications and integrations. Use MapLibre with licensed basemap and authoritative administrative boundary data. Use REST JSON `/api/v1`, OpenAPI, UUID internal IDs, ISO timestamps UTC, display Asia/Kolkata; attendance/service date computed in IST. Use BigDecimal for quantities/money, integer paise or numeric fixed precision for financial calculations. Never use floating point for money. Use `kg`, `litre`, `INR` canonical units and explicit conversions. Use secure cookie-based sessions or approved OIDC with short-lived tokens and rotation; do not store long-lived credentials in browser localStorage. Enforce MFA on privileged accounts in production. Implement dev login only behind a development profile.

Suggested repository: `backend/`, `frontend/`, `infra/`, `docs/`, `tests/e2e/`. Backend modules: identity, geography, policy, eligibility, camps, livestock, attendance, inventory, water, grievance, veterinary, inspection, finance, analytics, notification, audit, integration. Use transactions, optimistic locking, idempotency keys and transactional outbox. Prefer append-only transaction ledgers and correction events instead of deleting records.

## 3. Geography, identity and authorization

Hierarchy: State → Division → District → Taluka → Village/Gram Panchayat → Camp → Farmer → Animal. Maintain authoritative LGD codes and effective-dated mappings; support villages with no camp, multiple camps, and authorized inter-jurisdiction transfers. Do not assume the revenue and animal husbandry reporting chains are identical. Assign users multiple roles with explicit geographical/camp scopes and validity dates. Every server query and mutation must enforce role, scope, object ownership and workflow permission; hiding buttons is not security. State aggregate access does not automatically grant access to personal documents. Separate maker/checker and financial sanction permissions. No user may approve their own claim or inspection where separation is required. Log privileged access and exports. Farmer sees only their own animals, applications and deliveries. Supplier sees only its own orders/trips. Complaint identity may be masked from operators, but available to authorized investigators.

## 4. Role-wise product specification

Each role has a home dashboard, scoped navigation, search/filter, notifications, accessible form validation, history and relevant reports. For each listed action implement a corresponding backend permission and integration test. Read-only roles must not mutate data.

### 4.1 Farmer (`FARMER`)

**Scope:** Own household and animals.

**Screens/dashboard:** Mobile OTP login, language selection, home with application status, animals, camp, today’s fodder, water and complaint status.

**Functional actions:** Submit relief application with approved identity reference, village, ownership/land evidence where required; assisted submission for non-smartphone users. View animal tag masked, admission, health history and attendance. Receive delivery and tanker ETA/arrival notifications; confirm full/partial/not received/poor quality; enter actual quantity and optional photo or Marathi voice note. Confirm drinking-water adequacy, not bulk tanker volume. File confidential grievance, view ticket, timeline, resolution and reopen. Request correction to ownership or records.

**Restrictions:** Cannot change vet certification, attendance, stock, claims or other farmers’ data. No response is pending, never deemed consent. SMS/IVR assisted acknowledgement must capture actor and channel.

### 4.2 Government veterinary doctor / LDO (`VET_LDO`)

**Scope:** Assigned hospital, villages and camps.

**Screens/dashboard:** Pending examinations, registered animals, upcoming visits, disease alerts, treatment queue.

**Functional actions:** Search/scan existing 12-digit Bharat Pashudhan tag, manual entry fallback; verify ownership and eligibility; register local operational animal record, capture photo, species, breed, age, sex, weight, category and examination findings. Issue digitally recorded health certificate, recommend admission or quarantine, record vaccinations, treatment, prescription, referrals, mortality and discharge verification. Correct category only with reason and review; check active admission across all camps without exposing unrelated personal data.

**Restrictions:** Do not fabricate national identity or successful lookup. National integration requires authorization; offline records remain provisional until conflict checks. Only qualified authorized vet records medical findings.

### 4.3 Talathi / village revenue officer (`TALATHI`)

**Scope:** Assigned villages.

**Screens/dashboard:** Pending farmer eligibility, NOC requests, document verification, drought village status.

**Functional actions:** Review application, residence/ownership evidence and 7/12 or approved alternative where applicable; verify against authorized sources or physical records; approve, reject or request clarification with reason; issue digital NOC record; record suspected duplicate or disputed ownership; forward exceptions.

**Restrictions:** Cannot mark drought notification without authority, certify animal health, approve claims or access unrelated villages. Landless livestock owners must follow configured eligibility rules, not automatic rejection.

### 4.4 Gram Sevak / village relief coordinator (`VILLAGE_COORDINATOR`)

**Scope:** Assigned villages.

**Screens/dashboard:** Village livestock demand, relief requests, camp availability, unresolved issues.

**Functional actions:** Assist farmers with registration, record village-level demand and water shortages, publish authorized camp information, support assisted complaints and referrals; escalate emergencies.

**Restrictions:** Assistance is not eligibility approval; record farmer authorization and actor.

### 4.5 Camp operator (`CAMP_OPERATOR`)

**Scope:** Own approved camps.

**Screens/dashboard:** Capacity, admissions, attendance completion, stock, deliveries, water demand, claims and inspection tasks.

**Functional actions:** Apply for camp sanction; enter location, infrastructure, capacity and operator documents; upload evidence; after activation admit vet-cleared animals subject to capacity and statewide uniqueness; request transfer/discharge; record daily QR/barcode attendance, fodder receipts, weighing, distribution, spoilage and stock transfers; request water tanker and fodder supply; submit claims with documents and reconciliation; respond to authorized queries.

**Restrictions:** Cannot approve own camp, confirm on farmer behalf without recorded authorized assisted workflow, change vet category, delete historical data or sanction payment.

### 4.6 Camp supervisor (`CAMP_SUPERVISOR`)

**Scope:** Assigned camps.

**Screens/dashboard:** Pending attendance, stock receipt, tanker unloading, exceptions.

**Functional actions:** Verify physical attendance and manual exceptions; record bulk water received, quality checks and delivery evidence; review fodder distribution, stock adjustments and discrepancies; sign off operational records; report CCTV outage; request independent inspection.

**Restrictions:** Do not overwrite farmer disputes. Separate supervisor and operator for checker controls where required.

### 4.7 Water tanker supplier / dispatcher (`TANKER_SUPPLIER`)

**Scope:** Own fleet and orders.

**Screens/dashboard:** Assigned orders, vehicles, drivers, trip status, invoice status.

**Functional actions:** Register vehicle capacity, driver assignment, approved source; accept or decline orders, schedule dispatch and replacement; view telemetry and proof of delivery; submit invoice tied to verified trip.

**Restrictions:** Cannot approve own delivery, alter historical GPS or see other suppliers’ orders.

### 4.8 Tanker driver (`TANKER_DRIVER`)

**Scope:** Assigned active trips.

**Screens/dashboard:** Today’s trip, source, destination, navigation and delivery checklist.

**Functional actions:** Start trip, record source fill, transmit GPS with consent and active-trip scope, mark arrival, scan camp QR, capture unloading evidence and actual quantity; submit trip completion or incident; offline queue with timestamps.

**Restrictions:** GPS and QR are evidence, not proof of quantity. No tracking outside assigned active trip. No driver self-approval.

### 4.9 Fodder supplier (`FODDER_SUPPLIER`)

**Scope:** Own purchase orders.

**Screens/dashboard:** Orders, dispatch, deliveries, accepted quantities and invoice status.

**Functional actions:** Accept approved purchase order, record lot/type/quantity, challan, vehicle and weighbridge slip; dispatch and submit proof; invoice against accepted receipts; respond to rejected quality or quantity.

**Restrictions:** Cannot post camp inventory or approve own invoice.

### 4.10 Taluka veterinary / animal husbandry officer (`TALUKA_AH_OFFICER`)

**Scope:** Assigned talukas.

**Screens/dashboard:** Camp health coverage, stock, attendance anomalies, complaints, inspections.

**Functional actions:** Review vet and camp operational records, assign doctor visits, inspect animal welfare, review disease alerts and category disputes, verify operational claim evidence, assign/coordinate field inspections, recommend actions and escalate shortages.

**Restrictions:** Financial authority only if separately assigned under approved delegation.

### 4.11 Flying squad / inspection officer (`INSPECTOR`)

**Scope:** Assigned inspection missions.

**Screens/dashboard:** Inspection tasks, due dates, offline capture.

**Functional actions:** Accept assigned or authorized surprise inspection; record GPS/time, independent physical headcount by category before revealing system count; verify fodder stock, water, CCTV and shelter; capture evidence, discrepancy and recommendation; submit signed inspection report; follow-up verification.

**Restrictions:** Cannot edit original counts after submission; correction requires append-only amendment. No automatic finding of fraud from anomaly alone.

### 4.12 Tahsildar (`TAHSILDAR`)

**Scope:** Assigned talukas.

**Screens/dashboard:** Camp applications, eligibility exceptions, claims, grievances, tanker requests.

**Functional actions:** Review camp sanction recommendations and inspection evidence; approve operational requests only within delegation; review claim period, verified animal-days, delivery reconciliation, inspection findings and disputes; return for correction or recommend to authorized sanction authority; assign/escalate complaints and emergency resource requests.

**Restrictions:** Cannot override unresolved material discrepancies without authorized exception and reason. No self-approval or out-of-scope sanctions.

### 4.13 District animal husbandry officer (`DISTRICT_AH_OFFICER`)

**Scope:** Assigned district.

**Screens/dashboard:** Veterinary workload, outbreaks, mortality, camp compliance, supplies.

**Functional actions:** Assign vets and review health indicators, coordinate isolation and emergency response, review inspection results, verify district animal welfare and operational compliance; recommend redistribution and corrective measures; monitor clinical coverage.

**Restrictions:** Medical data access limited to assigned duties; financial sanction separately delegated.

### 4.14 District Collector (`DISTRICT_COLLECTOR`)

**Scope:** Assigned district.

**Screens/dashboard:** District command dashboard with taluka/village/camp drilldown, alerts, claims, complaints and inspections.

**Functional actions:** Monitor verified and pending headcounts, fodder stock by camp and days remaining, tanker GPS and delivery status, water availability, overdue complaints, disease/mortality, inspections and financial stages. Assign inspection or investigation, escalate resources, approve/reject camp and claim only within configured delegation; authorize eligible transfers; download audited reports.

**Restrictions:** Read personal data only with purpose and scope. Financial approval distinct from payment. Do not override evidence automatically.

### 4.15 Divisional commissioner / regional officer (`DIVISION_OFFICER`)

**Scope:** Assigned division.

**Screens/dashboard:** District comparison, high-risk camps, unresolved escalations and resource gaps.

**Functional actions:** Monitor districts, coordinate cross-district resource allocation, assign escalated review, compare compliance and timeliness, export aggregate reports; exercise approvals only when separately delegated.

**Restrictions:** No default access to raw identity documents or unrestricted financial authority.

### 4.16 State animal husbandry commissioner (`STATE_COMMISSIONER`)

**Scope:** Statewide operational aggregates and authorized drilldown.

**Screens/dashboard:** State GIS command centre, district scorecards, stock risk, tanker coverage, disease alerts, grievances and financial status.

**Functional actions:** Monitor all districts; drill down to camps with appropriate permissions; issue inspection tasks and resource coordination orders, approve authorized inter-district redistribution, set performance targets and review escalated matters; view policy impact and audit summaries.

**Restrictions:** No implicit ability to change source attendance, approve every payment or access all personal documents.

### 4.17 Finance verifier / sanction officer / payment operator (`FINANCE_VERIFIER / SANCTION_OFFICER / PAYMENT_OPERATOR`)

**Scope:** Assigned jurisdiction and delegated limits.

**Screens/dashboard:** Claim queue, reconciliation, sanctions and payment status.

**Functional actions:** Verifier checks evidence and calculation; sanction officer reviews verified claim and records approval/rejection; payment operator initiates authorized external payment or records verified transaction reference and reconciliation; handle failed/returned payments and adjustments.

**Restrictions:** Three separate permissions; no self-approval, no fake payment success, no double payment; sandbox adapter only until official authorization.

### 4.18 State system administrator / policy administrator (`SYSTEM_ADMIN / POLICY_ADMIN`)

**Scope:** Explicit administrative scope.

**Screens/dashboard:** Users, roles, scopes, policy versions, integrations, audit, system health.

**Functional actions:** Provision accounts, revoke sessions, assign jurisdiction, manage master data; policy admin drafts ration schedules, category rules, eligibility, attendance rules, claim periods, rates, SLAs and approval matrices; authorized second approver publishes effective-dated version; monitor integrations and backups.

**Restrictions:** Technical admin cannot automatically approve financial or operational transactions; no silent policy edits; protect audit logs and secrets.

### 4.19 Auditor / public viewer (`AUDITOR / PUBLIC_VIEWER`)

**Scope:** Auditor assigned scope; public aggregate only.

**Screens/dashboard:** Auditor evidence timeline and exports; public camp directory.

**Functional actions:** Auditor reads claim lineage, inspection evidence, changes and access logs; export authorized records. Public QR shows approved camp name, status, capacity/availability and official contact channel; publish approved aggregate indicators.

**Restrictions:** No public farmer identity, tag, CCTV feed, exact private location or internal complaint details; auditor read-only.

## 5. End-to-end domain workflows and invariants

### 5.1 Relief activation and farmer eligibility
Authorized authority creates a relief event with notified villages, effective dates, policy version and reference order. Farmer application: DRAFT → SUBMITTED → UNDER_REVIEW → NEEDS_INFORMATION / APPROVED / REJECTED / WITHDRAWN. Require reason for rejection and preserve history. Attach document references only where required; private storage with signed URLs and malware scanning. Approved eligibility is effective-dated; changes do not silently rewrite historical claims. Allow assisted and offline capture; do not deny essential care solely because digital verification is temporarily unavailable; route exceptions for human review.

### 5.2 Camp sanction and admission
Camp: DRAFT → SUBMITTED → INSPECTED → RECOMMENDED → SANCTIONED → ACTIVE → SUSPENDED / CLOSED. Approval checklist: shade, capacity by category, water storage, troughs, fodder storage, buffer stock, hygiene, veterinary assignment, isolation and CCTV if required. Evidence GPS/time recorded with accuracy and consent where appropriate. Camp approval and admission limits configurable. Animal admission: REQUESTED → VET_CLEARED → ADMITTED → TRANSFER_PENDING → TRANSFERRED / DISCHARGED / DECEASED. Validate eligibility, category, capacity, policy and active admission. Enforce unique active admission per animal in database using partial unique index; transfer is atomic, closes origin and opens destination without overlapping eligibility. Use immutable national tag identifier with unique constraint; correct erroneous tags through controlled identity merge/correction, never arbitrary edits. Tag stored as string (leading zeros preserved), exactly 12 digits where applicable. National ID verification source/status recorded separately from local registration.

### 5.3 Attendance
Attendance events record animal, camp, service date, session, actor, scan/manual method, device, GPS accuracy, captured time, received time, evidence and sync status. Policy defines required sessions (example morning/evening), acceptable geofence, exemptions and review. Unique `(admission_id, service_date, session)`; retries return same result. Record out-of-scope, duplicate, late, post-discharge and poor GPS quality as reviewable exceptions. Supervisor verifies attendance; correction requires reason, original event, authorized checker. Daily eligibility derived from effective policy and verified records. GPS/barcodes/CCTV can be manipulated; use independent physical audits and anomaly alerts, not claims of fraud-proofing. CCTV status/evidence references restricted; live streams optional and separately authorized.

### 5.4 Fodder inventory and distribution
Procurement: REQUESTED → APPROVED → ORDERED → DISPATCHED → RECEIVED → QUALITY_ACCEPTED / REJECTED → CLOSED. Stock ledger entries: receipt, issue, return, transfer out/in, spoilage, approved adjustment. Quantity movement and financial verification separate. On physical issue atomically post stock movement and record distribution; farmer confirmation updates reconciliation status, never deducts again. Pending/contested delivery remains visible as unreconciled. Prevent negative available stock, support reserved quantity and concurrent issues using transactional locking. Daily ration entitlement computed from verified attendance/category and effective policy. Distribution workflow: PREPARED → DISTRIBUTED → FARMER_CONFIRMED or FARMER_DISPUTED or AWAITING_RESPONSE → SUPERVISOR_AUDITED / CORRECTED. Keep original quantities, actual farmer-reported amounts and evidence. Partial delivery may be followed by additional delivery; do not count twice. Stock days = usable available stock / recent or projected daily consumption by type; zero consumption shows unknown/not applicable. Alert when forecast falls below replenishment lead time; no hardcoded official thresholds.

### 5.5 Water tanker / JalDoot
Water request: DRAFT → SUBMITTED → APPROVED → ASSIGNED → DISPATCHED → ARRIVED → UNLOADED → RECEIPT_VERIFIED → CLOSED / DISPUTED / CANCELLED. Trip includes source, destination, vehicle, capacity, planned and actual timestamps, telemetry, delivery quantity and evidence. Driver GPS only during trip; stale telemetry clearly marked. Camp QR verifies destination reference, not volume. Supervisor records actual received quantity and independent representative verification where required. Farmer records water adequacy separately; no requirement for every farmer to confirm bulk volume. Maintain water storage ledger, consumption estimate, losses and adjustments. Alert late tanker, route discrepancy, insufficient water, missing evidence and poor water quality. Allow partial delivery and emergency request with authorization. Notifications for ETA, arrival and shortages via approved channels.

### 5.6 Grievance and inspections
Complaint: SUBMITTED → ACKNOWLEDGED → ASSIGNED → IN_PROGRESS → RESOLVED → CLOSED / REOPENED. Escalate based on configurable SLA and severity; preserve history and evidence; notify farmer; allow confidential reporting and Marathi voice note. Delivery disputes automatically create/link complaint idempotently. Do not require proof for initial complaint. Emergency animal welfare issues routed immediately to authorized responders. Inspection: ASSIGNED → ACCEPTED → IN_PROGRESS → SUBMITTED → REVIEWED → FOLLOW_UP / CLOSED. Independent count locked before showing expected system count; amendments retain prior version. Risk flags indicate investigation need, not guilt. Record remedial actions and completion evidence.

### 5.7 Veterinary health
Record visit, symptoms, examination, diagnosis/provisional diagnosis, treatment, prescription, vaccinations, isolation and referral by authorized veterinarian. Disease alerts follow approved reporting protocol. Mortality verification records date and clinical evidence; stop future eligible attendance/claims according to policy and flag already submitted overlapping claims for reconciliation. No automated clinical diagnosis. Do not expose medical information beyond authorized roles.

### 5.8 Claims and payments
Claim lifecycle: DRAFT → SUBMITTED → OPERATIONAL_VERIFIED → ELIGIBILITY_VERIFIED → SANCTIONED → PAYMENT_PENDING → PAID / PAYMENT_FAILED; RETURNED, REJECTED, CANCELLED and ADJUSTMENT paths with reasons. Calculate eligible animal-days from frozen verified attendance snapshots and effective-dated category rates. Formula: sum eligible animal-days × applicable daily rate, plus permitted components minus authorized adjustments; policy decides whether expenses are included, never assume reimbursement beyond rate. Claim period configurable (15 days example, not mandatory). Enforce uniqueness of claimed eligible animal-day across claims including pending claims; amendments/reversals explicitly linked. Material disputes or missing evidence block affected items or route to authorized exception per policy; unaffected items may proceed if allowed. Store calculation breakdown, policy version, evidence hashes, approval history and amount. Maker/checker, delegated limits and scope enforced. Payment adapter records request idempotency, callback verification, transaction reference, amount and reconciliation; no real transfer in demo. Never mark paid solely from UI action or unverified callback. Reports differentiate sanctioned, verified, paid and pending values.

### 5.9 Closure and redistribution
Camp closure requires reconciled animal exits, inventory balance/disposition, pending complaints and claims disposition, final inspection and authorized approval. Historical records remain readable. Resource transfer: request → approve → dispatch → receive → reconcile; origin and destination ledgers linked; prevent double stock and jurisdiction leakage. Emergency operations supported with documented exceptions.

## 6. Data model (minimum viable schema)

All mutable master records carry UUID, created_at/by, updated_at/by, version; transactional records carry event timestamp, actor, scope and correlation ID. Foreign keys and check constraints mandatory. Prefer normalized data; use JSONB for versioned evidence/checklist payloads only. Scope is derived server-side, not trusted from client.

| Entity | Key fields and constraints |
|---|---|
| users, roles, permissions, role_permissions, user_role_scopes | login identity, status, role, jurisdiction/camp, validity; unique scoped grant |
| geographic_units | LGD code, type, parent, effective dates, geometry; validated hierarchy |
| relief_events, policy_versions | order reference, validity, publication, approver, rule JSON, version |
| ration_rates, subsidy_rates, approval_rules | policy, category, quantity/unit or amount, dates, delegated role/limit |
| farmers, farmer_applications, farmer_documents, eligibility_reviews | owner ID, contact encrypted, village, status, evidence ref, decision history |
| camps, camp_applications, camp_infrastructure, camp_approvals | location, operator, capacity, status, effective dates, checklists |
| animals, animal_identity_verifications | tag unique, owner, species, category, verification source/status, masked display |
| animal_admissions, animal_transfers | animal, camp, start/end, category snapshot, health cert, active uniqueness |
| veterinary_exams, treatments, vaccinations, mortality_reports | animal, vet, clinical details, status, date, evidence |
| attendance_events, attendance_reviews, eligible_animal_days | admission, date, session, method, evidence, policy, unique day/session |
| suppliers, purchase_orders, purchase_order_lines, goods_receipts | supplier, camp, item, ordered/received/accepted quantity, lot, invoice ref |
| stock_items, stock_ledger, stock_reservations, stock_transfers | camp, item, quantity/unit, movement, lot, reference; unique source event |
| ration_entitlements, fodder_distributions, farmer_delivery_responses | admission/date, entitlement, actual quantity, status, response, evidence |
| water_requests, tankers, tanker_drivers, tanker_trips, gps_points | source/destination, capacity, status, timestamps, coordinates/accuracy |
| water_receipts, water_ledger, water_adequacy_responses | trip/camp, quantity, quality, supervisor, farmer feedback |
| complaints, complaint_events, inspection_tasks, inspection_reports | category, priority, assignment, SLA, independent counts, discrepancy, evidence |
| claims, claim_lines, claim_approvals, payment_attempts, adjustments | period, policy, eligible day, money, stage, approver, external ref |
| notifications, notification_attempts, outbox_events | recipient, channel, template locale, status, retry, event idempotency |
| files, audit_events, anomaly_flags, sync_operations | private object key, hash, actor, event, reason, device, idempotency key |

Indexes: geography parent, camp scope/status, admissions active animal, attendance date/camp, stock camp/item, trip status/camp, complaints scope/status/SLA, claims scope/period/status, outbox pending. Use GiST for geometries. Do not expose raw object storage keys or private data in public APIs.

## 7. API contract and frontend routes

All endpoints scoped and authorized; use pagination, filters, stable sorting, structured error codes, correlation IDs and idempotency keys for commands. Return 409 for conflicting state/duplicate, 422 for invalid transition/validation, 403 for forbidden access (or 404 where needed to prevent existence disclosure). Use explicit action endpoints; never allow unrestricted status updates via generic PATCH.

| Domain | Representative endpoints |
|---|---|
| Auth | POST /auth/otp/request, /auth/otp/verify, /auth/logout; GET /me |
| Geography/policy | GET /geography, /policies/active; POST /policies/{id}/publish |
| Farmer | POST /applications; GET /me/animals, /me/deliveries, /me/complaints; POST /applications/{id}/review |
| Camp | POST /camps; POST /camps/{id}/inspect, /approve, /activate, /close; GET /camps/{id}/summary |
| Animal | GET /animals/lookup?tag=... (authorized); POST /animals, /animals/{id}/exams, /admissions, /admissions/{id}/transfer, /discharge |
| Attendance | POST /attendance/events, /attendance/{id}/verify, /attendance/{id}/corrections; GET /camps/{id}/attendance?date=... |
| Inventory | POST /purchase-orders, /goods-receipts, /stock-transfers, /stock-adjustments; GET /camps/{id}/stock |
| Fodder | POST /distributions, /distributions/{id}/issue, /distributions/{id}/responses; GET /me/deliveries |
| Water | POST /water-requests, /water-requests/{id}/approve, /trips, /trips/{id}/telemetry, /arrive, /unload, /verify; POST /water-adequacy-responses |
| Grievance | POST /complaints, /complaints/{id}/assign, /resolve, /reopen; GET /complaints |
| Inspection | POST /inspections, /inspections/{id}/counts, /submit, /review; GET /inspections |
| Finance | POST /claims, /claims/{id}/submit, /verify, /sanction, /payments; GET /claims/{id}/breakdown |
| Monitoring | GET /dashboards/summary, /dashboards/camps, /dashboards/resources, /dashboards/alerts, /reports/{type} |
| Sync/evidence | POST /sync/batch, /files/upload-init, /files/{id}/complete; GET /notifications; POST /notifications/{id}/read |

Frontend route groups: `/farmer`, `/vet`, `/village`, `/camp`, `/supervisor`, `/supplier`, `/driver`, `/inspector`, `/taluka`, `/district`, `/division`, `/state`, `/finance`, `/admin`, `/audit`, `/public/camp/:publicId`. Separate route guards plus backend enforcement. Each dashboard includes loading, empty, error, stale/offline states and last updated timestamp. Provide accessible forms, large touch targets, Marathi labels, local number/date display, screen reader labels and keyboard support. No hardcoded data in production screens; use seed API for demo. Use original MAHA-JEEVAN branding asset if supplied; otherwise text placeholder clearly labeled.

## 8. Dashboard KPIs and reports

Every KPI has definition, time window, scope, source, last refresh and drilldown. Never mix admitted animals with present animals or allocated water with delivered water. Distinguish pending confirmation from dispute. Show incomplete data and offline sync lag.

| KPI | Definition |
|---|---|
| Active camps | ACTIVE at selected timestamp |
| Admitted animals | active admissions by category |
| Verified present | distinct eligible/verified attendance by service date, show policy/session |
| Attendance completion | required verified sessions / required sessions for active admissions |
| Fodder stock | sum posted usable ledger balance by item and camp |
| Days remaining | usable stock / estimated daily consumption, with uncertainty |
| Water delivery compliance | verified delivered litres / approved scheduled litres for selected period |
| Tanker status | trips by state, GPS last seen, ETA only when supported |
| Farmer confirmation | confirmed deliveries / issued deliveries; show pending and disputed separately |
| Grievance SLA | open overdue, median resolution time, reopened cases |
| Inspection variance | independent physical count minus system count at snapshot; percentage only if denominator nonzero |
| Health | visits, cases, verified mortality, outbreak alerts by geography |
| Financial | claimed, verified, sanctioned, paid, failed and outstanding sums separately |

Collector dashboard: filters district → taluka → village → camp; GIS markers for camps/tankers, stock shortages, complaints and inspection queue; action to assign investigation; camp detail with attendance, stock, water, health, complaints and claims tabs. State dashboard: district comparisons, regional alerts, resource redistribution, trends, report exports. Use accessible map alternative table. Do not publicly expose individual animal or farmer location.

Reports: daily muster, camp capacity, fodder stock ledger, ration distribution and discrepancies, tanker trip/receipt, water adequacy, farmer grievance SLA, inspection report, vet health summary, claim calculation and approval history, district/state consolidated report, audit export. CSV and printable PDF optional in MVP; export access logged, sensitive fields masked, large exports queued.

## 9. Integrations, offline operation and security

Adapters: Bharat Pashudhan lookup (read-only mock until approved), identity/land verification (manual fallback), SMS/WhatsApp/IVR/push (consent and approved templates), GPS telemetry, CCTV evidence, maps/geography, financial payment gateway (mock), optional rainfall/drought indicators. Each adapter has interface, mock implementation, timeout/retry/circuit breaker, correlation ID, error status and explicit production configuration. Never scrape restricted portals or bypass authentication. No real Aadhaar collection in demo; use minimal authorized reference, masking and encryption. Do not send sensitive records to AI services without approval.

Offline: encrypted local queue for authorized field device, unique client operation ID, captured time and server received time, sync results ACCEPTED / DUPLICATE / CONFLICT / REJECTED, manual conflict review, retry with backoff. No offline final claim sanction, no offline statewide uniqueness guarantee; provisional admission pending server validation. Minimize local personal data, expire caches, support revocation and device lost procedure. UI must show pending sync and not claim live records when offline.

Security: TLS, least privilege, password hashing if passwords used, OTP throttling, secure cookies/CSRF, CORS allowlist, input validation, parameterized queries, dependency scanning, upload type/size validation and malware scan, private encrypted backups, secrets management, audit trail with tamper-evident hashes/access restrictions, retention schedule and deletion/legal hold policy approved by authority. Threat model includes cross-tenant access, duplicate admission race, fake GPS, forged payment callback, tampered offline events, malicious uploads, operator/farmer coercion and admin abuse. Include independent verification and human review. Production hosting, security audit and applicable Indian privacy/government standards require formal review; no blanket compliance claim.

Notifications: event-driven outbox, deduplicate, retry, templates Marathi/Hindi/English, channel preferences, SMS fallback; never include sensitive full identifiers. Farmer notifications: eligibility decision, admission, daily ration, tanker ETA/arrival, dispute updates, discharge. Officers: shortage, delayed tanker, disease, overdue complaint, inspection variance, claim review. Record delivery failure, not assumed receipt.

## 10. Acceptance tests and demo scenario

Use automated unit/integration tests with PostgreSQL Testcontainers and Playwright browser E2E. Include negative paths and concurrency. Minimum acceptance criteria:

1. Farmer A cannot access farmer B by changing URL or UUID; district A cannot query district B; supplier cannot approve own receipt.
2. Approved eligibility and active camp allow vet-cleared admission; two concurrent admissions of same tag result in at most one active admission; capacity respected; offline duplicate flagged.
3. Attendance retry does not duplicate; post-discharge event rejected or flagged; authorized correction preserves original; policy version selected by service date.
4. Fodder issue decreases stock exactly once; simultaneous issue cannot create negative stock; farmer dispute creates one complaint and preserves original; no response stays pending.
5. Tanker trip rejects invalid state transition; supervisor records actual quantity; farmer adequacy separate; stale GPS clearly marked; retry does not double receipt.
6. Inspection independent count remains hidden from expected system count until locked; report amendments auditable.
7. Claim uses correct category/date/rate and verified animal-days; duplicate day cannot be claimed concurrently; disputed items handled per configured policy; unauthorized/self-approval fails; payment retry/callback cannot double-pay; mock payment visibly identified.
8. Collector sees village/camp breakdown and all authorized alerts; Commissioner sees aggregate district comparison; public QR hides private data.
9. Offline event sync idempotent; conflict visible; uploads unauthorized access denied; audit events recorded for approval/export/correction.
10. Marathi UI works on 360px viewport, accessible keyboard navigation on desktop, no broken translation keys; demo accounts and sample data clearly labeled.

Seed fictional scenario: one relief event; two districts with two talukas each; multiple villages/camps; farmers with cows/buffaloes/calves; two suppliers and tankers; one low-stock camp, delayed tanker, disputed fodder delivery, inspection variance, pending claim and mock paid claim. Walkthrough: farmer applies → Talathi verifies → camp sanctioned → LDO verifies tag/health → admission → attendance → issue fodder → farmer disputes → tanker receipt and adequacy feedback → inspection → correction → claim calculation → authorized sanction → mock payment → state dashboard. Provide reset command. Never use actual identities or official rates; label all examples.

## 11. Implementation phases and definition of done

Phase 0: repository scaffold, Docker Compose, migrations, identity/RBAC, geography, i18n, UI shell, seed data, CI. Phase 1: farmer eligibility, camp sanction, animal/vet registration, admission and attendance, including uniqueness and offline provisional flow. Phase 2: inventory, fodder distribution, farmer confirmations, tanker workflows, grievance and notifications. Phase 3: inspections, claims and mock payments, district/state dashboards, reports and audit. Phase 4: hardening, offline sync, accessibility, performance, disaster recovery, authorized integration adapters, deployment documentation. Start with modular monolith; deliver runnable slices, not empty pages. For each phase update traceability, tests, migration instructions and known limitations. Definition of done: acceptance tests pass, no unimplemented critical buttons, server-side permissions tested, demo reset works, documented mock integrations, no exposed secrets, reviewer can reproduce setup from README.

## 12. Open policy decisions (must not silently hardcode)

Confirm official current GRs and applicable relief scheme; eligibility and acceptable documents; camp sanction delegation; animal classification and tag lookup authorization; daily attendance sessions and GPS/CCTV requirements; ration quantities and subsidy rates; stock thresholds; tanker quality criteria; complaint SLA; inspection thresholds; claim cycle, eligible expense components and approval limits; payment integration; data retention, hosting and privacy approvals; official Marathi terminology and branding authorization. Treat example 15-day claim period, 250m geofence, twice daily attendance and 3-day buffer from the earlier blueprint as configurable demonstration defaults only, not validated legal requirements. No claims of zero fraud, guaranteed savings, instant resolution or government endorsement.

## 13. Suggested first message to Claude Code / Codex

Read `MAHA-JEEVAN_Implementation_Specification.md` in full. Inspect the existing repository and implement the system described, starting with Phase 0 and the Phase 1 vertical slice. Create `docs/implementation-status.md` with requirement IDs, planned files, acceptance criteria and implementation status. If this is a new repository, scaffold using the specified stack and provide Docker Compose, migrations, demo accounts and setup commands. Implement real backend persistence, role/geography authorization and functional Marathi-first screens; do not produce only mockups. Run automated tests and builds, fix failures, then proceed through remaining phases as permitted by the coding environment. Use mock adapters for unapproved government integrations. Report exactly what was implemented, what was tested, commands to run, and what remains. Do not request real government credentials or deploy externally. Preserve existing work and avoid destructive changes.

## Source and design provenance

Based on the supplied `maha_pashudhan_master_blueprint.md` and the MAHA-JEEVAN project discussion. Role descriptions, endpoint names, state transitions, technical defaults, test cases and security controls are proposed engineering elaborations. The blueprint's legal-compliance and fraud-proof claims are not treated as verified facts. This is a specification, not implemented software.
