# SIM Platform – Identity & Access PRD

PRD ID: 84271
Module: Identity & Access
Status: Requirements Analysis Completed & Updated
Version: Final Updated Draft

⸻

## 1. Overview

The Identity & Access module manages user accounts, registration, authentication, profile management, account security, roles, permissions, account deletion, administrative account management, and online presence.

The platform supports three roles:

* Student
* Admin
* Owner

Identity & Access owns its identity-related data. Other modules own their respective data and are responsible for cleaning up that data when an account is deleted.

## 2. User Roles

### 2.1 Student

Students can:

* Register their own accounts.
* Log in and log out.
* Manage permitted profile information.
* Change their passwords.
* Recover forgotten passwords.
* Delete their accounts, subject to account-status restrictions.
* Use platform features according to their permissions.

### 2.2 Admin

Admins can:

* Create student accounts.
* View, search, and filter students.
* Edit all student information except passwords, subject to account-status restrictions.
* Delete student accounts, subject to the ban deletion restriction.
* Ban and unban students.
* View student online status.
* Manage Academic Levels and Groups.
* Perform administrative operations across the platform according to their permissions.

Admins cannot:

* Promote users to Admin.
* Demote Admin users.
* Create or modify Owner accounts.
* Change student passwords.
* Create another Owner.

### 2.3 Owner

The Owner has all Admin permissions, plus:

* Promote Students to Admin.
* Demote Admins to Students.
* Manage Admin roles.

There is only one Owner account.

No Admin or Student can create another Owner or promote themselves to Owner.

Owner initialization takes place during the initial system setup.

## 3. Student Registration

### 3.1 Registration Fields

The registration form requires:

## 1. Name
## 2. Student ID
## 3. Personal Email
## 4. Phone Number
## 5. Academic Level
## 6. Group
## 7. Password

### 3.2 Registration Rules

* Students can register independently at any time.
* New self-registered accounts receive the Student role automatically.
* Student ID must be unique across all existing accounts.
* Email must be unique across all existing accounts.
* Phone Number must be globally unique across all existing accounts.
* Identifiers belonging to existing banned accounts cannot be assigned to another account.
* Phone Number accepts Egyptian local mobile numbers consisting of exactly 11 digits.
* Phone Number validation must check the expected format and mobile prefixes.
* Leading zeros must be preserved.
* Academic Level and Group must be selected from options configured by Admin or Owner.
* The platform does not connect to the university’s database.
* The platform does not automatically verify Student IDs.

### 3.3 Self-Registration Activation

* A newly registered account requires email verification.
* The system sends an OTP to the submitted email.
* The account becomes Active immediately after successful verification.
* Student ID verification is not a separate account state.

## 4. Administrative Account Creation

Admin and Owner can create student accounts manually.

Rules:

* Every manually created account receives the Student role by default.
* Admin or Owner can specify the student’s account information and password.
* Manually created accounts become Active immediately.
* Email OTP verification is not required during administrative creation.
* The student can subsequently change their password through the Change Password page.
* Student ID, Email, and Phone Number must satisfy the same uniqueness constraints applied to self-registration.
* Identifiers belonging to existing banned accounts cannot be assigned to another account.

## 5. Account Status

The platform maintains two account statuses.

Active

The account can authenticate and access authorized platform services.

Banned

The account cannot log in or access protected services.

Online Presence is separate from Account Status.

## 6. Authentication

### 6.1 Login

Users authenticate using:

* Email
* Password

The system must verify account status before allowing access.

Banned accounts cannot authenticate.

### 6.2 Persistent Authentication

* After the first successful login, the system creates an authentication session using Cookies.
* When the user revisits the platform, the system restores the existing session automatically if it remains valid.
* The user does not need to log in again while the session remains valid.
* Cookie and session expiration policies are defined during Architecture Design.

### 6.3 Logout

When a user logs out:

* All active sessions associated with the account must be revoked.
* The user is logged out from all devices.
* The user must authenticate again to access the platform.

### 6.4 Password Changes and Existing Sessions

* Changing a password does not automatically terminate existing sessions on other devices.
* No email notification is sent after a successful password change.

### 6.5 Banned Accounts

When an account is banned:

* All active sessions must be revoked immediately.
* Existing authentication Cookies and sessions must no longer grant access.
* The user cannot log in until the ban is removed.

## 7. Email OTP System

The same OTP mechanism is used for:

* Initial email verification.
* Password recovery.
* Email changes.
* Phone number changes.

### 7.1 OTP Specifications

* OTP consists of six numeric digits.
* OTP validity is five minutes.
* A new OTP can be requested after 60 seconds.
* A maximum of five incorrect attempts is allowed per OTP.
* After five incorrect attempts, the OTP becomes invalid.
* The user must request a new OTP.
* A maximum of five OTP sends is allowed per account, per purpose, within one hour.
* Sending a new OTP immediately invalidates the previous OTP for the same operation.

### 7.2 OTP Security

* OTP reuse after successful verification must be prevented.
* OTP storage must use secure hashing.
* Rate limiting and abuse prevention mechanisms must be implemented.
* OTP records must be cleaned up according to the approved retention policy of 24 hours.

## 8. Password Management

### 8.1 Change Password

A dedicated Change Password page is required.

Fields:

* Current Password
* New Password
* Confirm New Password

The system must verify the current password before accepting the new one.

### 8.2 Password Requirements

New passwords must satisfy all of the following:

* Minimum 8 characters.
* At least one uppercase letter.
* At least one lowercase letter.
* At least one numeric digit.
* At least one special character.

### 8.3 Forgot Password

* The user submits their registered email.
* The system sends an OTP.
* The user verifies the OTP.
* The user can then set a new password.
* The same OTP specifications and restrictions apply.
* Banned accounts cannot bypass their ban through password recovery.

Admins and Owners cannot directly change student passwords.

## 9. Profile Management

### 9.1 Student Permissions

Students can edit:

* Name
* Student ID
* Email
* Phone Number

Students cannot edit:

* Academic Level
* Group

Academic Level and Group can only be changed by Admin or Owner.

Banned students cannot modify their profile information because they cannot authenticate or access protected services.

### 9.2 Email Changes

* The student requests an email change.
* The system sends an OTP to the currently registered email.
* The old email remains the primary email until verification succeeds.
* The new email must be unique across all existing accounts.
* An email belonging to an existing banned account cannot be assigned to another account.
* The new email becomes primary after successful OTP verification.

### 9.3 Phone Number Changes

* The student requests a phone number change.
* The system sends an OTP to the currently registered email.
* The new phone number becomes effective only after successful OTP verification.
* The new phone number must be globally unique across all existing accounts.
* A phone number belonging to an existing banned account cannot be assigned to another account.
* The Egyptian mobile number format and prefix validation rules apply.

### 9.4 Administrative Profile Editing

Admin and Owner can directly edit all student profile fields except Password, subject to the following restrictions:

* Name, Academic Level, and Group can be edited for Active or Banned accounts.
* Student ID, Email, and Phone Number can only be edited while the account is Active.
* Student ID, Email, and Phone Number cannot be modified while the account is Banned.
* Administrative changes do not require OTP.
* All uniqueness constraints must be enforced.

## 10. Academic Levels and Groups Management

* Admin and Owner can create and manage the Academic Level options available during student registration and profile administration.
* Admin and Owner can create and manage the Group options available during student registration and profile administration.
* Students select an Academic Level and Group from the configured options.
* Students cannot create or manage these options themselves.
* Academic Levels and Groups are independent entities.
* A Group is not structurally tied to a specific Academic Level.
* Admin and Owner can edit the name of an Academic Level or Group even when student accounts are associated with it.
* An Academic Level or Group cannot be deleted while student accounts reference it.
* The database must prevent deletion of referenced options through the approved Foreign Key deletion rules.

## 11. Account Deletion

### 11.1 Student-Initiated Deletion

Students can permanently delete their accounts from the Profile page, subject to the following rules:

* The student must be authenticated.
* The student must provide their current password.
* Successful password verification is required before deletion.
* No additional OTP confirmation is required.
* Deletion is permanent and cannot be reversed.
* A banned student cannot initiate account deletion because banned accounts cannot authenticate or access protected services.
* A banned account must be unbanned before it can be deleted.

### 11.2 Administrative Deletion

Admin and Owner can permanently delete student accounts, subject to the following rules:

* No additional Student ID confirmation is required.
* An Admin or Owner cannot delete a currently banned account.
* The account must first be unbanned through an authorized unban operation.
* After successful unban, the account can be deleted through the normal administrative deletion process.

### 11.3 Deletion Scope

Deleting an account permanently removes:

* Student account information.
* Schedule selections and related schedule records.
* Assignment submission status records.
* Team memberships.
* Invitations and join requests associated with the student.
* Personal notifications.
* Notification read statuses associated with the student.
* Push notification subscriptions.
* Content viewing history.

Each module is responsible for deleting the data it owns. Identity & Access coordinates the account-deletion process and the required cleanup across dependent modules.

A banned account cannot be deleted while it remains banned. Since the account remains in the users table throughout the ban period, its Student ID, Email, and Phone Number remain assigned to that account and cannot be reused by another account.

### 11.4 Projects and Teams Cleanup During Account Deletion

When account deletion removes a student from a team:

* If the deleted student is the team leader and other members remain, a new team leader must be selected automatically.
* Existing invitations and join requests must be transferred to the newly selected team leader.
* If no team members remain, the team must be deleted automatically, together with its related invitations and join requests.
* Projects & Teams owns and performs these changes to its data.

### 11.5 Re-registration and Identifier Availability

After permanent deletion of an account, its following identifiers become available for reuse:

* Student ID
* Email
* Phone Number

A banned account must be unbanned before deletion.

While the account remains banned, its identifiers remain assigned to the existing account and cannot be reused by another account.

After successful unban, the account becomes Active and its identifiers remain assigned to it until they are changed or the account is deleted.

Following successful account deletion, its identifiers become available for reuse, subject to normal uniqueness constraints.

### 11.6 Deletion Audit

* Administrative account deletions are recorded in the Administrative Audit Log.
* Student-initiated account deletions are not recorded in the Administrative Audit Log.
* Administrative deletion events must preserve the responsible administrator’s historical identity information.

## 12. Account Ban Management

### 12.1 Permissions

Admin and Owner can ban and unban students.

### 12.2 Ban Rules

* A ban remains permanent until manually removed.
* No automatic expiration exists.
* Banned users cannot log in.
* All active sessions are revoked immediately.
* Student ID, Email, and Phone Number remain assigned to the existing banned account.
* These identifiers cannot be assigned to another account while the banned account exists.
* Student ID, Email, and Phone Number cannot be modified while the account is Banned.
* A banned account cannot be permanently deleted before it is unbanned.
* Admin and Owner must unban the account before performing account deletion.

### 12.3 Unban

* Admin and Owner can unban students.
* Unban can be performed directly from the Student Management page while the account exists.
* The account becomes Active again.
* The user can log in normally.
* Unban removes the banned status and allows normal account operations.
* The account’s identifiers remain assigned to it until they are changed or the account is deleted.
* If the account is subsequently deleted, its identifiers become available for reuse subject to normal uniqueness rules.

### 12.4 Ban Records and History

The system must preserve every Ban and Unban cycle, not only the latest status.

Each ban cycle maintains:

* The associated student account reference, when the account still exists.
* Student Name snapshot at the time of the ban.
* Student ID snapshot at the time of the ban.
* Responsible administrator reference, when the administrator account still exists.
* Administrator Name snapshot at the time of the ban.
* Administrator Student ID snapshot at the time of the ban.
* Ban date and time.
* Unban date and time, if unbanned.
* Unbanning administrator reference, when the administrator account still exists.
* Unbanning administrator Name snapshot.
* Unbanning administrator Student ID snapshot.

Historical rules:

* Every Ban/Unban cycle has its own record.
* Historical records must remain available after later Ban/Unban cycles.
* Historical records must remain available after deletion of the associated student account.
* Historical administrator names and Student IDs must remain available even if an administrator later changes their profile or their account is deleted.
* The database references to deleted accounts may become NULL while historical snapshots remain preserved.
* Ban History does not store Student Email or Phone Number snapshots.
* Ban Reason is not part of the approved Ban History design.

### 12.5 Ban History Search

Admin and Owner can search Ban History using:

* Student Name
* Student ID

The search must work against the historical student information stored in the Ban History records, including records whose associated user account no longer exists.

Ban and Unban operations must also appear in the Administrative Audit Log.

## 13. Online Status

Online Status is separate from Account Status.

It is visible only to Admin and Owner.

Statuses

Online

The student is currently connected to the platform.

Offline

The student is not currently connected, and fewer than seven days have passed since their last visit.

Away

Seven days or more have passed since the student’s last visit.

Last Seen

Admin and Owner can view:

* Last online date.
* Last online time.

The status is determined from connection and activity information rather than manually managed account data.

The technical mechanism for detecting active connections and disconnections is defined during Architecture Design.

## 14. Student Management

### 14.1 Search

Admin and Owner can search students using one search field across:

* Name
* Student ID
* Phone Number

Email is not included in the search fields.

Search uses Contains matching.

### 14.2 Filters

The Student Management page supports:

* Academic Level
* Group
* Account Status
* Online Status

Search operates together with the selected filters and pagination.

### 14.3 Sorting and Pagination

* Default sorting uses created_at DESC, followed by id DESC.
* Page size is 20 records.
* Search, filters, sorting, and pagination state are represented in URL parameters.

### 14.4 Student Information

Admin and Owner can view student information and manage permitted account operations.

Online Status is available only in administrative interfaces and is separate from Account Status.

## 15. Administrative Role Management

* Admin and Owner can create student accounts.
* All newly created accounts receive the Student role.
* Only Owner can promote a Student to Admin.
* Only Owner can demote an Admin to Student.
* Admin cannot change another user’s role.
* No one can create another Owner.
* Role changes must be recorded in the Administrative Audit Log.

## 16. Administrative Audit Log

The system records administrative account operations.

### 16.1 Recorded Operations

* Student account creation.
* Student profile updates.
* Student account deletion.
* Student ban.
* Student unban.
* Role promotion.
* Role demotion.

### 16.2 Audit Information

Each record should include:

* Administrator identity.
* Administrator’s name as it was at the time of the operation.
* Administrator’s Student ID as it was at the time of the operation, when applicable.
* Action type.
* Affected account or entity.
* Date and time.
* Old values and new values when applicable.

The historical name and Student ID of the administrator who performed an action must remain available in the audit record even if that administrator later changes their profile information or their account is deleted.

Administrative Audit Logs are accessible through the Admin Dashboard.

They are not displayed within individual student profile pages.

Student-initiated profile changes and account deletions are not included in the Administrative Audit Log unless explicitly covered by another module’s requirements.

Audit Log retention duration remains to be defined during Architecture Design.

## 17. Business Rules Summary

1. Student ID must be unique across all existing accounts.
2. Email must be unique across all existing accounts.
3. Phone Number must be globally unique across all existing accounts.
4. Every newly created account receives the Student role by default.
5. Self-registration requires successful email OTP verification.
6. Administrative account creation activates the account immediately without OTP.
7. Only Owner can manage Admin roles.
8. Admin and Owner can edit student profile information except passwords, subject to account-status restrictions.
9. Only students manage their own passwords.
10. Account deletion is permanent.
11. A banned account cannot be deleted until it has been unbanned.
12. Unban changes the account status to Active and allows normal account operations.
13. Banned users cannot authenticate or access protected services. Their identifiers remain assigned to their existing accounts until those accounts are deleted.
14. Manual logout revokes all sessions across all devices.
15. Ban immediately revokes all sessions.
16. Account Status and Online Status are independent.
17. Online Status is visible only to Admin and Owner.
18. Academic Level and Group are administratively managed options.
19. Academic Level and Group names can be modified while accounts reference them.
20. Academic Level and Group cannot be deleted while accounts reference them.
21. Every Ban and Unban cycle must be preserved in Ban History.
22. Ban History preserves student and administrator identity snapshots.
23. Ban History search supports Student Name and Student ID.
24. Each module deletes the data it owns when responding to an account-deletion request.
25. Account deletion removes personal notifications, read statuses, and push subscriptions owned by Notifications.
26. When account deletion removes a team leader, Projects & Teams automatically selects a new leader if members remain and transfers invitations and join requests to that leader.
27. If account deletion leaves a team with no members, Projects & Teams deletes the team and its related invitations and join requests.
28. Identity & Access coordinates account deletion across dependent modules within the approved transaction strategy.
29. Student ID, Email, and Phone Number cannot be modified while an account is Banned.

## 18. Dependencies on Other Modules

Admin Dashboard

Provides administrative access to:

* Student Management.
* Administrative Audit Logs.
* Account and role management.
* Academic Level and Group management.
* Ban History.

The Admin Dashboard is a presentation and query layer. It does not own Identity & Access data.

User Schedule

Account deletion must remove the student’s schedule selections and related schedule records.

User Schedule owns and deletes its own data.

Assignments

Account deletion must remove the student’s assignment submission status records.

Assignments owns and deletes its own data.

Projects & Teams

Account deletion must remove memberships, invitations, and join requests associated with the student.

Projects & Teams owns and deletes its own data. It must also apply the team-leader replacement and empty-team deletion rules defined in Section 11.4.

Notifications

Account deletion must remove the student’s personal notifications, notification read statuses, and push notification subscriptions.

Notifications owns and deletes its own data.

Courses Management

Student access to course content depends on their approved schedule selections.

Account deletion must remove the student’s content viewing history through the Courses module.

Audit Log

Identity & Access sends the required administrative account-operation information to Audit Log.

Audit Log preserves the administrator’s name and Student ID as they were at the time of the operation.

## 19. Engineering Decisions

The following decisions have been established during Architecture Design.

### 19.1 Database

* MySQL is used as the module’s relational database.
* The module owns its tables and data.
* Primary Keys use BIGINT UNSIGNED AUTO_INCREMENT.
* Table and column names use snake_case.
* Tables use InnoDB.
* Character Set is utf8mb4.
* Collation is utf8mb4_unicode_ci.
* Date and time values are stored in UTC using DATETIME.
* Schema changes are managed through Laravel Migrations.
* Database constraints and Laravel Validation are both used.

### 19.2 Session Management

* Laravel Redis Session Driver is used.
* Authentication sessions are not stored in a MySQL sessions table.
* A Session Management Service exists inside Identity & Access.
* Modules and application operations that require session revocation use this service.
* Other modules must not interact directly with Redis to revoke Identity & Access sessions.

### 19.3 Account Deletion Coordination

* Account deletion uses one Database Transaction for related database cleanup operations.
* Identity & Access coordinates cleanup by calling services owned by dependent modules.
* Each module deletes only the data it owns.
* The account record is deleted after all required database cleanup operations succeed.
* Any database cleanup failure causes the transaction to roll back.
* Physical file deletion from external storage occurs after the database transaction commits.

### 19.4 Ban History Data

* Ban History uses historical identity snapshots.
* Foreign Key references to users are nullable where account deletion may remove the referenced account.
* Deleting a referenced account must not delete its historical Ban records.
* The historical snapshots remain available independently of the associated account record.
* Ban Reason is excluded from the approved database design.
* Banned accounts cannot be deleted before unban.
* Student ID, Email, and Phone Number remain assigned to the existing account throughout the ban period.
* No separate reserved-identifier records are required for banned accounts.

### 19.5 Database Relationships

* Academic Level and Group references use ON DELETE RESTRICT.
* OTP records use ON DELETE CASCADE when their associated user is deleted.
* Ban History user and administrator references use ON DELETE SET NULL.
* All Foreign Keys use ON UPDATE RESTRICT.

## 20. Acceptance Criteria

The module is considered ready when:

* Students can register and verify their email.
* Admin and Owner can create active student accounts without OTP.
* Users can authenticate and maintain persistent sessions.
* Logout revokes all sessions across devices.
* Banned accounts cannot authenticate.
* Password recovery and password changes work according to the defined rules.
* Profile editing respects role and account-status permissions.
* Student ID, Email, and Phone Number cannot be modified while an account is Banned.
* Students can permanently delete their accounts after password verification, provided they are not banned.
* Admin and Owner can permanently delete student accounts after the account has been unbanned.
* Account deletion removes associated data through the responsible modules.
* Account deletion follows the approved transaction and rollback strategy.
* Student ID, Email, and Phone Number belonging to a banned account cannot be reused by another account while the banned account exists.
* Unban removes the banned status and allows normal account operations.
* After unban and successful account deletion, the identifiers become available for reuse, subject to normal uniqueness constraints.
* Admin and Owner can manage bans and unbans.
* All Ban and Unban cycles remain available in Ban History.
* Ban History preserves student and administrator snapshots after account deletion.
* Admin and Owner can search Ban History by Student Name and Student ID.
* Admin and Owner can search and filter students using the defined fields and filters.
* Admin and Owner can view Online Status and Last Seen.
* Admin and Owner can create and manage Academic Level and Group options.
* Referenced Academic Levels and Groups cannot be deleted.
* Academic Level and Group names can be edited without changing their IDs.
* Only Owner can manage Admin roles.
* Administrative account operations appear in the Audit Log.
* Audit Log preserves the responsible administrator’s name and Student ID as they were at the time of the operation.
* Account deletion removes personal notifications, read statuses, and push subscriptions.
* Account deletion applies the agreed team-leader replacement, invitation/request transfer, and empty-team deletion rules.

⸻

End of PRD

Status: Updated Draft – Incorporates the latest approved Identity & Access decisions.
