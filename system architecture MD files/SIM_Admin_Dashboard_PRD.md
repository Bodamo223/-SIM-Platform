# SIM Platform – Admin Dashboard PRD

PRD ID: 62841
Status: Requirements Analysis Completed

## Overview
Central presentation/query layer for Admin and Owner. Both see the same dashboard data; only Owner manages Admin roles. Dashboard owns no business data and cannot override module permissions.

## Statistics
Total Students, Active Students, Banned Students, Total Courses, Total Assignments, Total Exams, Total Projects, Total Teams. Click through to details. Student registration statistics over time may be added. Exclude pending Student ID reviews and notification counts.

## Global Search
Students (name, Student ID), Courses (name, code), Assignments, Exams, Projects. Open results directly.

## Quick Actions
Direct links/forms for all permitted administrative operations, including student, course, assignment, exam, project/team and notification management.

## Attention Required
Examples: course missing lectures/sections, content missing files, project without teams, incomplete student schedules. Show problem type, related entity, reason and resolution link. Derive from current data; disappear automatically when resolved.

## Upcoming Events
Upcoming exams, assignment deadlines and project deadlines, sourced from owning modules; open details.

## Recent Activity
Show administrative create/update/delete operations for courses/content, assignments, exams, projects/teams and notifications where supported by approved audit requirements. Exclude student bans from this view. Display admin name, action, entity, timestamp and details, including old/new values. Filter by action, admin and date range. All admins and Owner see all activity. Projects & Teams audit coverage remains a conflict to resolve with its PRD.

## Out of Scope
Excel/PDF export, activity export, notification count, pending-review count, ban activity in Recent Activity, custom admin permissions, separate activity logs per admin.

## Engineering Considerations and Open Decisions
Query service, aggregate queries versus caching, global search boundaries and pagination, audit normalization, efficient attention queries, default activity/event periods, deleted entities, timezone, independent loading of dashboard sections and cache invalidation.

## Acceptance Criteria
Accurate statistics, working drill-down, search and quick actions; automatically updated attention items; accurate upcoming events; shared activity and working filters; old/new values; no permission escalation.
