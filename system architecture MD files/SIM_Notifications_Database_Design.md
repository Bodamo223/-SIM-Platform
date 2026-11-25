# SIM Platform – Notifications Database Design

Status: Logical schema approved; implementation details pending.

## Standards
MySQL InnoDB, utf8mb4, utf8mb4_unicode_ci. Plural snake_case tables and snake_case columns. All primary keys are BIGINT UNSIGNED AUTO_INCREMENT. Foreign keys match referenced types. DATETIME values are UTC; display in Africa/Cairo. Use Laravel migrations. All foreign keys use ON UPDATE RESTRICT.

## 1. notifications

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT |
| message | TEXT | NOT NULL |
| category | ENUM | NOT NULL |
| type | ENUM | NOT NULL |
| created_by | BIGINT UNSIGNED | NULL, FK users.id |
| created_at | DATETIME | NOT NULL |

Category values: `courses`, `schedule`, `assignments`, `exams`, `projects`, `announcements`, `account`, `system`.
Type values: `manual`, `automatic`.
Manual notifications require an authorized Admin or Owner sender at creation, validated in Laravel. Automatic notifications have NULL created_by. A deleted sender also results in NULL. Notifications are immutable; no updated_at, title or data column. Navigation uses category rather than stored item links.

Indexes: PRIMARY KEY (id), INDEX (created_by), INDEX (created_at), INDEX (category, created_at).

## 2. notification_recipients

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT |
| notification_id | BIGINT UNSIGNED | NOT NULL, FK notifications.id |
| user_id | BIGINT UNSIGNED | NOT NULL, FK users.id |
| read_at | DATETIME | NULL DEFAULT NULL |
| deleted_at | DATETIME | NULL DEFAULT NULL |

One row per notification and recipient. NULL read_at means unread. Non-NULL deleted_at hides the notification from that student while preserving admin statistics. No created_at or updated_at; notification creation time is obtained from notifications. Recipient membership is determined at creation.

Indexes: PRIMARY KEY (id), UNIQUE (notification_id, user_id), INDEX (user_id, deleted_at, read_at).

## 3. notification_preferences

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT |
| user_id | BIGINT UNSIGNED | NOT NULL, UNIQUE, FK users.id |
| push_enabled | BOOLEAN | NOT NULL DEFAULT true |
| created_at | DATETIME | NOT NULL |
| updated_at | DATETIME | NOT NULL |

One record per student, created in the account creation transaction. The preference applies across devices and does not override browser permissions or affect in-app delivery.

Indexes: PRIMARY KEY (id), UNIQUE (user_id).

## 4. push_subscriptions

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT UNSIGNED | PK, AUTO_INCREMENT |
| user_id | BIGINT UNSIGNED | NOT NULL, FK users.id |
| endpoint | TEXT | NOT NULL |
| endpoint_hash | CHAR(64) | NOT NULL, UNIQUE |
| p256dh | VARCHAR(255) | NOT NULL |
| auth | VARCHAR(255) | NOT NULL |
| expiration_time | DATETIME | NULL DEFAULT NULL |
| created_at | DATETIME | NOT NULL |
| updated_at | DATETIME | NOT NULL |

Store the original endpoint for delivery and its SHA-256 hexadecimal hash for uniqueness and lookup. Laravel must compute the hash from the exact endpoint. p256dh and auth come from the browser and support Web Push encryption. Expiration is optional because browsers may not supply it. A student can have multiple subscriptions.

Indexes: PRIMARY KEY (id), UNIQUE (endpoint_hash), INDEX (user_id).

## 5. Relationships and foreign keys

| Foreign key | Referenced key | ON DELETE | ON UPDATE |
|---|---|---|---|
| notifications.created_by | users.id | SET NULL | RESTRICT |
| notification_recipients.notification_id | notifications.id | CASCADE | RESTRICT |
| notification_recipients.user_id | users.id | RESTRICT | RESTRICT |
| notification_preferences.user_id | users.id | RESTRICT | RESTRICT |
| push_subscriptions.user_id | users.id | RESTRICT | RESTRICT |

Notifications to recipients: one-to-many. Users to recipients and subscriptions: one-to-many. Users to preferences: one-to-zero-or-one at database level; one per student enforced by account creation workflow. Users to manually created notifications: one-to-many, optional sender reference.

## 6. Integrity and queries
Unique constraints prevent duplicate recipients, preference records and endpoint hashes. No additional CHECK constraints were approved. Student queries exclude deleted_at IS NOT NULL and restrict user_id to the authenticated student. Admin recipient statistics include student-deleted records. Read status derives from read_at. The five counters derive from recipient records joined to notifications; no stored counters. Use notification created_at for date filters and ordering.

## 7. Lifecycle and cleanup
Laravel Scheduler deletes notifications at 30 days from creation. Foreign-key cascade deletes recipient records. Term reset deletes notifications with categories courses, schedule, assignments, exams and projects; other categories retain normal expiry. Preferences and subscriptions survive term reset.

Account deletion explicitly removes recipient, preference and subscription records within the cross-module database transaction before deleting the user. Disable push without deleting subscriptions; resume for valid subscriptions when enabled. Delete confirmed invalid or expired subscriptions. Stop push for banned accounts. Logout from all devices deletes all subscriptions for the account.

## 8. Implementation boundaries
Notifications owns all four tables. Other modules publish relevant events; Notifications handles recipient selection, persistence and delivery. Laravel Reverb provides in-app real-time updates; Laravel Queues handles background push delivery. PWA, Service Worker, Web Push API and VAPID support browser delivery.

Exact transaction boundaries for notification creation, queue retry/deduplication, subscription ownership on shared browsers, and race handling remain implementation decisions. No additional columns or indexes are approved for these concerns. Protect endpoint and auth values from unauthorized exposure. The schema has not been executed or migration-tested.
