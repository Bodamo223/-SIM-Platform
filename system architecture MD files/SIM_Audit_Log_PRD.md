# SIM Platform — Administrative Audit Log PRD

Status: Design completed under Owner delegation; implementation and cross-module review pending.

## Purpose and access
Provide Admin and Owner with a shared history of successful administrative mutations. Students cannot access it. Neither Admin nor Owner may edit or manually delete records. Viewing, searching, filtering, student self-service, application errors and security telemetry are outside scope.

## Recorded operations
| Module | Operations |
|---|---|
| Identity & Access | Administrative account creation, update, deletion, ban, unban, promotion, demotion; academic level and group creation, update and deletion |
| Courses | Course creation and update; lecture, section and educational file creation, update/replacement and deletion; schedule option and one-time change management |
| Schedule Builder | Semester start and end; administrative schedule-related mutations not already recorded by Courses |
| Assignments | Create, update, delete |
| Exams | Create, update, delete |
| Projects & Teams | Project create, update, delete; administrative leader changes |
| Notifications | Manual send and administrative deletion |

Record each operation once in its owning module. Do not introduce new administrative permissions. Projects & Teams and other previously unspecified audit coverage are decisions made under the Owner's delegation and must be reflected in the respective PRDs during integration review. Automated cleanup and student actions are excluded.

## Record contents
Actor ID, name, Student ID and role snapshots; module, action, entity type and ID, optional entity name snapshot, old/new values, UTC timestamp. Preserve history after actor or entity deletion. Never record passwords, OTPs, session tokens, push credentials or other secrets. Use allowlisted fields; record only relevant values.

## Administration
List newest first (created_at DESC, id DESC), paginate, filter by actor, module, action and date range. Show historical actor and entity details and old/new values. Admin Dashboard Recent Activity is a filtered projection of the same records; exclude bans and unbans there, not from the full audit log. No export requirement.

## Retention
Retain for 365 days from creation; Laravel Scheduler deletes expired records. Semester reset does not delete audit history. No manual deletion or editing. Retention is a delegated design decision, not a legal compliance determination.

## Consistency
Write audit records in the same MySQL transaction as the successful mutation. Roll back both if either fails. Failed attempts belong to separate operational/security logs. Record semester end with the semester ID and summary rather than duplicating every automated cleanup deletion.

## Acceptance criteria
- Authorized administrators can view and filter records; students cannot.
- Successful covered mutations generate records with correct snapshots; rolled-back mutations do not.
- Creation records have new values, deletion records old values, and updates have relevant before/after values.
- Actor/entity deletion and semester reset preserve unexpired history.
- Expired records are removed automatically; no manual mutation endpoint exists.
- Secrets are excluded from snapshots.

## Remaining implementation review
Confirm module ownership and event mappings against all module PRDs, including Projects & Teams; define exact snapshot allowlists, pagination size, cleanup batch size and authorization tests. No migrations or runtime tests have been executed.
