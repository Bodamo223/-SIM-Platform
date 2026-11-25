# SIM Platform – Exams Management

Status: Requirements Analysis Completed; updated with approved database-design requirements.

## Overview and Roles

Admin and Owner manage exams. Students access only exams for courses in their confirmed schedules, including direct API authorization. The module displays exam information rather than administering exams or recording grades.

## Exam Types and Data

Supported types:

- Quiz (`quiz`)
- Midterm Exam (`midterm`)
- Final Exam (`final`)
- Practical / Oral Exam (`practical_or_oral`), displayed as عملي / شفوي.

Unlimited exams of each type per course are allowed.

| Field | Requirement |
|---|---|
| Course | Required |
| Exam type | Required |
| Date | Required |
| Start time | Required |
| End time | Required; later than start time on the same date |
| Location | Required, free text |
| Exam topics | Required, nonempty list of nonempty text items |

Exam topics replace the previous description field. Topics are stored as a JSON array and displayed as a list in the interface.

## Admin Operations

Create exams with validation and conflict checks. Exams become visible immediately; there is no separate publishing date. Admin and Owner can edit all fields including course, even after expiry. Reject conflicting changes and preserve old data. Permanently delete exams, notify affected students and retain audit records. No restoration.

## Student Operations

Show full exam details, including the topics list, in the Exams page only, not the weekly schedule. Students see exams only for their confirmed courses. Expired exams disappear from student view but remain available to administrators until term reset. Deleted exams disappear from all views.

## Filtering and Ordering

Support filtering exams by exam type and chronological ordering. Authorization and expiry rules remain applicable to filtered results. No additional default sort direction or filter behavior has been approved.

## Conflict Validation

Reject overlapping exams for the same course, or for different courses with at least one shared student in confirmed schedules. Check on creation, time changes and course changes; exclude the exam being edited. Compare overlapping time ranges on the same date; adjacent ranges are permitted. Do not compare against weekly lectures or sections. Reject the entire operation on conflict and display the reason.

Conflict validation and persistence must use a database transaction and appropriate locking to prevent concurrent administrative operations from creating conflicting exams. Exact locking details remain pending implementation design.

## Notifications

Notify affected students on creation, update and deletion/cancellation. Recipients are students with the relevant course in confirmed schedules. The Notifications module handles delivery and storage.

## Audit Log

Record create, update and delete operations, actor, affected exam, timestamp, old/new values and deletion snapshots. Preserve history after exam deletion. The Audit Log module owns these records.

## Engineering Considerations and Open Decisions

The approved database design uses one `exams` table, a required course foreign key, a fixed exam-type ENUM, date and time fields, location, required JSON topics, and timestamps. Database constraints enforce valid time ranges and nonempty topic arrays. Laravel validates individual topic strings.

Schedule integration for shared students, exact concurrency-safe locking, notification delivery, expiry filtering, UTC timestamp storage and Cairo display, access control and audit retention remain implementation considerations. The timezone convention for separate date/time fields, exact expiry boundary and recipient handling for course changes require clarification. No new functional decisions are made on these points.

## Acceptance Criteria

- Students cannot access exams outside their confirmed courses.
- Admin and Owner can create, edit and permanently delete exams.
- All required fields are validated; topics contain at least one nonempty text item.
- The four approved exam types are supported and filterable.
- Exams become visible immediately to eligible students.
- Same-course overlaps and shared-student cross-course overlaps are rejected without changing existing data.
- Weekly schedule events do not participate in exam conflict validation.
- Expired exams are hidden from students and retained for administrators until term reset.
- Administrative changes generate the required notifications and audit records.
