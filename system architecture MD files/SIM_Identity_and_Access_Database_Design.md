# SIM Platform – Identity & Access Database Design

Module: Identity & Access
PRD ID: 84271
Design Status: Reviewed – Based on Approved Architectural Decisions

## 1. Overview

يتولى Identity & Access إدارة:

* User Accounts
* Registration and Authentication
* Account Roles and Status
* OTP Verification
* Profile Management
* Academic Levels and Groups
* Account Ban and Unban History
* Online Presence
* Account Deletion Coordination

يمتلك الموديول بياناته وجداوله، ولا تسمح المعمارية للموديولات الأخرى بالتعامل المباشر مع هذه الجداول.

## 2. Database Tables

الجداول الأساسية:

1. users
2. otp_verifications
3. academic_levels
4. groups
5. ban_records

لا يتضمن التصميم جدول Sessions لأن Laravel يستخدم Redis Session Driver.

ولا يتضمن جدول Audit Log لأن Audit Log موديول مستقل.

⸻

## 3. Table Definitions

### 3.1 users

Column	Data Type	Constraints
id	BIGINT UNSIGNED	PK, AUTO_INCREMENT
name	VARCHAR(255)	NOT NULL
student_id	VARCHAR(11)	NOT NULL, UNIQUE
email	VARCHAR(255)	NOT NULL, UNIQUE
phone_number	VARCHAR(11)	NOT NULL, UNIQUE
password	VARCHAR(255)	NOT NULL
role	ENUM(‘student’,‘admin’,‘owner’)	NOT NULL, DEFAULT ‘student’
academic_level_id	BIGINT UNSIGNED	FK, NULL
group_id	BIGINT UNSIGNED	FK, NULL
account_status	ENUM(‘active’,‘banned’)	NOT NULL, DEFAULT ‘active’
is_online	BOOLEAN	NOT NULL, DEFAULT FALSE
last_seen_at	DATETIME	NULL
created_at	DATETIME	NOT NULL
updated_at	DATETIME	NOT NULL

Constraints and Rules

* Student ID فريد بين الحسابات، ويتكون من أرقام فقط، وبحد أقصى 11 رقمًا.
* Email إلزامي وفريد.
* Phone Number إلزامي وفريد عالميًا.
* Phone Number يقبل أرقام المحمول المصرية المحلية المكونة من 11 رقمًا فقط، مع التحقق من الصيغة والبادئة.
* Password يخزن باستخدام Password Hashing.
* الحسابات الجديدة تحصل على Student role تلقائيًا.
* Owner واحد فقط، ويُنشأ أثناء Initial Setup.
* Academic Level وGroup قابلان لـ NULL للحسابات التي لا تحتاج بيانات الطالب.
* Account Status مستقل عن Online Status.

Indexes

* PRIMARY KEY (id)
* UNIQUE (student_id)
* UNIQUE (email)
* UNIQUE (phone_number)
* INDEX (academic_level_id)
* INDEX (group_id)
* INDEX (account_status)
* INDEX (is_online)
* INDEX (created_at)

Search

حقل البحث الإداري الواحد يبحث باستخدام Contains في:

* name
* student_id
* phone_number

Email غير داخل البحث.

الفلاتر:

* Academic Level
* Group
* Account Status
* Online Status

الترتيب: created_at DESC, id DESC.

حجم الصفحة: 20 سجلًا.

تُحفظ حالة البحث والفلاتر والصفحة في URL Parameters.

البحث باستخدام Contains مع Leading Wildcard لا يستفيد عادةً من B-tree Index بالطريقة المعتادة.

### 3.2 otp_verifications

Column	Data Type	Constraints
id	BIGINT UNSIGNED	PK, AUTO_INCREMENT
user_id	BIGINT UNSIGNED	FK, NULL
email	VARCHAR(255)	NOT NULL
code_hash	VARCHAR(255)	NOT NULL
purpose	ENUM(‘email_verification’,‘password_recovery’,‘email_change’,‘phone_number_change’)	NOT NULL
failed_attempts	TINYINT UNSIGNED	NOT NULL, DEFAULT 0
expires_at	DATETIME	NOT NULL
verified_at	DATETIME	NULL
created_at	DATETIME	NOT NULL
updated_at	DATETIME	NOT NULL

Rules

* OTP يتكون من 6 أرقام.
* مدة الصلاحية 5 دقائق.
* إعادة الإرسال بعد 60 ثانية.
* الحد الأقصى 5 محاولات خاطئة لكل OTP.
* الحد الأقصى 5 عمليات إرسال لكل حساب ولكل Purpose خلال ساعة.
* إرسال OTP جديد لنفس العملية يبطل OTP السابق.
* لا يسمح بإعادة استخدام OTP بعد نجاح التحقق.
* يتم حذف سجلات OTP بعد 24 ساعة.
* user_id يسمح بـ NULL لدعم التسجيل الأولي قبل إنشاء الحساب.
* code_hash يحتفظ بالـ Hash وليس OTP الأصلي.

Indexes

* PRIMARY KEY (id)
* INDEX (email, purpose)
* INDEX (expires_at)
* INDEX (user_id)

Foreign Key

user_id references users.id

* ON DELETE CASCADE
* ON UPDATE RESTRICT

### 3.3 academic_levels

Column	Data Type	Constraints
id	BIGINT UNSIGNED	PK, AUTO_INCREMENT
name	VARCHAR(255)	NOT NULL, UNIQUE
created_at	DATETIME	NOT NULL
updated_at	DATETIME	NOT NULL

Indexes

* PRIMARY KEY (id)
* UNIQUE (name)

Rules

* Admin وOwner يستطيعان إنشاء وتعديل وإدارة Academic Levels.
* تعديل الاسم مسموح حتى مع وجود حسابات مرتبطة.
* الحذف ممنوع طالما توجد حسابات مرتبطة.

### 3.4 groups

Column	Data Type	Constraints
id	BIGINT UNSIGNED	PK, AUTO_INCREMENT
name	VARCHAR(255)	NOT NULL, UNIQUE
created_at	DATETIME	NOT NULL
updated_at	DATETIME	NOT NULL

Indexes

* PRIMARY KEY (id)
* UNIQUE (name)

Rules

* Admin وOwner يستطيعان إنشاء وتعديل وإدارة Groups.
* تعديل الاسم مسموح حتى مع وجود حسابات مرتبطة.
* الحذف ممنوع طالما توجد حسابات مرتبطة.
* Group مستقل عن Academic Level ولا يحتوي على academic_level_id.

### 3.5 ban_records

كل سجل يمثل دورة Ban وUnban واحدة.

Column	Data Type	Constraints
id	BIGINT UNSIGNED	PK, AUTO_INCREMENT
user_id	BIGINT UNSIGNED	FK, NULL
student_name	VARCHAR(255)	NOT NULL
student_id	VARCHAR(11)	NOT NULL
banned_by	BIGINT UNSIGNED	FK, NULL
banned_by_name	VARCHAR(255)	NOT NULL
banned_by_student_id	VARCHAR(11)	NOT NULL
banned_at	DATETIME	NOT NULL
unbanned_by	BIGINT UNSIGNED	FK, NULL
unbanned_by_name	VARCHAR(255)	NULL
unbanned_by_student_id	VARCHAR(11)	NULL
unbanned_at	DATETIME	NULL
created_at	DATETIME	NOT NULL
updated_at	DATETIME	NOT NULL

Historical Snapshots

يحتفظ الجدول بنسخ تاريخية من:

* Student Name
* Student ID
* Banning Admin Name
* Banning Admin Student ID
* Unbanning Admin Name
* Unbanning Admin Student ID

لا يتم الاحتفاظ بنسخ من Email أو Phone Number داخل Ban History.

Rules

* كل Ban/Unban cycle له سجل مستقل.
* لا يتم حذف السجلات التاريخية عند تنفيذ Ban أو Unban جديد.
* لا يتم حذف سجل الحظر عند حذف حساب الطالب.
* حذف حساب Admin لا يحذف السجل التاريخي أو بيانات المسؤول المحفوظة في Snapshots.
* Ban Reason غير موجود في التصميم المعتمد.
* البحث في Ban History يكون باستخدام Student Name وStudent ID.
* لا يسمح بحذف حساب محظور قبل فك الحظر.

Indexes

* PRIMARY KEY (id)
* INDEX (user_id)
* INDEX (student_id)
* INDEX (student_name)
* INDEX (banned_by)
* INDEX (unbanned_by)

Foreign Keys

user_id references users.id

* ON DELETE SET NULL
* ON UPDATE RESTRICT

banned_by references users.id

* ON DELETE SET NULL
* ON UPDATE RESTRICT

unbanned_by references users.id

* ON DELETE SET NULL
* ON UPDATE RESTRICT

⸻

## 4. Relationships

Parent	Child	Relationship
academic_levels	users	One-to-Many
groups	users	One-to-Many
users	otp_verifications	One-to-Many
users	ban_records	One-to-Many
users	ban_records.banned_by	One-to-Many
users	ban_records.unbanned_by	One-to-Many

## 5. Referential Integrity

Relationship	ON DELETE	ON UPDATE
users → academic_levels	RESTRICT	RESTRICT
users → groups	RESTRICT	RESTRICT
otp_verifications → users	CASCADE	RESTRICT
ban_records → users	SET NULL	RESTRICT
ban_records → banned_by	SET NULL	RESTRICT
ban_records → unbanned_by	SET NULL	RESTRICT

⸻

## 6. Authentication and Session Management

* Laravel Redis Session Driver.
* لا يوجد MySQL Sessions Table.
* Session Management Service داخل Identity & Access.
* جميع عمليات إبطال الجلسات تمر عبر هذه الخدمة.
* Logout يبطل كل جلسات المستخدم على جميع الأجهزة.
* Ban يبطل جميع الجلسات فورًا.
* تغيير Password لا يبطل الجلسات الأخرى تلقائيًا.
* Cookie وSession Expiration Policies تُحدد في إعدادات المعمارية.

## 7. Account Deletion

Deletion Strategy

يتم تنفيذ عمليات تنظيف بيانات الحساب في Database Transaction واحدة.

1. Identity & Access يبدأ عملية الحذف.
2. يستدعي خدمات الموديولات المالكة للبيانات لتنظيف بيانات الحساب.
3. كل Module يحذف بياناته من جداول الموديول الخاص به.
4. عند نجاح جميع عمليات التنظيف، يحذف Identity & Access سجل الحساب.
5. عند فشل أي عملية، يتم Rollback لكل تغييرات قاعدة البيانات.

حذف الملفات الفعلية من File Storage يتم بعد نجاح Commit، لأن عمليات Storage لا تدخل ضمن Database Transaction.

Deletion Rules

* Hard Delete وليس Soft Delete.
* لا يسمح بحذف الحساب المحظور قبل فك الحظر.
* الطالب يحذف حسابه بعد التحقق من كلمة المرور الحالية.
* Admin وOwner يستطيعان حذف حسابات الطلاب.
* لا يتطلب الحذف الإداري تأكيد Student ID إضافيًا.
* لا يتم تسجيل Student-Initiated Deletion في Administrative Audit Log.
* الحذف الإداري يُسجل في Audit Log.

Data Ownership During Deletion

Module	Data to Delete
User Schedule	Schedule selections and related records
Assignments	Student submission status records
Projects & Teams	Memberships, invitations, join requests
Notifications	Personal notifications, read statuses, push subscriptions
Courses	Content viewing history
Identity & Access	Account and owned identity data

Projects & Teams مسؤول عن تعيين قائد جديد عند حذف قائد فريق لديه أعضاء آخرون، ونقل الدعوات والطلبات إليه. إذا لم يتبق أعضاء، يحذف الفريق والبيانات المرتبطة به.

## 8. Ban and Unban

* Admin وOwner يستطيعان Ban وUnban الطلاب.
* الحظر يستمر حتى إزالته يدويًا.
* لا يوجد انتهاء تلقائي للحظر.
* Ban يغير account_status إلى banned.
* Unban يعيد account_status إلى active.
* Ban يلغي جميع الجلسات فورًا.
* كل دورة Ban/Unban تحفظ في ban_records.
* لا يسمح بحذف الحساب المحظور قبل Unban.
* Ban History تحتفظ بالـ Snapshots حتى بعد حذف الحساب لاحقًا.

## 9. Online Presence

* is_online يمثل حالة الاتصال الحالية.
* last_seen_at يمثل آخر وقت نشاط أو اتصال مسجل.
* Online وOffline وAway حالات مشتقة حسب قواعد الـ PRD.
* Away تبدأ بعد مرور 7 أيام أو أكثر من آخر زيارة.
* Online Status مرئي فقط لـ Admin وOwner.
* تحديد الاتصال والانقطاع وتحديث Last Seen يتم عبر آلية Presence المعتمدة في المعمارية.

## 10. Administrative Audit Log Integration

لا ينشئ Identity & Access جدول Audit Log.

يرسل الموديول الأحداث المطلوبة إلى Audit Log، ومنها:

* Student Account Creation
* Student Profile Updates
* Student Account Deletion
* Ban
* Unban
* Role Promotion
* Role Demotion

Audit Log يحتفظ بهوية المسؤول واسم المسؤول وStudent ID كما كانت وقت العملية.

## 11. Database Standards

يلتزم الموديول بمعايير قاعدة البيانات الموحدة لمنصة SIM:

* MySQL.
* InnoDB.
* BIGINT UNSIGNED لجميع Primary Keys.
* snake_case لأسماء الجداول والأعمدة.
* utf8mb4 وutf8mb4_unicode_ci.
* DATETIME لتخزين الوقت بصيغة UTC.
* NOT NULL افتراضيًا، وNULL عند وجود غياب اختياري حقيقي.
* ENUM للقيم الثابتة المحددة.
* Laravel Migrations لكل تغييرات Schema.
* Database Transactions للعمليات المترابطة.
* ON UPDATE RESTRICT لجميع Foreign Keys.
* عدم تكرار Indexes.
* Laravel Validation بجانب Database Constraints.
* Redis لإدارة Sessions.

## 12. PRD Alignment Updates

توجد اختلافات بين بعض نصوص أحدث نسخة PRD والقرارات المعمارية اللاحقة التي يجب عكسها في نسخة PRD النهائية:

1. منع حذف الحساب المحظور قبل فك الحظر.
2. حذف Ban Reason من المتطلبات والتصميم.
3. إضافة البحث في Ban History باستخدام Student Name وStudent ID.
4. توحيد قواعد الحذف الإداري مع قرار منع حذف الحساب المحظور.

Final Design Status: Reconstructed from previously approved database decisions. The PRD alignment updates above must be reflected in the final PRD.
