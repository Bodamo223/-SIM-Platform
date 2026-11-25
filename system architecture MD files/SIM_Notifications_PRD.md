# SIM Platform – Notifications Management PRD

Status: Requirements and database design approved; implementation details pending.

## Overview
In-app notifications are stored in the platform. Web Push can reach devices while the site is closed, subject to browser/device permission and support. Students receive in-app notifications regardless of their push preference.

## Student operations
- Open notifications, mark individual notifications or all notifications as read, and delete individual notifications without affecting other recipients.
- Display notifications grouped by Today, Yesterday and Older, with category filtering and pagination (Load More).
- Display five counters: total, unread, read, today and this week. Exclude notifications deleted by the student. Use Cairo local dates for display and calendar boundaries.
- Provide one global push preference across devices, enabled by default. Students can disable or re-enable it at any time. No per-category preferences.
- Browser permission must be granted separately; the platform preference cannot override it.

## Manual notifications
Admin and Owner send immediately to all students or students with a selected course in confirmed schedules. No individual targeting, scheduled sending or separate title; message only. Sent notifications cannot be edited; delete and resend instead. Admin and Owner may delete manual or automatic notifications. Store in-app notifications for all intended recipients and send push only to eligible users with enabled preferences and valid subscriptions.

## Automatic notifications
- Schedule: selected option deletion, conflicting option modification, one-time change creation, update and revocation.
- Courses: invalid schedules caused by credit-hour changes. No general content-management notification requirement.
- Assignments: creation, update and deletion; course correction exception per Assignments PRD.
- Exams: creation, update and deletion/cancellation.
- Projects & Teams: project update, invitation, join request, request acceptance/rejection, invitation acceptance, removal, leader change and departure (all remaining members).

## Categories and navigation
Categories: courses, schedule, assignments, exams, projects, announcements, account, system.
React uses category-based navigation to the relevant module page rather than item-specific deep links. This supersedes the earlier deep-link requirement. Announcements and system messages remain accessible in the notifications interface. Backend authorization remains mandatory. Specific automatic account and system event triggers are not yet defined.

## Admin management
Admin and Owner can view and delete notifications, view recipient counts, individual read/unread status and student information. Student-side deletion does not erase recipient statistics. No push delivery tracking.

## Retention
Automatically hard-delete manual and automatic notifications after 30 days, including recipient records. At term reset, delete courses, schedule, assignments, exams and projects notifications. Retain announcements, account and system notifications until normal expiry, regardless of manual or automatic type.

## Delivery architecture
Use PWA, Service Worker, Web Push API, VAPID, Laravel Reverb and Laravel Queues. No paid third-party notification service is required. Hosting and domain costs are separate. HTTPS and browser permission are required. Supported Android browsers can receive push without installation; iOS/iPadOS 16.4+ requires a supported Home Screen web app. Store multiple subscriptions per student. Create the preference record automatically in the account creation transaction.

## Subscription lifecycle
- Disabling push retains subscriptions and stops sending; enabling resumes sending to valid subscriptions.
- Delete expired subscriptions or subscriptions confirmed invalid by the push service.
- Stop push delivery to banned students.
- Delete subscriptions on account deletion and logout from all devices.
- Account deletion also removes recipient and preference records within the account deletion transaction.

## Implementation decisions still pending
Exact account/system event triggers, week start, category filter control, retry/deduplication strategy, concurrency and shared-browser subscription reassignment are not finalized. The database design does not record push delivery status. These are not additional approved requirements.
