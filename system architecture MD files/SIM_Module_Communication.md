# Module Communication – Architecture Design Document

**Project:** SIM Platform  
**Phase:** Architecture Design  
**Status:** Completed & Approved

---

## 1. Overview

تهدف هذه المرحلة إلى تحديد كيفية التواصل والتكامل بين Modules منصة SIM، مع الحفاظ على استقلالية كل Module ومسؤولياته.

تشمل هذه الوثيقة:

- العلاقات بين الموديولات.
- استخدام Service Calls وDomain Events.
- تحديد مسؤولية كل Module في العمليات المشتركة.
- التكامل مع Shared Infrastructure.
- قواعد حذف البيانات وتصفير الترم.
- قواعد الوصول إلى بيانات الموديولات الأخرى.

## 2. Architectural Communication Rules

تعتمد المنصة القواعد التالية:

### 2.1 Module Isolation

- كل Module مسؤول عن بياناته وBusiness Logic الخاصة به.
- ممنوع على أي Module قراءة أو تعديل جداول Module آخر مباشرة.
- التواصل بين الموديولات يتم من خلال Services أو Domain Events.
- كل Module ينفذ التعديلات على بياناته بنفسه.

### 2.2 Service Calls

تُستخدم عندما يحتاج Module إلى بيانات أو نتيجة فورية من Module آخر.

أمثلة:
- الحصول على بيانات الطالب من Identity & Access.
- الحصول على بيانات المقرر من Courses.
- التحقق من المقررات الموجودة في جدول الطالب المعتمد.

### 2.3 Domain Events

تُستخدم للإعلان عن حدوث تغيير تحتاج Modules أخرى إلى الاستجابة له.

أمثلة:
- حذف مقرر.
- حذف أو تعديل موعد محاضرة.
- اعتماد تغيير في جدول الطالب.
- إنشاء تكليف أو امتحان.

الموديول الذي يرسل الحدث لا ينفذ Business Logic الخاصة بالموديولات المستقبلة.

## 3. Identity & Access Integrations

### 3.1 Identity & Access ↔ User Schedule

- عند حذف حساب طالب، يطلب Identity & Access من User Schedule حذف الجدول والاختيارات التابعة للطالب.
- User Schedule ينفذ الحذف على بياناته بنفسه.
- لا يتم إنشاء جدول تلقائي عند إنشاء حساب جديد.
- User Schedule يستعلم من Identity & Access عن بيانات الهوية وحالة الحساب عند الحاجة.

**Communication:** Service Calls + Coordinated Deletion.

### 3.2 Identity & Access ↔ Assignments

- عند حذف حساب طالب، يطلب Identity & Access من Assignments حذف سجلات حالات التسليم الخاصة به.
- Assignments ينفذ الحذف على بياناته.
- عند عرض بيانات الطالب للإدارة، يستعلم Assignments عن الاسم وStudent ID من Identity & Access.

**Communication:** Service Calls + Coordinated Deletion.

### 3.3 Identity & Access ↔ Exams

- Exams يحتفظ بمعرف المسؤول الذي أنشأ أو عدّل أو حذف الامتحان.
- يستعلم من Identity & Access عن بيانات المسؤول عند الحاجة.
- لا توجد بيانات امتحانات شخصية للطالب تحتاج إلى حذف عند حذف حسابه.
- يحتفظ Audit Log بهوية المسؤول التاريخية.

**Communication:** Service Calls.

### 3.4 Identity & Access ↔ Projects & Teams

- Projects & Teams يحتفظ بمعرفات الطلاب في العضويات والدعوات وطلبات الانضمام.
- يستعلم من Identity & Access عن بيانات الطلاب عند الحاجة، مثل الاسم وStudent ID.
- يتحقق من صلاحية الحساب عند تنفيذ العمليات التي تتطلب حسابًا نشطًا.
- عند حذف حساب طالب، يطلب Identity & Access من Projects & Teams حذف البيانات التابعة له.
- Projects & Teams ينفذ الحذف بنفسه.

**Communication:** Service Calls + Coordinated Deletion.

### 3.5 Identity & Access ↔ Notifications

عند حذف حساب طالب، يطلب Identity & Access من Notifications حذف:

- إشعارات الطالب الشخصية.
- حالات القراءة الخاصة به.
- اشتراكات Push Notifications الخاصة به.

Notifications مسؤول عن تنفيذ الحذف.

يمكن الاستعلام من Identity & Access عن بيانات الحساب عند الحاجة لعرض معلومات المستلمين.

**Communication:** Service Calls + Coordinated Deletion.

### 3.6 Identity & Access ↔ Audit Log

Identity & Access يرسل أحداث العمليات الإدارية إلى Audit Log، وتشمل:

- إنشاء حساب طالب.
- تعديل بيانات الطالب بواسطة Admin أو Owner.
- حذف حساب طالب بواسطة Admin أو Owner.
- Ban / Unban.
- ترقية أو تخفيض دور المستخدم.

Audit Log مسؤول عن حفظ السجلات.

**Historical Identity Preservation:**

- لا يعتمد السجل التاريخي على وجود حساب المسؤول حاليًا.
- يحتفظ باسم المسؤول وStudent ID كما كانا وقت تنفيذ العملية.
- تظل السجلات محفوظة حتى بعد حذف الحساب أو تغيير بياناته.

**Communication:** Domain Events.

## 4. Courses Integrations

### 4.1 Courses ↔ User Schedule

User Schedule يستعلم من Courses عن:

- بيانات المقررات.
- المواعيد المتاحة للمحاضرات والسكاشن.
- بيانات المواعيد اللازمة لعرض الجدول.

Courses يرسل Domain Events عند تعديل أو حذف Schedule Option.

User Schedule مسؤول عن معالجة تأثير التغيير على جداول الطلاب:

- عند حذف موعد، تزال الاختيارات التي تشير إليه.
- تتحول الجداول المتأثرة إلى Incomplete.
- عند تعديل موعد، يعاد التحقق من جداول الطلاب الذين اختاروه.
- تتحول الجداول التي ظهر بها تعارض فقط إلى Incomplete.
- تصل إشعارات التعديل أو الحذف لجميع الطلاب الذين لديهم المقرر في جداولهم المعتمدة.

Courses لا يتولى معالجة تعارضات جداول الطلاب.

### 4.2 Courses ↔ Assignments

- Assignments يحتفظ بمعرف المقرر Course ID.
- يستعلم من Courses عن اسم المقرر وكوده عند الحاجة.
- عند حذف Owner لمقرر، يرسل Courses حدث حذف المقرر.
- Assignments يستقبل الحدث ويحذف التكليفات وحالات التسليم التابعة للمقرر.
- عند تصفير الترم، يحذف Assignments بيانات التكليفات وحالات التسليم التشغيلية.

### 4.3 Courses ↔ Exams

- Exams يحتفظ بمعرف Course ID.
- يستعلم من Courses عن بيانات المقرر عند الحاجة.
- عند حذف مقرر، يرسل Courses حدث الحذف.
- Exams يحذف الامتحانات التابعة للمقرر.
- عند تصفير الترم، يحذف Exams بيانات الامتحانات التشغيلية.

### 4.4 Courses ↔ Projects & Teams

- Projects & Teams يحتفظ بمعرف Course ID المرتبط بالمشروع.
- يستعلم من Courses عن بيانات المقرر عند الحاجة.
- عند حذف مقرر، يرسل Courses حدث الحذف.
- Projects & Teams يحذف المشروع والفرق والعضويات والدعوات وطلبات الانضمام التابعة له.
- عند تصفير الترم، يحذف بيانات المشروعات والفرق التشغيلية.

### 4.5 Courses ↔ File Storage

- Courses يطلب من File Storage رفع ملفات المحاضرات والسكاشن.
- File Storage مسؤول عن التخزين الفعلي والتنزيل والحذف.
- Courses يحتفظ ببيانات الملف الوصفية، مثل الاسم والنوع والحجم والمعرف.
- عند حذف محتوى أو مقرر أو تصفير الترم، يطلب Courses حذف الملفات التابعة.
- File Storage لا يقرر متى يُحذف الملف؛ القرار يعود إلى الموديول المالك للمحتوى.

## 5. User Schedule Integrations

### 5.1 User Schedule ↔ Assignments

- Assignments يستعلم من User Schedule عن المقررات الموجودة في جدول الطالب المعتمد.
- User Schedule يعيد بيانات المقررات وحالة اعتماد الجدول.
- Assignments يعرض التكليفات التي يحق للطالب الوصول إليها.
- عند إزالة مقرر من الجدول المعتمد، يفقد الطالب الوصول إلى تكليفاته.
- تظل حالات التسليم محفوظة.
- إذا أعاد الطالب المقرر إلى جدوله المعتمد، يستعيد الوصول إلى حالاته السابقة.
- عند تصفير الترم، يحذف Assignments بيانات التكليفات وحالات التسليم.

**Communication:** Service Calls.

### 5.2 User Schedule ↔ Exams

- Exams يستعلم من User Schedule عن المقررات الموجودة في جداول الطلاب المعتمدة.
- يستخدم البيانات لتحديد الامتحانات التي تظهر لكل طالب.
- يستعلم عن الطلاب المسجلين في المقررات للتحقق من تعارض امتحانات مقررات مختلفة.
- Exams مسؤول عن منطق التحقق من تعارض الامتحانات.

**Communication:** Service Calls.

### 5.3 User Schedule ↔ Projects & Teams

- Projects & Teams يستعلم من User Schedule للتأكد من وجود المقرر في جدول الطالب المعتمد.
- يجب أن يكون المقرر ضمن الجدول المعتمد حتى يستطيع الطالب الوصول إلى المشروع أو إنشاء فريق أو الانضمام إلى فريق.

**عند إزالة مقرر من الجدول المعتمد:**

- User Schedule يصدر Domain Event.
- Projects & Teams يستقبل الحدث ويزيل الطالب نهائيًا من الفريق المرتبط بالمشروع.
- إذا كان الطالب قائد الفريق، يتم اختيار قائد جديد تلقائيًا.
- تنتقل الدعوات وطلبات الانضمام الحالية إلى القائد الجديد.
- إذا أصبح الفريق بلا أعضاء، يُحذف الفريق مع العضويات والدعوات والطلبات التابعة له.

User Schedule لا يتولى تعديل عضويات الفرق بنفسه.

**Communication:** Service Calls + Domain Events.

## 6. Modules ↔ Notifications

Notifications هو المسؤول المركزي عن إنشاء الإشعارات وإدارة المستلمين وحالات القراءة وإرسال Push Notifications.

تقوم الموديولات بإرسال Domain Events عند وقوع الأحداث التي تتطلب إشعارات.

### 6.1 Event Sources

**Courses**
- إنشاء تغيير استثنائي لموعد محاضرة أو سكشن.
- تعديل التغيير الاستثنائي.
- إلغاء التغيير الاستثنائي.
- تعديل أو حذف موعد أساسي وفق قواعد الإشعارات المعتمدة.

**User Schedule**
- حذف موعد اختاره الطالب.
- تعديل موعد أدى إلى تعارض في جدول الطالب.

**Assignments**
- إنشاء Assignment.
- تعديل Assignment.
- حذف Assignment.

**Exams**
- إنشاء Exam.
- تعديل Exam.
- حذف Exam.

**Projects & Teams**
- تعديل بيانات Project.
- إرسال Invitation.
- إرسال Join Request.
- قبول أو رفض Join Request.
- قبول Invitation.
- إزالة عضو.
- تغيير قائد الفريق.
- مغادرة عضو.

### 6.2 Notification Processing

- الموديول المصدر يرسل الحدث.
- Notifications يستقبل الحدث.
- Notifications يحدد المستلمين وفق قواعد الجمهور المعتمدة.
- ينشئ سجلات الإشعارات.
- يدير Read/Unread.
- يرسل Push للمستخدمين الذين فعّلوا هذه الخاصية.

### 6.3 One-Time Schedule Changes

عند إنشاء أو تعديل تغيير استثنائي لموعد:

- يظهر الموعد الاستثنائي بدل الموعد الأسبوعي الأساسي في التاريخ المحدد.
- لا يتغير الموعد الأسبوعي الأساسي.
- بعد انتهاء تاريخ الاستثناء، يعود عرض الموعد الأساسي.
- Notifications يرسل الإشعارات المتعلقة بإنشاء التغيير أو تعديله أو إلغائه.

Notifications مسؤول عن الإشعار، بينما Courses مسؤول عن بيانات التغيير وUser Schedule مسؤول عن معالجة تأثيره على جداول الطلاب عند الحاجة.

## 7. Modules ↔ Audit Log

Audit Log يستقبل الأحداث الإدارية من الموديولات المسؤولة عنها.

### Modules that send Audit Events

| Module | Operations |
|---|---|
| Identity & Access | إنشاء الحسابات وتعديلها وحذفها إداريًا، Ban/Unban، تغيير الأدوار |
| Courses | إنشاء وحذف Lecture/Section، تعديل البيانات، رفع وحذف الملفات، حذف المقرر |
| Assignments | Create / Update / Delete |
| Exams | Create / Update / Delete |
| Notifications | العمليات الإدارية الخاصة بالإشعارات |

### Modules without Administrative Audit Records

- Projects & Teams: لا يسجل عملياته في Audit Log وفق القرار المعتمد.
- User Schedule: تعديلات جداول الطلاب لا تسجل في Administrative Audit Log.

Audit Log يحتفظ بالسجلات التاريخية حتى بعد حذف البيانات التشغيلية المرتبطة بها.

## 8. Admin Dashboard Integrations

Admin Dashboard هي Presentation / Query Layer وليست Module مالكًا للبيانات.

### Communication Rules

- تستعلم Dashboard من الموديولات من خلال Services أو Query Interfaces.
- لا تقرأ جداول الموديولات مباشرة.
- كل Module مسؤول عن حساب الإحصائيات الخاصة ببياناته.
- Dashboard تجمع النتائج وتعرضها.

### Data Sources

| Dashboard Data | Source Module |
|---|---|
| Total Students | Identity & Access |
| Active Students | Identity & Access |
| Banned Students | Identity & Access |
| Courses | Courses |
| Content Analytics | Courses |
| Assignments | Assignments |
| Exams | Exams |
| Projects | Projects & Teams |
| Notifications Statistics | Notifications |
| Recent Activity | Audit Log |
| Incomplete Schedules | User Schedule |

## 9. Shared Infrastructure Integrations

### 9.1 Queue Management

- الموديول يرسل Job عند الحاجة إلى تنفيذ عملية غير متزامنة.
- Queue Management يتولى تخزين المهام وتشغيلها وإعادة المحاولة عند الفشل.
- Worker ينفذ المهمة باستخدام Business Logic الخاصة بالموديول المصدر.
- Queue Management لا يحتوي على Business Logic خاصة بالموديولات.

مثال: Courses يرسل Job لتوليد AI Summary، والـ Worker ينفذ منطق التوليد التابع لـ Courses.

### 9.2 Scheduled Tasks

- كل Module يحدد العمليات الدورية التي يحتاج إليها.
- Scheduled Tasks يحدد مواعيد تشغيلها.
- عند حلول موعد التنفيذ، يتم إرسال Job إلى Queue.
- Worker ينفذ العملية باستخدام منطق الموديول المسؤول عنها.

أمثلة:
- Notifications: حذف الإشعارات بعد 30 يومًا.
- Assignments: تحديث حالات التكليفات إلى Missed بعد انتهاء المواعيد النهائية.
- Identity & Access: عمليات تحديث Online Status عند الحاجة.

Scheduled Tasks لا ينفذ Business Logic بنفسه.

### 9.3 Redis / Caching

- كل Module يحدد البيانات التي يحتاج إلى تخزينها مؤقتًا.
- يستخدم Laravel Cache API للتعامل مع Redis.
- كل Module مسؤول عن تحديث أو إبطال Cache الخاص ببياناته.
- Redis يوفر التخزين المشترك ولا يحتوي على Business Logic.
- يمكن استخدام Redis أيضًا كـ Queue Backend وSession Storage وRate Limiting وOnline Presence.

### 9.4 Real-Time Communication

تعتمد المنصة على Laravel Reverb.

- كل Module يحدد الأحداث التي يحتاج إلى بثها.
- Reverb مسؤول عن توصيل الأحداث لحظيًا إلى الواجهات المتصلة.
- الموديول يظل مسؤولًا عن Business Logic والبيانات والصلاحيات.

أمثلة:
- Identity & Access: Online Presence.
- Notifications: الإشعارات الجديدة.
- Courses: تغييرات المواعيد الاستثنائية.

## 10. Cross-Module Deletion Rules

### 10.1 Account Deletion

Identity & Access هو المسؤول عن تنسيق حذف الحساب.

كل Module يحذف البيانات التي يملكها بنفسه.

تشمل البيانات التابعة:

- User Schedule: الجداول والاختيارات.
- Assignments: حالات التسليم.
- Projects & Teams: العضويات والدعوات وطلبات الانضمام.
- Notifications: الإشعارات الشخصية وحالات القراءة واشتراكات Push.

لا يتم حذف بيانات الموديولات الأخرى مباشرة من Identity & Access.

### 10.2 Course Deletion

Owner وحده يستطيع حذف Course.

Courses يرسل حدث حذف المقرر، وتقوم الموديولات المستقبلة بمعالجة البيانات التابعة:

- User Schedule: اختيارات المقرر من جداول الطلاب.
- Assignments: التكليفات وحالات التسليم.
- Exams: الامتحانات.
- Projects & Teams: المشروع والفرق والعضويات والدعوات والطلبات.
- File Storage: الملفات التابعة من خلال طلبات الحذف الصادرة عن Courses.

يتم الاحتفاظ بسجل الحذف في Audit Log مع بيانات المقرر قبل الحذف.

### 10.3 Term Reset

Owner يبدأ عملية التصفير يدويًا من Admin Dashboard.

الموديولات المشاركة:

| Module | Reset Responsibility |
|---|---|
| Courses | حذف المحتوى التعليمي والمواعيد التشغيلية والملفات التابعة، وتصفير إحصائيات المشاهدة |
| User Schedule | حذف جداول الطلاب واختيارات المقررات |
| Assignments | حذف التكليفات وحالات التسليم |
| Exams | حذف الامتحانات |
| Projects & Teams | حذف المشروعات والفرق والعضويات والدعوات والطلبات |
| Notifications | حذف الإشعارات والبيانات التابعة |

لا يتم تصفير:

- Identity & Access.
- Audit Log.

**Term Reset Rules:**

- التصفير يدوي بواسطة Owner.
- يجب أن تشارك الموديولات في حذف البيانات التي تملكها.
- لا يبدأ الترم الجديد قبل اكتمال التصفير بنجاح.
- أي إحصائيات مرتبطة ببيانات تشغيلية سيتم حذفها تُحذف معها.
- لا نحتفظ بأرشيف للمحتوى التشغيلي القديم.
- تفاصيل Transactions وFailure Recovery وRollback تحدد في Integration & Transactions.

## 11. Final Architectural Decisions

1. Modular Monolith هو الأسلوب المعماري المعتمد.
2. كل Module يمتلك بياناته وBusiness Logic الخاصة به.
3. لا يوجد وصول مباشر لجداول Module آخر.
4. Service Calls تستخدم للطلبات التي تحتاج بيانات أو نتائج فورية.
5. Domain Events تستخدم للإعلان عن أحداث تتطلب استجابة من Modules أخرى.
6. كل Module ينفذ التغييرات على بياناته بنفسه.
7. Notifications وAudit Log مسؤولان عن البيانات التي يملكانها.
8. Admin Dashboard تجمع البيانات من خلال Interfaces ولا تمتلكها.
9. Shared Infrastructure توفر الخدمات التقنية دون امتلاك Business Logic الخاصة بالموديولات.
10. حذف الحساب وحذف المقرر وتصفير الترم عمليات متعددة الموديولات تحتاج تنسيقًا مركزيًا.
11. تفاصيل Transactions وRetries وOutbox وFailure Recovery مؤجلة إلى Integration & Transactions.

## 12. Pending Technical Design

لم يتم تحديد تفاصيل التنفيذ التالية في هذه المرحلة:

- أسماء Services وMethods النهائية.
- أسماء Domain Events وPayload Schemas.
- Event Dispatching وListeners.
- Queue Jobs وWorkers وRetry Policies.
- Database Transactions وOutbox Pattern.
- Idempotency وRace Conditions.
- آليات Recovery عند فشل عمليات الحذف أو التصفير.
- آليات Authorization على مستوى APIs.
- تفاصيل Real-Time Channels وBroadcasting.

سيتم تحديد هذه التفاصيل في مراحل التصميم التالية.

---

**Document Status: Completed & Approved**  
**Next Phase: Database Architecture**
