# SIM Platform — Administrative Audit Log Database Design

Status: Design completed under Owner delegation; integration review and migration testing pending.

## Standards
MySQL InnoDB, utf8mb4/utf8mb4_unicode_ci, snake_case, BIGINT UNSIGNED identifiers, UTC DATETIME, Africa/Cairo display, Laravel migrations. One module-owned table: audit_logs.

## Schema
| Column | Type | Constraints |
|---|---|---|
| id | BIGINT UNSIGNED | PRIMARY KEY AUTO_INCREMENT |
| actor_id | BIGINT UNSIGNED | NULL, FK users.id |
| actor_name | VARCHAR(255) | NOT NULL |
| actor_student_id | VARCHAR(11) | NOT NULL |
| actor_role | ENUM('admin','owner') | NOT NULL |
| module | ENUM('identity_access','courses','schedule_builder','assignments','exams','projects_teams','notifications') | NOT NULL |
| action | ENUM('create','update','delete','ban','unban','promote','demote','send','start','end','revoke') | NOT NULL |
| entity_type | VARCHAR(100) | NOT NULL |
| entity_id | BIGINT UNSIGNED | NOT NULL |
| entity_name | VARCHAR(255) | NULL |
| old_values | JSON | NULL |
| new_values | JSON | NULL |
| created_at | DATETIME | NOT NULL |

No updated_at or deleted_at. No arbitrary defaults. actor_id is required at creation by application validation and becomes NULL when the actor is deleted. Actor snapshots remain unchanged. entity_type uses a stable application allowlist, not PHP class names. entity_id is a historical polymorphic identifier, not a foreign key. No uniqueness constraint on actor/entity/action because repeated legitimate operations are possible.

## Relationships
users (1) to audit_logs (N), optional historical actor reference. FK actor_id references users(id) ON DELETE SET NULL ON UPDATE RESTRICT. No entity FK; deleting a target cannot cascade into audit history.

## Indexes
- PRIMARY KEY (id)
- INDEX (actor_id, created_at)
- INDEX (module, created_at)
- INDEX (action, created_at)
- INDEX (entity_type, entity_id)
- INDEX (created_at)

The actor composite index supports the FK; no redundant standalone actor index. Default ordering created_at DESC, id DESC. Validate query plans with production-like data before adding more indexes.

## Validation and snapshots
Application validates module/action/entity combinations. JSON values are objects when present. Create: old_values NULL, new_values object. Delete: old_values object, new_values NULL. Update and state transitions: relevant before/after values; send/start/end/revoke use applicable snapshots. Use explicit allowlists and omit secrets, credentials and unnecessary personal data. Store historical names and roles at action time, not by later joins. Entity names may be NULL for entities without names. Actor ID is nullable solely to preserve history after deletion.

## Transactions and lifecycle
The owning module calls a central Audit Log service within the same MySQL transaction as the mutation. No asynchronous audit write for required records. Rollback removes both mutation and audit record. Avoid duplicate logs from cross-module listeners. No manual update/delete API; scheduled retention deletion and FK actor nullification are permitted lifecycle exceptions. Delete records older than or equal to the 365-day cutoff in batches; do not purge on semester reset. Account deletion preserves actor snapshots. Admin Dashboard reads a filtered projection, not another table.

## Review and testing
Test role access, snapshot correctness, transaction rollback, repeated operations, actor deletion, entity deletion, semester reset, retention cutoff and secret exclusion. Exact action mappings and snapshot allowlists must be reconciled with each module PRD. Retention, expanded audit scope, enum values and index selection are delegated design decisions. No migrations have been executed or tested.
