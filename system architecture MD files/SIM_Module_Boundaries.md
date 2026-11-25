# SIM Platform – Module Boundaries

## Architecture Style
Modular Monolith. A single backend application divided into independent modules communicating through defined service interfaces and domain events, without direct access to each other's internal tables.

## Core Business Modules
| Module | Responsibilities |
|---|---|
| Identity & Access | Registration, authentication, authorization, roles, profiles, academic levels and groups, verification, OTP, passwords, bans, deletion, sessions, online presence |
| Courses | Course definitions, semesters, prerequisites, lectures, sections, weekly options, one-time changes, educational files, PDF, AI summaries, content analytics |
| User Schedule | Course and option selections, weekly schedules, credit hours, validation, conflicts, confirmation, status, handling Courses changes |
| Assignments | Assignments, deadlines, submission tracking and status rules |
| Exams | Exam management, types, scheduling and conflict validation |
| Projects & Teams | Projects, teams, memberships, leadership, invitations, join requests and capacity |
| Notifications | In-app and push notifications, recipients, read status, preferences, subscriptions, retention and deletion |
| Audit Log | Administrative actions, actor and entity snapshots, old/new values and historical records |

## Presentation / Query Layer
Admin Dashboard aggregates information from business modules and owns no business data.

## Shared Infrastructure
- File Storage
- Email Delivery
- Queue Management
- Scheduled Tasks
- Logging
- Exception Handling
- Redis / Caching
- Real-Time Communication (Laravel Reverb)

## Internal Capabilities
OTP and Online Presence belong to Identity & Access. Content Analytics, AI Summary Generation and GeminiClient belong to Courses. Cache management uses Laravel Cache API and Redis.

## Architectural Rules
Each module owns its data and business logic. Cross-module access uses services or domain events. Shared infrastructure owns technical capabilities, not business rules. Do not create independent modules without architectural justification.

## Approved Structure
8 core business modules, 1 presentation/query layer, 8 shared infrastructure components.
