# SIM Platform — Projects & Teams PRD

Status: Requirements updated; structural database design completed

## 1. Overview

The module enables students to form teams for courses in their confirmed schedules. It does not manage project submissions, marks, deadlines, titles, descriptions or attachments. One project may exist per course per term; each project supports unlimited teams.

## 2. Roles and permissions

Students may view eligible projects, create teams, invite students when leading a team, request to join teams with vacancies, respond to their own invitations, and leave teams without approval. The current leader may remove other members, accept or reject join requests and delete the team, regardless of who created it.

Admin and Owner create, edit and permanently delete projects, view teams, members, leaders, pending invitations and requests, and change a team's leader. They cannot add/remove members, delete individual teams or accept/reject requests on behalf of leaders.

## 3. Project management

A project is associated with one course and is immediately available to eligible students upon creation. A course may have at most one project during the term. Projects have no separate title or submission-related fields. Project updates notify students with the relevant course in their confirmed schedules. Permanent project deletion removes all associated teams, memberships, invitations and requests; no restoration is available.

## 4. Access control

Students can access projects and perform team operations only for courses in their confirmed schedules. Enforce access on every API operation. A student may belong to at most one team per project, including a team they create. Removing a course from the confirmed schedule removes the student's membership in its project.

## 5. Teams

Students create teams without administrative approval. The creator becomes the initial leader and is counted as a member. The creator selects the maximum number of members at creation; the capacity cannot subsequently be changed. The minimum capacity is one, permitting individual teams. No team may exceed its capacity, including during concurrent acceptance operations.

The team's displayed name is the current leader's name; no independent team name is stored. Leadership changes automatically change the displayed name.

## 6. Membership and leadership

Students can leave without approval. The current leader can remove other members and delete the team. A student who already belongs to a team cannot join another team in the same project.

If the leader leaves, loses membership through a schedule change, or has their account deleted, the system selects the oldest remaining member by membership creation time, then membership identifier. If no members remain, the team and its dependent records are deleted. Pending invitations and join requests remain attached to surviving teams. Only the new current leader can manage them. Admin/Owner may change leadership to an existing member.

## 7. Invitations

The current leader sends invitations to eligible students. An invited student must accept before becoming a member. The invited student may reject the invitation. Only pending invitations are stored; acceptance or rejection deletes the invitation record. Acceptance adds membership and deletes the invitation atomically. Multiple pending invitations from different teams are allowed. Other pending invitations remain after joining a team but cannot be accepted until the student leaves that team and is otherwise eligible. Duplicate pending invitations for the same team and student are prohibited.

## 8. Join requests

Eligible students can request to join teams with vacancies. A student already in a team cannot send a join request in the same project. Only the current team leader can accept or reject requests. Acceptance adds membership and deletes the request atomically; rejection deletes the request. Only pending requests are stored. Duplicate pending requests for the same team and student are prohibited. Leadership changes do not delete pending requests.

## 9. Notifications

The Notifications Module handles project updates, invitations, join requests, request acceptance/rejection, invitation acceptance, member removal, leadership changes and member departures. Departure notifications go to remaining members. Notification storage and delivery belong to Notifications.

## 10. Administrative monitoring and audit

Admin and Owner may inspect teams, members, leaders and pending invitations/requests. They may change the leader but may not perform student membership operations on behalf of leaders. No Projects & Teams audit logging requirement has been approved.

## 11. Data requirements

| Entity | Data |
|---|---|
| Project | ID, course ID, timestamps |
| Team | ID, project ID, current leader ID, fixed maximum members, timestamps |
| Membership | ID, project ID, team ID, user ID, timestamps |
| Pending invitation | ID, team ID, invited user ID, creation time |
| Pending join request | ID, team ID, requesting user ID, creation time |

The companion database design specifies data types, foreign keys, constraints and indexes. Team names are derived from leader names. Accepted/rejected invitations and requests are not retained as history.

## 12. Dependencies and lifecycle

Courses provides course identifiers. Schedule Builder is the source of truth for confirmed course selections. Identity & Access provides user identities, roles and account deletion orchestration. Notifications handles approved events. The module cleans its owned records during account deletion and term reset. Term reset removes projects, teams, memberships, invitations and requests.

## 13. Acceptance criteria

- At most one project exists per course per term.
- Students cannot access projects outside their confirmed courses.
- Team creation inserts the creator as leader and member atomically.
- Capacity is selected by the creator, includes the leader, is at least one and cannot be edited.
- The displayed team name follows the current leader's name.
- No student belongs to two teams in one project.
- Concurrent acceptances cannot exceed capacity or violate membership uniqueness.
- Only the invited student can accept or reject their invitation.
- Only the current leader can accept or reject join requests, remove other members or delete the team.
- Resolved invitations and requests are deleted; duplicates are prevented.
- Other pending invitations remain after joining but cannot be accepted while membership exists.
- Leadership transfers to the oldest remaining member when necessary; empty teams are deleted.
- Pending invitations and requests survive leadership changes in surviving teams.
- Admin/Owner may change the leader but cannot directly manage membership or resolve join requests.
- Course removal and account deletion trigger membership cleanup and leadership handling.
- Project deletion and term reset remove dependent records.

## 14. Open implementation and policy decisions

Invitation/request expiry and cancellation, treatment of other pending join requests after a successful acceptance, and exact concurrency-control implementation remain unspecified. No new policies are assumed. The earlier project-wide administrator-set capacity and creator-only team deletion rules are superseded by the approved creator-set fixed capacity and current-leader deletion rules.
