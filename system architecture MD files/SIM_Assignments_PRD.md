# SIM Platform – Assignments Management

Status: Requirements Analysis Updated — Database Design Completed; implementation decisions pending
Module: Assignments Management

⸻

## 1. Overview

The Assignments Module allows students to view assignments associated with the courses they have selected in their confirmed schedules.

Students use the module to track their assignment deadlines and manually record whether they have submitted each assignment outside the platform.

Admins manage assignments through the Admin Dashboard.

The platform does not handle actual assignment submissions or verify whether a student has submitted the assignment to the instructor.

## 2. User Roles and Permissions

Student

Students can:

* View assignments associated with their confirmed courses.
* View assignment details and deadlines.
* Mark an assignment as Submitted.
* Revert an assignment status to Not Submitted before the deadline.
* View their assignment statistics.

Students cannot:

* Create or modify assignments.
* Delete assignments.
* Modify another student’s submission status.
* Mark an assignment as Submitted after its deadline.

Admin and Owner

Admins and Owner can:

* Create assignments.
* View all assignments.
* Edit assignment information.
* Change the associated course.
* Modify deadlines.
* Delete assignments permanently.
* View students’ submission statuses.

Admins and Owner cannot:

* Modify students’ submission statuses.

## 3. Assignment Management

### 3.1 Create Assignment

The Admin must provide:

| Field | Requirement |
|---|---|
| Course | Required |
| Assignment Title | Required |
| Submission Requirements | Required |
| Maximum Marks | Optional; non-negative integer |
| Location | Required |
| Deadline | Required |

Deadline consists of both date and time.

The Admin can specify any deadline, including a past date.

Multiple assignments can belong to the same course.

Each assignment has exactly one deadline at a time.

There is no attachment upload field.

### 3.2 Assignment Availability

An assignment becomes visible to eligible students immediately after creation.

There is no separate publishing date or scheduled publishing mechanism.

Students see assignments grouped under their selected courses.

The platform retrieves the student’s confirmed course selections through the Schedule Module.

### 3.3 Edit Assignment

The Admin can modify any assignment field:

* Course
* Assignment Title
* Submission Requirements
* Maximum Marks (optional)
* Location
* Deadline

Changing the associated course is intended primarily to correct an administrative mistake.

When the course changes:

* The assignment becomes associated with the new course.
* It disappears from the old course’s assignment list.
* It appears in the new course’s assignment list.
* Existing student submission records remain associated with the assignment.
* No notification is sent for this course correction.
* The operation is recorded in the Audit Log.

### 3.4 Delete Assignment

The Admin can permanently delete an assignment at any time, including after students have recorded their submission statuses.

Deletion follows Hard Delete behavior.

When an assignment is deleted:

* It disappears from all student views.
* All associated student submission records are permanently deleted.
* The assignment cannot be restored.
* The administrative deletion operation remains recorded in the Audit Log.

## 4. Student Submission Tracking

### 4.1 Submission Method

Actual assignment submission takes place outside the platform.

The platform only provides a manual tracking mechanism.

Students use the Mark as Submitted action to record that they have submitted the assignment.

This status is not official proof of submission and is not verified by the platform.

### 4.2 Assignment Statuses

Each student has one current status for each assignment.

Status	Description
Not Submitted	The student has not recorded submission.
Submitted	The student has manually recorded submission.
Missed	The deadline has passed without a recorded submission.

### 4.3 Status Rules

* New assignments start with Not Submitted for eligible students.
* Students can change Not Submitted to Submitted before the deadline.
* Students can revert Submitted to Not Submitted before the deadline.
* After the deadline, students cannot record submission or revert their status.
* If the deadline passes while the status is Not Submitted, the status becomes Missed.
* The system maintains the current status only.
* No history of previous status changes is required.

### 4.4 Deadline Changes

The Admin can modify a deadline at any time.

When the deadline is extended:

* Students with Missed status return to Not Submitted.
* They can record submission before the new deadline.
* Students already marked Submitted retain their status.

When the deadline is shortened:

* Students who have not submitted become Missed if the new deadline has passed.
* A student previously marked Submitted becomes Missed if the new deadline is changed to a time that has already passed.

The system must apply the resulting status rules consistently after every deadline modification.

### 4.5 Course Enrollment Changes

Submission records belong to the student and the assignment.

They are not dependent on how long the student remains enrolled in the course.

If a student removes a course from their confirmed schedule:

* The student loses access to that course’s assignments.
* Existing submission records are retained.

If the student adds the course again:

* Previous submission statuses remain available.

## 5. Assignment Display

The Assignments Page displays courses selected by the student in their confirmed schedule.

Under each course, the system displays its associated assignments.

Assignments are ordered by the nearest deadline first by default. Students can sort all accessible assignments by maximum marks or deadline, ascending or descending. Assignments with unknown maximum marks appear last in either direction. The default grouped-by-course view remains available; the exact presentation of global sorting is pending.

Expired assignments remain visible.

Deleted assignments disappear permanently.

Each assignment displays its relevant information, including:

* Assignment Title
* Submission Requirements
* Maximum Marks (optional)
* Location
* Deadline
* Current Submission Status

The platform already includes student assignment statistics. These remain part of the existing design and are not treated as a new requirement.

## 6. Admin Assignment Monitoring

The Admin can open an assignment and view the students associated with its course.

The student list includes:

* Student Name
* Student ID
* Submission Status

The Admin can inspect these statuses but cannot modify them.

The submission status represents the student’s personal tracking record, not an officially verified submission.

## 7. Notifications

The system sends notifications to students affected by the following operations:

* Creating a new assignment.
* Updating assignment information.
* Updating the deadline.
* Deleting an assignment.

Notifications are sent to students who have the associated course in their confirmed schedules.

Changing the course association as an administrative correction does not trigger notifications.

The Notifications Module handles notification delivery and storage.

## 8. Audit Log

All administrative assignment operations must be recorded.

The Audit Log includes:

* Admin or Owner identifier.
* Operation type: Create, Update, Delete.
* Affected assignment identifier.
* Date and time.
* Previous values and new values for modifications.

The Audit Log must retain administrative operation records even after the assignment and its submission records are permanently deleted.

## 9. Out of Scope

The following are not included in the current Assignments Module requirements:

* Uploading assignment solutions.
* Downloading assignment submissions.
* Official submission verification.
* Admin modification of student submission statuses.
* Assignment attachments.
* Assignment version history.
* Submission status history.
* Optional or mandatory assignment classification.
* Multiple deadlines for one assignment.
* Separate pages for expired assignments.

## 10. Data Requirements

The module requires the following logical entities:

Assignment

* Assignment ID
* Course ID
* Title
* Submission Requirements
* Maximum Marks (optional)
* Location
* Deadline
* Created At
* Updated At

Assignment Submission Status

* Student ID
* Assignment ID
* Current Status
* Submission Confirmation Time (nullable)

The student and assignment identifiers together must uniquely identify one submission tracking record.

Administrative Audit Record

* Actor ID
* Operation Type
* Assignment Identifier
* Previous Values
* New Values
* Timestamp

These are logical data requirements, not a finalized database schema.

## 11. Module Dependencies

Schedule Module

Provides the confirmed course selections for each student.

Assignments uses this information to determine which assignments the student can access.

Schedule remains the source of truth for course selections.

Courses Module

Provides course information and course identifiers.

Notifications Module

Handles notifications generated by assignment creation, modification, and deletion.

Identity Module

Provides student and administrator identity and role information.

Audit Log

Records administrative operations and changes.

## 12. Acceptance Criteria

* Students only see assignments associated with their confirmed courses.
* Direct API access must enforce the same access rules.
* Admins can create multiple assignments for one course.
* Every assignment requires a title, submission requirements, course, location, and deadline. Maximum marks are optional.
* All accessible assignments can be sorted by maximum marks or deadline in either direction; unknown marks appear last.
* Assignments become visible immediately after creation.
* Students can manually mark assignments as Submitted before the deadline.
* Students can revert their status before the deadline.
* The system automatically handles expired deadlines.
* Deadline extensions restore Missed statuses to Not Submitted.
* Deadline modifications apply the agreed status transition rules.
* Students cannot modify statuses after the deadline.
* Admins can edit all assignment fields.
* Admins can permanently delete assignments.
* Deleting an assignment also deletes its associated submission records.
* Course enrollment changes do not delete historical submission statuses.
* Admins can inspect student submission statuses but cannot change them.
* Required notifications are sent to affected students.
* Administrative operations and previous/new values are recorded in the Audit Log.

## 13. Engineering Considerations

The following decisions belong to the Architecture Design phase:

* Database relationships and constraints are documented in `SIM_Assignments_Database_Design.md`.
* Whether submission status records are created for all eligible students at assignment creation or created lazily.
* Efficient retrieval of assignments grouped by course.
* Approved: store all three statuses and run Laravel Scheduler every minute to update expired Not Submitted records. The backend checks the deadline directly on reads and mutations.
* Handling deadline changes across large numbers of students.
* Reliable notification dispatch.
* Timezone and deadline comparison rules.
* Transaction boundaries for permanent assignment deletion.
* Audit Log storage and retention policy.
* Event handling and consistency between modules.

These considerations do not change the confirmed product requirements.

Open clarification: the existing extension rule restores Missed to Not Submitted, but does not specify what happens when the extended deadline is still in the past. The expired-deadline rule also applies; precedence needs confirmation. Record initialization (eager or lazy) remains undecided.

⸻

End of Requirements Analysis
