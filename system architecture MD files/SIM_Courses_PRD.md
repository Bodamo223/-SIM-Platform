# SIM Platform – Courses Management

Product Requirements Document (PRD)

Status: Requirements Analysis Completed

## 1. Overview

The Courses Management module allows students to view the courses they selected through Schedule Builder and access their educational materials.

Admins and Owner manage courses and their educational content through the Admin Dashboard.

The module supports:

* Course management.
* Lecture and Section management.
* Educational file uploads.
* PDF viewing and downloading.
* Content access control.
* Content view analytics.
* Administrative activity tracking.

## 2. User Roles and Permissions

Student

* View courses selected through Schedule Builder.
* Access lectures and sections belonging to selected courses.
* View PDF files inside the platform.
* Download PDF files.
* Cannot access content belonging to unselected courses.

Admin

* Create and manage courses.
* Create, edit, and delete lectures and sections.
* Upload and delete educational files.
* Access content from all courses.
* View course enrollment statistics.
* View content analytics.
* View administrative activity logs.

Owner

* Has the same content access and management capabilities as Admin.
* Content views are included in analytics.

## 3. Course Management

### 3.1 Course Information

Each course contains:

* Course Name
* Course Code
* Credit Hours

Course Code must be unique across the platform. Credit Hours may be zero.

### 3.2 Course Creation

* Admin can create a course without adding lectures or sections.
* A newly created course becomes immediately available to students in Schedule Builder.
* No separate publishing process is required.

### 3.3 Course Editing

Admin can modify:

* Course Name
* Course Code
* Credit Hours

The course identity and its relationships with student schedules and educational content must remain intact after editing.

Course Code must remain unique.

### 3.4 Credit Hours Changes

When Credit Hours changes, the system must update the total credit hours of every affected student schedule.

The system must validate the affected schedules against the existing Schedule Builder rules.

If the updated total becomes:

* Less than 13 credit hours: Schedule becomes Incomplete.
* More than 21 credit hours: Schedule becomes Incomplete.

The affected student must receive a notification and correct the schedule before submitting it again.

### 3.5 Course Deletion and Availability

* Admin cannot delete a course.
* Admin cannot deactivate a course.
* Every created course remains available in the platform.

## 4. Course Page

Students see all courses they have selected through Schedule Builder.

Each course displays basic information, including:

* Course Name
* Course Code
* Credit Hours

The existing frontend layout and its separation of Lectures and Sections will be retained.

## 5. Lectures and Sections Management

### 5.1 Structure

Each course can contain multiple:

* Lectures
* Sections

Lectures and Sections are independent entities.

Their numbers are not related.

For example:

* Lecture 1
* Lecture 2
* Section 1
* Section 2
* Section 3

A course can have a different number of lectures and sections.

### 5.2 Required Information

Each Lecture and Section contains:

* Number
* Topic
* One educational file

### 5.3 Numbering Rules

Numbering is independent for each course and content type.

* Lecture numbers are unique within the same course.
* Section numbers are unique within the same course.
* Lecture 3 and Section 3 can coexist.
* Different courses can use the same numbers.
* Deleted numbers can be reused.

### 5.4 Content Ordering

Lectures and Sections are automatically displayed in ascending numerical order.

Manual ordering is not required.

### 5.5 Creating Content

* Admin selects a course.
* Admin chooses whether to create a Lecture or Section.
* Admin enters its number and topic.
* Admin uploads the required educational file.
* The item becomes available to students immediately after successful creation.

A Lecture or Section cannot initially be created without its required file.

### 5.6 Editing Content

Admin can modify:

* Number
* Topic

The system must enforce numbering uniqueness after editing.

### 5.7 Deleting Content

* Admin can permanently delete a Lecture or Section.
* Soft Delete is not used.
* Deleted content cannot be restored.
* The associated file must also be deleted from storage.
* Associated viewing statistics and view records must also be deleted.

### 5.8 Deleting Educational Files

Admin can delete the educational file independently without deleting the Lecture or Section.

After deletion:

* The Lecture or Section remains available.
* Its Number and Topic remain unchanged.
* Students cannot view or download its content.
* Admin can upload a replacement file later.

## 6. Educational File Management

### 6.1 Supported Formats

The system supports:

* PDF
* PPT
* PPTX

Word documents and other formats are not supported.

### 6.2 PowerPoint Conversion

When Admin uploads a PowerPoint file:

1. The system processes the uploaded file.
2. The system converts it into PDF.
3. Only the resulting PDF is retained.
4. The original PowerPoint file is not retained.

If conversion fails:

* The upload operation is rejected.
* Admin receives an error message.
* Admin can retry or upload a PDF directly.

### 6.3 File Size Limit

The maximum original file size is 10 MB.

* Files exceeding 10 MB must undergo compression.
* The original file size is evaluated before conversion.
* If the file remains above 10 MB after compression, the upload is rejected.
* Admin must reduce the file size manually.

### 6.4 File Replacement

There is no direct Replace operation.

Admin must:

1. Delete the existing file.
2. Upload the new file.

The old file is not retained as a previous version.

### 6.5 File Access

Students can view or download files only after authentication.

Access must also be checked against the student’s current course selections.

Knowing the direct file URL must not bypass authorization.

## 7. PDF Viewer

Students have two ways to access educational files:

1. Open the file inside the platform using PDF Viewer.
2. Download the file to their devices.

Admin can preview and download files through the Admin Dashboard.

The Admin does not need to use the student-facing content page.

## 8. Content Access Rules

Student Access

A student can access content only for courses currently included in their approved schedule.

When a student removes a course while editing their schedule:

* Access remains available until the student submits the schedule changes.
* After successful submission, access to that course’s content is revoked.

If the student adds the course again and submits the schedule, access is restored.

Admin and Owner Access

Admin and Owner can access content belonging to all courses regardless of their own schedule selections.

## 9. Shared Educational Content

Educational content belongs to the Course, not to individual Lecture or Section schedule options.

All students selecting the same course see the same educational files.

This applies to both:

* Lectures
* Sections

Files do not differ according to the student’s selected lecture or section time.

## 10. Content View Analytics

The system tracks educational content views.

### 10.1 View Counting

A view is recorded only when:

* The student opens the PDF Viewer.
* The PDF loads successfully.
* The file appears inside the viewer.

Opening the viewer without successfully loading the file does not count.

Direct file downloads do not count as views.

### 10.2 Analytics Metrics

For each Lecture and Section, the system tracks:

Total Views

* Total number of successful content openings.

Unique Viewers

* Number of distinct accounts that opened the content.

Admin and Owner views are included in these metrics.

### 10.3 Individual Viewer Details

Admin can open the Unique Viewers count and see the accounts that viewed the content.

The list includes:

* Student Name
* Student ID
* Number of Views
* Last Viewed At

Each account is counted once in the unique viewer metric, regardless of how many times it opened the content.

### 10.4 Analytics Levels

Analytics must be available at two levels:

Content Level

* Individual Lecture statistics.
* Individual Section statistics.

Course Level

* Total Views across the course.
* Number of distinct viewers across the course.

Course-level unique viewers must count each account once, even if it viewed multiple items in that course.

### 10.5 Analytics Interface

* Analytics are displayed inside the Admin Dashboard.
* No CSV or Excel export is required.

## 11. Course Enrollment Monitoring

Admin can see the number of students who selected each course.

Admin can click the count to view the list of students.

The list supports searching by:

* Student Name
* Student ID

Admin cannot add or remove students from courses.

Managing course selections remains the student’s responsibility through Schedule Builder.

Enrollment counts must update when student selections are successfully submitted.

## 12. Administrative Activity Log

The system must maintain an audit log for content management operations.

Tracked actions include:

* Creating Lectures and Sections.
* Editing Number and Topic.
* Uploading educational files.
* Deleting educational files.
* Deleting Lectures and Sections.

Each log entry should record:

* The responsible account.
* The performed action.
* The affected content.
* Date and time.

Deleted files are not retained for version recovery.

The activity log records the operation, not the deleted file itself.

## 13. Notifications and Schedule Integration

When a Credit Hours change causes a student’s schedule to become invalid:

* The schedule becomes Incomplete.
* The student receives a notification.
* The student must correct and submit the schedule again.

Changes to student course selections affect content access after successful schedule submission.

Other content-management notification requirements have not been established.

## 14. Out of Scope

The following are not included in this feature:

* Word document uploads.
* Video and external links.
* Multiple files per Lecture or Section.
* Content completion tracking.
* Manual content ordering.
* Content search inside the course page.
* File version history.
* Soft Delete and content recovery.
* Course deletion or deactivation.
* Manual student enrollment management.
* Analytics export to CSV or Excel.

## 15. Open Architectural Decisions

The following implementation details still need technical design:

1. File storage strategy and secure file delivery.
2. PowerPoint-to-PDF conversion mechanism.
3. Compression mechanism and supported compression limits.
4. Handling of temporary files during failed uploads and conversions.
5. Database structure for Courses, Lectures, Sections, Files, and View Records.
6. Efficient calculation and aggregation of viewing statistics.
7. Audit log data structure and retention policy.
8. Whether the analytics interface should label the distinct-account metric as Unique Viewers rather than Unique Students, since Admin and Owner views are included.
9. Exact fields displayed on the course cards, beyond the confirmed basic information.
10. Detailed behavior for course-level analytics when some content has been permanently deleted.

These are engineering decisions, not additional confirmed product requirements.

## 16. Acceptance Criteria

The feature is considered complete when:

* Students can see only courses selected through Schedule Builder.
* Students can access content only for their currently approved courses.
* Admin and Owner can access all course content.
* Admin can create and edit courses.
* Course Code uniqueness is enforced.
* Credit Hours changes trigger schedule recalculation and validation.
* Admin can create, edit, and permanently delete Lectures and Sections.
* Numbering rules are enforced.
* Lectures and Sections appear in ascending numerical order.
* Only supported file formats are accepted.
* PowerPoint files are converted successfully into PDF before publication.
* The file size rules are enforced.
* Students can view and download authorized PDF files.
* Viewing statistics are recorded only after successful PDF display.
* Total Views and Unique Viewers are calculated correctly.
* Admin can inspect individual viewer activity.
* Course enrollment counts and student lists are available.
* Administrative actions are recorded in the audit log.

## Approved AI Summary Update

Gemini generates a summary on content upload and regenerates it when the PDF is replaced. The summary is stored as a JSON file in File Storage; the database stores its path. AI Summary Generation and GeminiClient belong to Courses.
