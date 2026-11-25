# SIM Platform – Courses Database Design

Status: Reviewed and Approved (six-table content design)

## Table Definitions
All IDs BIGINT UNSIGNED; primary IDs AUTO_INCREMENT. Timestamps DATETIME UTC, NOT NULL unless stated. All FKs ON UPDATE RESTRICT.

| Table | Columns | Constraints and indexes |
|---|---|---|
| courses | id, name VARCHAR(255), course_code VARCHAR(50), credit_hours TINYINT UNSIGNED, created_at, updated_at | PK id; UNIQUE course_code; zero hours allowed |
| lectures | id, course_id, number SMALLINT UNSIGNED, topic VARCHAR(255), created_at, updated_at | UNIQUE(course_id, number); CHECK number > 0; course FK DELETE RESTRICT |
| sections | id, course_id, number SMALLINT UNSIGNED, topic VARCHAR(255), created_at, updated_at | UNIQUE(course_id, number); CHECK number > 0; course FK DELETE RESTRICT |
| educational_files | id, lecture_id NULL, section_id NULL, file_path VARCHAR(1024), file_size BIGINT UNSIGNED, created_at, updated_at | UNIQUE each parent FK; XOR parent; CHECK file_size > 0; parent DELETE CASCADE |
| ai_summaries | id, lecture_id NULL, section_id NULL, file_path VARCHAR(1024), created_at, updated_at | UNIQUE each parent FK; XOR parent; parent DELETE CASCADE |
| content_views | id, lecture_id NULL, section_id NULL, user_id, created_at | XOR parent; content DELETE CASCADE; user DELETE RESTRICT; INDEX(lecture_id,user_id,created_at), INDEX(section_id,user_id,created_at), INDEX(user_id) |

All unspecified columns NOT NULL. Every table has PK(id). Lecture and section parent IDs reference their corresponding tables; user_id references users.id.

## Relationships
Courses 1:M lectures and sections. Each lecture/section 1:0..1 educational file and AI summary, 1:M content views. Users 1:M content views.

## Content and Analytics
Store final PDF files and Gemini JSON summaries in File Storage; DB stores paths and metadata. Generate summary on upload and regenerate on replacement. Each successful PDF viewer load counts one immutable view, including Admin and Owner. Failed loads and direct downloads do not count. No updated_at on views. Total Views COUNT(*), Unique Viewers COUNT(DISTINCT user_id), Views Per User grouped by user_id, Last Viewed At MAX(created_at). Course-level unique viewers deduplicate across lectures and sections.

## Deletion
Course deletion/deactivation prohibited. Lecture/section hard deletion cascades metadata, summaries and views. Independent file deletion removes file metadata and physical file. Physical storage cleanup is performed by application after commit. Identity & Access calls Courses to remove user views within coordinated account-deletion transaction before deleting user. Term reset retains course definitions and removes term content and analytics.

## Integration
User Schedule owns enrollments. Courses accesses selections through service interfaces. Audit Log owns administrative records; Courses emits events. File Storage owns physical operations. Weekly schedule options and one-time changes are owned by Courses but documented separately in Schedule Builder design.

## Future Feature
Student PDF summaries are outside current design; student_summaries is a proposed future table.

## Standards
MySQL InnoDB, utf8mb4_unicode_ci, snake_case, Laravel migrations, UTC timestamps, transactions, NOT NULL by default, no redundant indexes, ON UPDATE RESTRICT.
