# SIM Platform — Comprehensive Database Review

Date: 2026-11-25  
Status: Logical design reviewed; requirements reconciliation and runtime verification pending  
Architecture: Laravel modular monolith, MySQL/InnoDB, React, Redis

## 1. Scope and authority

The Owner delegated engineering decisions for the review while requiring existing PRDs to remain unchanged. This report records logical findings, implementation decisions and unresolved issues. It does not amend any PRD, execute migrations or certify production readiness.

Sources inspected: SIM_Identity_and_Access_Database_Design.md, SIM_Courses_Database_Design.md, SIM_Schedule_Builder_Database_Design.md, SIM_Assignments_Database_Design.md, SIM_Exams_Database_Design.md, SIM_Projects_and_Teams_Database_Design.md, SIM_Notifications_Database_Design.md and SIM_Audit_Log_Database_Design.md; relevant PRD excerpts and prior approved review decisions. Multiple document versions exist; a complete version-controlled PRD comparison has not been performed.

## 2. Schema inventory

| Design document | Tables | Count |
|---|---|---:|
| Identity & Access | users, otp_verifications, academic_levels, groups, ban_records | 5 |
| Courses | courses, lectures, sections, educational_files, ai_summaries, content_views | 6 |
| Schedule Builder | semesters, course_schedule_options, student_schedules, schedule_selections, one_time_schedule_changes, schedule_confirmation_history, schedule_modification_history | 7 |
| Assignments | assignments, assignment_submission_statuses | 2 |
| Exams | exams | 1 |
| Projects & Teams | projects, teams, team_members, team_invitations, team_join_requests | 5 |
| Notifications | notifications, notification_recipients, notification_preferences, push_subscriptions | 4 |
| Audit Log | audit_logs | 1 |
| Admin Dashboard | None | 0 |
| Total | Logical business tables; excludes infrastructure | 31 |

Courses owns course_schedule_options and one_time_schedule_changes despite their documentation in Schedule Builder. Semester ownership remains an architecture decision. No new business table is presently recommended; infrastructure needs may change this assessment.

## 3. Review results and engineering decisions

### R01 — Requirements and access
- Retain exam conflict checks on exam creation and applicable edits; do not add exam conflict validation to schedule confirmation. This intentionally does not guarantee that later course selections cannot introduce an exam overlap.
- Confirmed weekly schedule totals: 13–21 hours, including zero-credit courses. Calculate totals from selections.
- Retain course access when administrative changes make a previously confirmed schedule incomplete. Persisted course selections, not completeness, determine enrollment; enforce account and role authorization separately. This interpretation was approved during review, not written back to PRDs.

### R02 — Types and nullability
Retain unsigned BIGINT identifiers, matching FK types, VARCHAR(11) student IDs and Egyptian phone numbers, unsigned credit hours and numbering, UTC DATETIME timestamps, and meaningful nullable fields. Use a supported MySQL release that enforces CHECK constraints; verify Laravel migration output rather than assuming timestamps() creates DATETIME. Use explicit dateTime definitions where required. Keep InnoDB and utf8mb4_unicode_ci. Validate lengths, unsigned ranges and strict SQL mode. TIME columns used for clock times require valid local clock ranges, not arbitrary MySQL duration values.

### R03 — Relationships
All FKs use ON UPDATE RESTRICT. Use RESTRICT for shared references requiring coordinated deletion, CASCADE for dependent records and SET NULL for historical references or invalidated schedule options. Replace standalone team_members.team_id FK with approved composite FK (team_id, project_id) -> teams(id, project_id), ON DELETE CASCADE, ON UPDATE RESTRICT. Retain UNIQUE teams(id, project_id) and child INDEX(team_id, project_id); remove redundant standalone team_id index. Retain separate project FK per existing design and explicitly delete memberships before deleting projects. No new columns needed for this change.

### R04 — Constraints
Preserve existing unique keys, XOR parent constraints for educational_files, ai_summaries and content_views, positive numbering/file size/capacity checks, assignment status/timestamp consistency, semester bounds and valid time ranges, and nonempty exam topics array. Laravel validates individual topic strings, schedule option course/type, role restrictions, team leader membership, eligibility and capacity. Unique keys are necessary but not sufficient for concurrency-sensitive invariants.

### R05 — Indexes
Retain documented indexes; avoid duplicate leading-prefix FK indexes when already supported. Review schedule course-to-student lookup, recipient reads and counts, status scheduler scans, history queries and chronological ordering with EXPLAIN on representative data. Contains searches with leading wildcards and NULL-last marks sorting may require scans or sorting. Do not claim measured performance before testing.

### R06 — Atomicity and concurrency
- Serialize competing schedule confirmations and administrative option/credit changes using a consistent lock order; revalidate within the transaction.
- Lock teams for membership capacity and leadership operations. Unique(project_id,user_id) prevents duplicate memberships across teams. Recheck eligibility and capacity at commit time; handle deadlocks with bounded retries.
- Protect exam checks against concurrent inserts even when no matching row exists; use a stable locking target and consistent ordering. Exact lock implementation belongs to detailed architecture.
- Serialize semester start/end and term-scoped writes using a stable lifecycle lock, including when no active semester exists. Preserve one-active-term invariant.
- Assignment deadline edits, confirmation and expiry jobs must use atomic conditional writes or locking and recheck deadline.
- OTP issuance, consumption, attempts and rate limits must be concurrency-safe. Old codes must be unusable; no plaintext OTP storage.
- Required administrative audit writes share the business mutation transaction. Dispatch external delivery only after commit.

### R07 — Ownership and derived data
Schedule Builder owns persisted enrollment. Do not duplicate credit totals, team member counts or notification counters. Historical snapshots in ban_records and audit_logs and the team_members.project_id redundancy are intentional. Admin Dashboard remains a read/query layer, not an independent data owner.

### R08 — Account deletion
Hard delete only eligible accounts; respect the approved ban restriction and resolve conflicting older documents before coding. Identity coordinates cleanup of course views, schedules/history, assignment statuses, memberships/invitations/requests and leadership, notification recipients/preferences/subscriptions and OTPs. Preserve ban/audit snapshots; nullable historical references use SET NULL. All MySQL changes share one transaction. Redis revocation and external side effects need failure recovery; authentication must reject deleted/banned users even if session revocation fails. Prevent concurrent writes for the deleting account.

### R09 — Semester lifecycle
Retain users, lookup records, course definitions, ban history, audit records, notification preferences/subscriptions, and unexpired account/system/announcement notifications. Delete academic content, file/summary metadata, views, schedules and histories, regular options and exceptions, assignments and statuses, exams, projects and teams, and academic notification categories. Delete in dependency-safe order. Preserve ended semester rows pending retention decision. Gate writes and stale jobs by the active semester identity; do not allow queued work from an ended term to recreate operational data. Physical file removal is after commit with retry/reconciliation. Confirm any conflicting reset wording before destructive implementation.

### R10 — Time
System timestamps/deadlines use UTC DATETIME; academic weekly times and occurrence/exam DATE/TIME values represent Africa/Cairo local civil time. Convert scheduled local dates/times to instants for expiry checks; test DST transitions and boundary conditions. This is an engineering convention selected for consistent implementation, not a new functional requirement.

### R11 — Storage and delivery
Keep PDF and AI JSON files outside MySQL. Stage uploads, publish metadata only when valid, and compensate for failed storage or database operations. Delete or invalidate AI summaries when their source file disappears; regenerate only for the current source version. Exact file version/race handling belongs to architecture. Push delivery checks current user status, permission, preference and subscription ownership; remove confirmed invalid subscriptions. Do not expose endpoints or encryption secrets. Use after-commit dispatch and define retries, deduplication and durable recovery; after-commit alone is not a guarantee of delivery.

## 4. Module-specific remaining items

| Module | Finding / next action |
|---|---|
| Identity & Access | Enforce one Owner and one active ban cycle using concurrency-safe services; verify role-dependent academic profile requirements and OTP retention. Older PRD wording about deleting banned accounts and ban reasons conflicts with documented later design decisions. Preserve files unchanged and resolve precedence explicitly before implementation. |
| Courses | Required file on initial creation is application-enforced; independent file deletion must not expose stale AI summaries. Determine whether informational prerequisites and earlier ownership document's course attributes are represented in the authoritative PRD; do not invent schema fields. |
| Schedule Builder | Exact lifecycle owner and ended-semester retention unresolved. Multiple active exceptions for one occurrence and exception history after deleting parent option require PRD-based clarification. Nullable option references are intentional. |
| Assignments | Recommend lazy status creation: derive missing status from current deadline and persist on mutation, use left joins for admin lists. Confirm deadline-extension-to-still-expired behavior against PRD; no late confirmation. Scheduler cannot update missing rows. |
| Exams | Retain schema and conflict scope. Define expiry boundary and course-change notification recipients from existing requirements. No new exam conflict rule for schedules. |
| Projects & Teams | Composite FK cleanup specified in R03. Pending invitation/request handling and exact lock order remain open. Projects audit coverage differs between its design document and later delegated Audit Log PRD; do not silently expand scope. |
| Notifications | Ensure promoted/demoted users have correct preference lifecycle; verify subscription re-registration, shared-browser ownership, queue retries and recipient snapshot consistency. Account/system event triggers not fully specified; do not add events arbitrarily. |
| Audit Log | Preserve append-only behavior apart from retention and FK nullification. Retain 365-day policy. Reconcile exact audit coverage with each PRD; allowlist snapshots and exclude credentials, OTPs, tokens and push secrets. |
| Admin Dashboard | Query existing module data. Older dashboard references project deadlines, absent from current project design; no new project deadline field authorized. |

## 5. Final decision register

- KEEP: 31-table logical baseline, existing PRDs, approved key types, constraints, retention and module boundaries.
- CHANGE AT MIGRATION DESIGN: remove redundant team_members standalone team FK/index in favor of approved composite relationship; explicitly declare DATETIME timestamps and nonduplicated supporting indexes.
- IMPLEMENT: atomic mutations, lock ordering, authorization, safe file lifecycle and after-commit delivery with recovery.
- DEFER: exact technical mechanisms and genuine ambiguities listed above. No PRD modifications authorized by this review.

## 6. Verification gates

Before release: reconcile authoritative PRD versions and conflicting requirements; generate consolidated ERD and migration dependency order; run migrations and rollbacks on the target MySQL version; test FK deletion paths, CHECK and UNIQUE violations, concurrent writes, account deletion rollback, term reset and stale jobs, timezone boundaries, authorization, retention and file/queue failures; inspect EXPLAIN plans using representative data.

## 7. Outcome and next phase

Logical schema review documented. No runtime database, migration tests, benchmark or full line-by-line comparison of every PRD was performed. This is not a claim of full verification. Preserve the logical baseline; carry unresolved requirement conflicts into a decision register before affected implementation. Next: consolidated ERD and dependency map, then detailed Laravel modular architecture, API design, security design and implementation planning.
