# SIM Platform – Schedule Builder

**Document:** Product Requirements Document (PRD)  
**Status:** Requirements Analysis Completed – Updated to reflect approved design decisions  
**Module:** User Schedule (Schedule Builder)  
**Target users:** Students, Admins, Owner  
**Platform:** Web Application  
**Backend:** Laravel | **Database:** MySQL | **Frontend:** React  
**Companion document:** `SIM_Schedule_Builder_Database_Design.md`

## 1. Overview

Schedule Builder allows SIM students to create and manage their personal academic schedules. Students manually select courses they have registered for at college, then select one lecture time and one section time for each course.

The system validates credit hours, completeness and time conflicts. There is no integration with the university registration system. Students are responsible for selecting their actual registered courses.

## 2. User Roles

### Student

- Build, review, confirm and view a personal academic schedule.
- Select courses and lecture and section options.
- Modify a confirmed schedule and reconfirm it.
- Receive notifications about relevant schedule changes.

### Admin

- Manage courses and regular lecture and section schedule options through Courses.
- Create, edit and revoke one-time schedule changes.
- View student schedules and search by student name or Student ID.
- Cannot directly edit student schedule selections.

### Owner

Has the same schedule management permissions as Admin, in addition to platform-wide Owner privileges and manual control of the semester lifecycle.

## 3. Academic Structure

### 3.1 Weekly Schedule

Regular options run from Saturday through Thursday. Friday is excluded from regular weekly scheduling.

### 3.2 Course Selection

Students can select courses across academic levels, including courses carried over from previous levels. Prerequisites are displayed for information only and do not prevent selection.

### 3.3 Course Information

Each course displays its name, unique course code and credit hours. Credit hours are nonnegative integers; zero-credit courses are permitted.

## 4. Credit-Hour Rules

- Minimum confirmed total: 13 credit hours.
- Maximum confirmed total: 21 credit hours.
- Students cannot add courses that would exceed the maximum.
- Course removal must respect the minimum credit-hour rule.
- Confirmation is rejected if the total is outside the permitted range.
- The same limits apply to initial confirmation and later modifications.
- If an administrative credit-hour change causes a confirmed schedule to fall outside the range, the schedule becomes Incomplete and the student is notified to correct and reconfirm it.

Total credit hours are calculated from selected courses rather than stored as a separate total.

## 5. Schedule Conflict Validation

The system prevents overlapping weekly time intervals on the same day between:

- Lecture and lecture.
- Lecture and section.
- Section and section.

Adjacent intervals do not overlap. The system validates selected options and their associated courses. Exams and quizzes are not part of weekly schedule conflict validation.

Backend validation is required before confirmation and reconfirmation.

## 6. Schedule Building Workflow

After first login or successful registration verification, a student without a schedule is directed to Build Schedule.

1. Select courses.
2. Select one lecture option per course.
3. Select one section option per course.
4. Review the resulting weekly schedule and credit-hour total.
5. Validate and confirm.
6. Display the confirmed schedule.

Each student has at most one persisted schedule. No server-side Draft Mode or multiple saved schedules are supported. Initial selections are not persisted until successful confirmation.

## 7. Schedule Modification

### 7.1 Permitted Changes

Students may add courses within the 21-hour maximum, remove courses subject to the 13-hour minimum, or change selected lecture and section options.

### 7.2 Validation

All modifications must satisfy the same completeness, credit-hour and conflict rules as initial confirmation.

### 7.3 Persistence

Changes are persisted only after successful reconfirmation. The previous persisted schedule remains effective while the student edits it, except for administrative invalidation. Unsuccessful confirmation must not overwrite the previous persisted schedule.

Access to course-related content changes after successful confirmation of modified course selections.

### 7.4 History

- Record each successful confirmation and reconfirmation with its timestamp.
- Record persisted course additions, removals, lecture changes, section changes and administrative option invalidations.
- Modification history includes source, action, timestamp and before/after data.
- Confirmation history does not contain full schedule snapshots.
- Unconfirmed edits are not recorded as committed modifications.
- Both histories are removed during term reset or deletion of their parent schedule.

## 8. Administrative Schedule Management

### 8.1 Course Management

Courses owns course definitions and their management. Course codes are unique and credit hours may be zero. Changes to course credit hours require revalidation of affected schedules.

### 8.2 Regular Schedule Options

Each option belongs to one course and includes:

| Field | Requirement |
|---|---|
| Course | Required |
| Type | Lecture or Section |
| Day | Saturday–Thursday |
| Start time | Required |
| End time | Required; later than start time |
| Location | Required |

A course can offer multiple options of either type. Options have no capacity limit. Educational lectures and sections are separate from timetable options.

### 8.3 Editing Regular Options

Admin and Owner may edit options even when students have selected them. User Schedule revalidates affected schedules. If an edit causes conflicts, affected selections are invalidated, the schedules become Incomplete, and affected students receive notifications. If no conflict occurs, selections remain valid.

### 8.4 Deleting Regular Options

Deleting a selected option clears references to that option without removing the selected course. Affected schedules become Incomplete and affected students receive notifications. Students select replacements and reconfirm their schedules.

Admin and Owner cannot choose replacements on behalf of students.

## 9. One-Time Schedule Changes

### 9.1 Supported Changes

- Cancelled occurrence.
- Rescheduled occurrence.
- Location changed for a specific occurrence.

Each exception references a regular schedule option and its original occurrence date. The original location is retained. Rescheduling records the new date and start/end times; location changes record the new location. No separate notes field is required.

### 9.2 Administrative Operations

Admin and Owner can create, edit and revoke exceptions. An exception affects only the specified occurrence and does not modify the recurring weekly option or subsequent occurrences.

Revoking an exception is distinct from cancelling an occurrence. Revocation restores the regular occurrence and records the revocation time (`revoked_at`) instead of immediately deleting the exception record.

### 9.3 Conflict Handling

If a rescheduled occurrence conflicts with a student's schedule, notify the affected student. Do not automatically remove or change the student's weekly selections. No mandatory schedule modification is required for this temporary conflict.

### 9.4 Visibility and History

Students see relevant current and upcoming changes, with the exception reflected in place of the regular occurrence on the affected date. Historical exception records remain internal, subject to parent-option deletion and term reset. Creating, editing and revoking an exception triggers notifications.

## 10. Schedule Status

| Status | Meaning |
|---|---|
| Complete | Persisted schedule satisfies the required selection, credit-hour and conflict rules. |
| Incomplete | A previously persisted schedule has been invalidated by an administrative change and requires correction. |

Before first successful confirmation, no schedule record exists. Incomplete schedules can contain missing lecture or section options. Students must correct the relevant problems and reconfirm.

## 11. Weekly Schedule Display

The schedule displays days, course names, lecture/section types, start and end times, and locations. Students can filter by day. Relevant one-time exceptions are reflected on the specified occurrence dates.

Exams and quizzes appear on the Exams page only.

## 12. Administrative Monitoring

Admin and Owner can view student schedules, search by student name or Student ID, and inspect selected courses, lecture and section options, total credit hours and schedule status. They cannot directly modify a student's choices.

Admin Dashboard is a presentation and query layer and does not own schedule data.

## 13. Notifications

Notifications are required for affected students when:

- A selected regular lecture or section option is deleted.
- A regular option edit creates a conflict and invalidates a selection.
- A credit-hour change invalidates a schedule.
- A one-time schedule change is created, edited or revoked.
- A rescheduled occurrence introduces a temporary conflict.

Notifications Module owns notification creation, recipient handling and delivery. Courses and User Schedule communicate relevant events through the approved module interfaces.

## 14. Out of Scope

- University registration integration or verification of actual college enrollment.
- Multiple saved schedules or server-side draft schedules.
- Schedule export as PDF or image, or printing functionality.
- Capacity management for regular schedule options.
- Prerequisite enforcement.
- Exams and quizzes within the weekly timetable.

## 15. Data Requirements and Ownership

| Entity / table | Ownership |
|---|---|
| Course (`courses`) | Courses; existing table |
| Regular schedule option (`course_schedule_options`) | Courses |
| One-time schedule change (`one_time_schedule_changes`) | Courses |
| Student schedule (`student_schedules`) | User Schedule |
| Schedule selection (`schedule_selections`) | User Schedule |
| Confirmation history (`schedule_confirmation_history`) | User Schedule |
| Modification history (`schedule_modification_history`) | User Schedule |
| Semester (`semesters`) | Lifecycle ownership to be confirmed |

Identity & Access owns users. Each selection belongs to one schedule and one course and references the chosen lecture and section options. A course cannot appear twice in the same schedule. Invalidated option references may be null until corrected.

User Schedule is the source of truth for persisted course selections. Other modules obtain this information through defined interfaces. Detailed columns, constraints, indexes and relationships are documented in the companion database design.

## 16. Acceptance Criteria

- Students can build a schedule using courses from different academic levels, including zero-credit courses.
- A confirmed schedule contains one valid lecture and section option per selected course.
- Confirmation rejects totals below 13 or above 21 credit hours and overlapping weekly intervals.
- Initial choices are saved only after successful confirmation.
- Later changes are saved only after successful reconfirmation.
- Each student has at most one persisted schedule.
- Administrative option deletion clears affected references and marks affected schedules Incomplete.
- Administrative option edits preserve valid selections and invalidate conflicting ones.
- Credit-hour changes trigger revalidation and notification when necessary.
- Students receive required notifications and can correct incomplete schedules.
- One-time changes affect only their specified occurrence; revocation restores the regular occurrence.
- Temporary rescheduling conflicts do not automatically modify weekly selections.
- Confirmation and modification histories record the approved information.
- Admin and Owner can inspect and search student schedules but cannot directly edit them.
- Weekly schedule display excludes exams and quizzes.
- Unauthorized access is prevented at the backend.
- Term reset removes operational schedule data and requires students to build new schedules.

## 17. Semester Lifecycle

The platform supports two terms only, with no summer term. Owner manually starts and ends a term; dates are not fixed in advance. At most one term may be active at a time.

At term end, the platform coordinates removal of term-specific operational data across owning modules, including student schedules, selections, both histories, regular schedule options and one-time changes. Students rebuild their schedules in the next term. User accounts remain.

Only current-term operational data is retained; the approved operational schema does not include semester_id columns. The exact reset boundary for stable course definitions, identity lookup data and Audit Log records must be reconciled with the later broad reset instruction before destructive implementation. Retention of ended semester records and ownership of the semester lifecycle remain open.

## 18. Engineering Considerations and Remaining Review Items

- Use Laravel backend validation and MySQL constraints to protect data integrity.
- Use transactions for confirmation, history writes and coordinated administrative invalidation.
- Coordinate account deletion through Identity & Access and owning-module services.
- Use service calls and domain events; modules must not directly access another module's internal tables.
- Preserve applicable administrative audit records through Audit Log rather than duplicating actor data in exception records.
- Store DATETIME values in UTC and display them in Africa/Cairo; weekly TIME and occurrence DATE represent local academic timetable values, subject to implementation confirmation.
- Define concurrency control, event delivery and history JSON payload details during implementation design.
- Confirm handling of multiple active exceptions for the same occurrence; no new uniqueness rule is assumed.
- The approved cascade deletion of regular options also removes their exception records. Retention after parent-option deletion is not guaranteed; revocation alone preserves the record.
- Reconcile the precise term-reset boundary and semester ownership before implementation, as documented in the companion database design.

These review items do not introduce new approved product requirements.

---

**End of PRD**
