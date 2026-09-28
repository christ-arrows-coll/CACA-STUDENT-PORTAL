# SAIMS — Firebase Connected Build

## Added
- Teacher → Class → Subject assignment workflow
- Attendance module
- Daily attendance by class and date
- Present / Absent / Late / Excused statuses
- Firestore attendance records tied to student, class, date and academic session
- Admin dashboard links updated

## Important
Before testing live data, configure Firebase in `js/firebase-config.js`, enable Email/Password Authentication, create Firestore, publish `firestore.rules`, and create an admin user/profile.

## Next
Results and grading → three-term academic session → promotion/repeat/graduation → student portal → fees → reports.


## Stage 5 — Results & Grading

The project now includes `admin/results.html` and `js/results.js`.

### Result structure
- CA1: 10 marks
- CA2: 10 marks
- Assignment: 10 marks
- Examination: 70 marks
- Total: 100 marks

### Default grade scale
- A = 70–100
- B = 60–69
- C = 50–59
- D = 45–49
- E = 40–44
- F = 0–39

The result is stored with the student, class, subject, term and academic session so it can later be used for report cards and promotion.

The grade scale is a configurable project default, not a claim that every school must use this exact grading policy.


## Stage 6 — Academic Session & Promotion Management

Added `admin/promotion.html` and `js/promotion.js`.

Administrators can:
- Select the completed/current academic session.
- Select a current class.
- Load all students in that class.
- Select individual students or the whole class.
- Mark each student as Promoted, Repeat, Transferred or Graduated.
- Choose a destination class.
- Move students into the new academic session.
- Preserve existing Results and Attendance documents.
- Record every movement in `promotionHistory`.
- Keep the student's previous class/session information in the promotion history.

Promotion is intentionally an administrator-controlled action. The system does not automatically promote a student merely because a calculated average meets a threshold.


## Stage 7 — Student Portal & Student Login

Added:
- `student-login.html`
- Student dashboard
- My Results
- My Attendance
- My Fees
- Announcements
- My Profile
- Student authentication using Firebase Authentication
- Student profile lookup through the `users` and `students` collections.

### Student account setup
For a real student account, the administrator must create the student's email/password account in Firebase Authentication and create/update the corresponding `users/{uid}` document with:
```json
{
  "role": "student",
  "studentId": "THE_STUDENT_DOCUMENT_ID",
  "status": "active"
}
```
The student's `students/{studentId}` document should contain an `email` field matching the Firebase Authentication email.

The registration number is used to find the student's registered email before Firebase authentication is attempted.

For security, students should only receive access to their own records. The Firestore rules should be tightened to student-specific reads before production deployment.


## Stage 8 — Student Account Management

Added:
- `admin/student-accounts.html`
- `js/student-accounts.js`
- Student portal email/account status fields
- Admin can prepare and save a student's portal account information.
- Student account status is recorded on the student document.

Security note: the frontend must not contain a Firebase service-account private key or Admin SDK credentials. Creating Firebase Authentication users programmatically should be done through a trusted backend/Cloud Function or manually through Firebase Console during this development stage.


## SAIMS-v10 — Fees Management
This version adds:
- Admin fee creation and assignment to an individual student or an entire class.
- Fee fields: session, term, description, amount, due date, paid, balance and status.
- Automatic balance/status calculation.
- Duplicate protection for the same student/session/term/description.
- Payment recording with payment reference/date and payment history.
- Admin fee summary cards and searchable/filterable fee table.
- Student portal fee page showing billed amount, payments and outstanding balance.
- Firestore rule support for a student to read only fee records linked to their own `studentId`.

### Important
Before production use, deploy the included `firestore.rules`. Student portal accounts must have a `studentId` field in `users/{uid}` matching the student's document ID.


## SAIMS-v11 — Report Card & Three-Term Summary
- Admin report-card generator.
- Select student, class, academic session and term.
- Displays CA1 /10, CA2 /10, Assignment /10, Exam /70 and Total /100.
- Automatic grade and remark display.
- Attendance summary.
- Fee status on the report.
- Annual / three-term subject summary.
- Browser print and Save as PDF support using the A4 print layout.
- Student portal includes a personal printable report card.


## SAIMS-v12 — Teacher Dashboard
- Dedicated teacher login.
- Role verification using Firestore `users/{uid}.role == "teacher"`.
- Teacher dashboard with assigned classes and subjects.
- Teacher attendance workspace.
- Teacher result entry using CA1 /10, CA2 /10, Assignment /10, Exam /70.
- Automatic total, grade and remark.
- Existing result records are updated for the same student/subject/term/session.
- Teacher profile page.
- Teacher-created attendance/results carry `teacherId`.
- Firestore rules require the authenticated teacher ID for teacher-created/updated records.

### Teacher account setup
Create the teacher's Firebase Authentication email/password account, then create:
`users/{uid}`:
```json
{"name":"Teacher Name","email":"teacher@example.com","role":"teacher","status":"active"}
```
and a matching `teachers/{uid}` document. The `classes` and `subjects` fields may contain assigned document IDs, class names, subject names or subject codes.


## Stage 12 — Advanced Student Management & Secure Student Records

Added:
- Admin student management page at `admin/students.html`
- Search by student name or registration number
- Filters by class, status and academic session
- Add/edit student records
- Proper `classId` + `className` relationship
- Student status: Active, Graduated, Transferred, Withdrawn, Suspended
- Guardian, phone, email and admission date fields
- Student class/status change history in `studentHistory`
- Class ID normalization for older records
- Student-specific Firestore read access
- Student history read access for the student concerned and administrators

Important:
- Existing records can be normalized from the Student Management page.
- A class/status change requires a reason and creates a history record.
- Do not put Firebase service-account credentials in the frontend.

## Stage 13 — Unified Class Relationships

Promotion management now uses `classId` as the primary student-class relationship while retaining `className` for compatibility.
- Promotion/repeat/transfer/graduation updates student classId and className.
- Promotion history now stores fromClassId and toClassId.
- Student history is also recorded for session/class/status changes.
- A shared `school-data.js` helper was added for class/subject/student lookups.

## Stage 14 — Academic Structure & Teacher Assignment

Added:
- `admin/assignments.html`
- Central teacher/class/subject assignment management
- Duplicate assignment protection
- Assignment activation/deactivation
- Class teacher assignment stored directly on the class document
- Assignment records now contain teacherId, classId and subjectId
- Academic session attached to every assignment

Recommended production rule:
- Keep `assignments` as the authoritative source for teacher workload and teaching relationships.


## Firebase-connected build
This package is configured for the **SAIMS Christ Arrows** Firebase project (`saims-christ-arrows-college`). The Firebase Web App configuration is stored in `js/firebase-config.js`.

### First login
Use the administrator email/password created in Firebase Authentication. The matching `users/{UID}` document must have `role: "admin"`.

### Before deployment
- Publish `firestore.rules` from Firebase Console.
- Confirm Email/Password Authentication is enabled.
- Do not commit service-account credentials.
- For teacher portal access, create a Firebase Auth account, create `users/{UID}` with `role: "teacher"`, and link the same UID to the teacher record.
- Use Admin → Academic Assignments to create active teacher/class/subject assignments.

### Teacher assignment behavior
Teacher dashboard, attendance and result-entry pages now use the `assignments` collection as the authoritative source of active class/subject relationships. The teacher's Firebase Auth UID must therefore match the `teacherId` stored in assignment records.
