# SIM Platform – Data Ownership

## 1. Purpose

This document defines which module owns each category of data, which module is responsible for managing it, and how other modules access or interact with that data.

The owning module is the single source of truth for its data.

Other modules must use defined communication mechanisms rather than directly modifying another module's data.

## 2. Core Business Modules

### 2.1 Identity & Access

Owns:
- User accounts.
- User credentials.
- User profiles.
- Roles and permissions.
- Student academic profile attributes.
- OTP records.
- Account status.
- Ban and Unban records.
- Session-related data.
- Online Presence data.

Responsibilities:
- Registration and authentication.
- Profile updates.
- Password changes and recovery.
- Email and phone verification.
- Role management.
- Account banning and unbanning.
- Account deletion.
- Session revocation.
- Online status calculation.

Other modules store user identifiers when needed and request user information through Identity & Access.

### 2.2 Courses

Owns:
- Course definitions.
- Course codes.
- Credit hours.
- Academic Level associations.
- Semester associations.
- Course types.
- Prerequisite relationships.
- Lecture and Section definitions.
- Lecture and Section schedule options.
- Educational content metadata.
- AI-generated summaries.
- Content viewing statistics and history.

Responsibilities:
- Course management.
- Lecture and Section management.
- Schedule option management.
- Educational content management.
- AI Summary Generation.
- Course analytics.

Course definitions remain available across terms.

Term-specific educational content and operational data are removed during term reset, while the stable course information remains.

### 2.3 User Schedule

Owns:
- Student course selections.
- Selected Lecture Option IDs.
- Selected Section Option IDs.
- Schedule status.
- Approved schedule data.
- Total selected credit hours.

Responsibilities:
- Managing student schedules.
- Validating course selections.
- Detecting schedule conflicts.
- Enforcing credit-hour limits.
- Approving schedule changes.
- Processing schedule changes received from Courses.

User Schedule is the single source of truth for student course selections.

It stores identifiers for schedule options owned by Courses and retrieves their details through the Courses interface.

### 2.4 Assignments

Owns:
- Assignment records.
- Assignment titles and descriptions.
- Assignment deadlines.
- Student submission status records.

Responsibilities:
- Assignment creation and modification.
- Deadline management.
- Submission status management.
- Not Submitted, Submitted, and Missed state transitions.

Assignments are associated with the relevant course offering and term.

### 2.5 Exams

Owns:
- Exam records.
- Exam types.
- Exam dates and times.
- Exam locations.
- Exam descriptions.

Responsibilities:
- Exam creation and modification.
- Exam deletion.
- Exam conflict validation.
- Exam-related business rules.

Exam records belong to their relevant term.

### 2.6 Projects & Teams

Owns:
- Project records.
- Team records.
- Team memberships.
- Team leader assignments.
- Invitations.
- Join Requests.
- Team capacity configuration.

Responsibilities:
- Project management.
- Team creation and deletion.
- Membership management.
- Invitation management.
- Join Request management.
- Automatic leader reassignment.
- Automatic team deletion when its last member leaves.

When a team leader leaves:
- The system automatically selects a new leader.
- Pending invitations and Join Requests remain associated with the team.
- Their management responsibility transfers to the new leader.

When the last member leaves:
- The team is deleted.
- Its memberships, invitations, and Join Requests are deleted.

### 2.7 Notifications

Owns:
- Notification records.
- Notification recipients.
- Read and Unread statuses.
- Push notification preferences.
- Push device subscriptions.

Responsibilities:
- Creating and managing notifications.
- Managing recipient-specific read statuses.
- Sending Push Notifications.
- Deleting notifications after 30 days.
- Handling notification deletion requested by administrators or users.

When a student account is deleted:
- The student's notifications are deleted.
- Their read statuses are deleted.
- Their push subscriptions are deleted.
- Other students' notification data remains unaffected.

### 2.8 Audit Log

Owns:
- Administrative activity records.
- Actor information snapshots.
- Action types.
- Affected entity identifiers and types.
- Timestamps.
- Old and new values.
- Entity snapshots when required for deletion records.

Responsibilities:
- Receiving administrative activity records.
- Storing historical administrative operations.
- Preserving records independently of the lifetime of the original entity.

Audit Log records remain available after:
- Deleting a course.
- Deleting an assignment.
- Deleting an exam.
- Deleting a student account.
- Deleting or changing an administrator account.

Projects & Teams currently has no approved Audit Log requirements.

## 3. Presentation / Query Layer

### Admin Dashboard

Does not own business data.

It retrieves information from the owning modules and presents:
- Platform statistics.
- Recent administrative activity.
- Attention Required items.
- Upcoming events.
- Global search results.
- Administrative management views.

## 4. Shared Infrastructure Ownership

Shared infrastructure owns technical responsibilities rather than business data.

### 4.1 File Storage
Owns the technical storage and retrieval operations for uploaded files.

The owning business module determines when files should be uploaded, replaced, or deleted.

File metadata remains managed by the relevant business module.

### 4.2 Email Delivery
Handles email delivery requests.

Identity & Access owns OTP rules, verification processes, and email-related business logic.

### 4.3 Queue Management
Manages asynchronous jobs, workers, retries, and failed jobs.

Business modules own the logic executed by their jobs.

### 4.4 Scheduled Tasks
Runs scheduled operations.

Examples:
- Notification expiration.
- Periodic cleanup.
- Other scheduled maintenance operations.

### 4.5 Logging
Owns technical logging operations.

Approved logging use cases:
- System Errors.
- API Errors.
- Queue Job execution.
- Email Delivery failures.
- AI API failures.
- Slow Database Queries.
- Authentication-related technical and security events.
- Scheduled Task execution.
- Performance monitoring.

Sensitive information such as passwords and OTP values must not be logged.

Logging is separate from Audit Log.

### 4.6 Exception Handling
Provides centralized exception handling and consistent error responses.

It handles:
- Exception capture.
- Error classification.
- HTTP response formatting.
- Safe user-facing error messages.
- Technical error logging.

### 4.7 Redis / Caching
Provides:
- Application caching.
- Queue backend support.
- Session storage when configured.
- Online Presence temporary data.
- Rate Limiting.
- Distributed Locks when required.

Each business module manages its own cached data through Laravel Cache APIs.

No independent Cache Management module is required.

### 4.8 Real-Time Communication
Uses Laravel Reverb to deliver real-time events to connected clients.

Business modules determine which events should be broadcast and to whom.

Real-Time Communication handles delivery without owning the underlying business logic or primary records.

## 5. Cross-Module Data Access Rules

1. A module is the sole owner of its business data.
2. Other modules must not directly modify its tables.
3. Immediate information requests use Service Calls.
4. Notifications of completed business actions use Domain Events.
5. Each module remains responsible for its own data deletion.
6. Cross-module deletion must be coordinated to prevent orphaned records.
7. Shared infrastructure must not become the owner of business rules.

## 6. Term Reset Policy

### 6.1 General Rule

Every new term starts with fresh operational data.

The Owner manually initiates the term reset through the administrative interface.

### 6.2 Data That Remains

Stable course definitions remain available, including:
- Course Name.
- Course Code.
- Credit Hours.
- Academic Level.
- Semester association.
- Course Type.
- Prerequisites.

### 6.3 Data That Is Deleted

The reset removes term-specific operational data, including:
- Lectures and Sections.
- Lecture and Section schedule options.
- Educational content files.
- AI summaries.
- Content viewing statistics and history.
- Assignments and submission statuses.
- Exams.
- Projects and Teams.
- Team memberships.
- Invitations and Join Requests.
- Student schedules and course selections.
- Term-related notifications.

### 6.4 Audit Log Preservation

Audit Log records are not deleted during term reset.

Historical administrative records remain available even when the original entities are deleted.

### 6.5 Reset Coordination

Each owning module is responsible for deleting its own term-specific data.

The reset process must coordinate deletion across modules and ensure successful completion before the new term begins.

Detailed transaction handling, failure recovery, and execution order will be defined in Integration & Transactions.

## 7. Account Deletion Ownership

Identity & Access coordinates account deletion.

Each module deletes its own user-related records.

Examples:
- User Schedule deletes the student's schedules and selections.
- Assignments deletes the student's submission statuses.
- Projects & Teams deletes memberships and related invitations or requests.
- Notifications deletes the student's notifications, read statuses, and push subscriptions.

Audit Log preserves historical administrative records.

Detailed transaction coordination will be defined in Integration & Transactions.

## 8. Important Architectural Distinctions

### Course Definition vs. Term Data

Stable course information remains across terms.

Operational content and data associated with the completed term are removed during reset.

### Logging vs. Audit Log

Logging records technical events and system behavior.

Audit Log records administrative actions and preserves their history.

### Business Data vs. Infrastructure

Business modules own domain data and rules.

Shared infrastructure provides technical capabilities used by those modules.

## 9. Pending Design Details

The following matters will be addressed in later architecture stages:

- Exact database tables and foreign keys.
- Cross-module service interfaces.
- Domain event definitions and payloads.
- Term reset execution order and recovery strategy.
- Account deletion transaction strategy.
- File storage implementation details.
- Queue configuration and retry policies.
- Redis key organization and expiration.
- Reverb broadcasting channels and authorization.
- Logging retention and monitoring configuration.
