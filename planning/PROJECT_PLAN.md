# Project Plan: Online Student Attendance & Academic Monitoring System

**One scenario runs through everything:**
Faculty submits attendance → attendance % recalculated → risk rule fires → student status changes → mentor notified.

The same scenario is used across:
- the Level 2 data flow diagram (DFD)
- the sequence diagram
- the state transition diagram
- the Flask implementation (`/src`)
- the live demo

## Stage 1: Problem and scope

**Problem statement.** Colleges track attendance in registers or spreadsheets and internal marks in separate sheets. Students who fall below the attendance requirement or fail internal tests are often noticed only when exam eligibility is checked. By then it is too late to help them. The system records attendance and internal marks in one place. It flags at-risk students early and gives the reason for each flag, so mentors can step in.

**Actors**
- **Admin:** sets up courses, users and the academic calendar.
- **Faculty:** records attendance and marks.
- **Mentor (faculty advisor):** acts on risk flags.
- **HOD:** views reports and sets the risk thresholds.
- **Student:** views their own attendance, marks and status.
- **Parent:** secondary stakeholder. Has no login. Receives notices only if the student agrees.

**Objectives**
1. Record attendance and marks digitally.
2. Show each student's attendance % per course, updated after every entry.
3. Detect at-risk students with rules that can be changed and that state their reason.
4. Give mentors a way to record and track interventions.
5. Provide dashboards and reports for each role.

**In scope:** a web application with the five roles above, attendance, internal marks, rule-based risk flags, interventions, email and in-app alerts, CSV export.

**Out of scope:** biometric, RFID or face-recognition attendance, fees, timetable generation, end-semester exam results, ML prediction, mobile app, SMS.

**Assumptions**
- The 75% attendance minimum comes from university rules, so it is a setting that can be changed.
- The internal-marks cutoff (default: below 40% of the maximum) is a configurable default.
- Each course section has one faculty member.
- The college supplies the student lists.

**Default risk rules** (stored in `risk_rules` so the HOD can edit them)

| Condition | Status |
|---|---|
| Attendance ≥ 75% | Normal |
| Attendance 65–74.99% (the band many universities allow to be condoned) | Warning |
| Attendance < 65% | At-Risk |
| Any internal test below 40% of its maximum | At-Risk |
| Below a cutoff in 2 or more courses | At-Risk |

**Functional requirements**
- FR1: Login, with role-based access control (RBAC).
- FR2: Admin creates and edits students, faculty, courses, sections and enrolments, and can import them from CSV.
- FR3: Faculty creates a class session (date and period).
- FR4: Faculty marks each enrolled student Present, Absent or Late (P/A/L).
- FR5: Faculty can edit attendance for 48 hours, after which it locks. Every edit is logged.
- FR6: Faculty defines assessments (name and maximum marks).
- FR7: Faculty enters or uploads marks. Each mark must be between 0 and the maximum.
- FR8: Attendance % is recalculated after every submission.
- FR9: Risk rules are checked after every attendance or marks change.
- FR10: The mentor and student are notified when the student's status changes.
- FR11: The mentor records an intervention (such as counselling or a remedial class) and closes it when done.
- FR12: Students view their own attendance, marks and status, including how many classes they still need to attend.
- FR13: The HOD views a department dashboard and an at-risk list, and can export it as CSV.
- FR14: The HOD or Admin sets the thresholds.
- FR15: An audit log records every change to marks or attendance.

**Non-functional requirements**
- **Security:** passwords hashed with bcrypt, access checked on the server, HTTPS.
- **Privacy:** students see only their own records.
- **Performance:** saving attendance for a class of 60 takes under 2 seconds.
- **Usability:** works on mobile; meets the basic WCAG 2.1 AA accessibility checks.
- **Reliability:** daily backups; submitting the same form twice loses no data.
- **Maintainability:** thresholds are stored as data, not in code; the project can be set up locally from the README in 10 minutes or less.

## Stage 2: Process model

**Choice:** Agile Scrum with 2 short sprints.

**Why it fits**
- The timeline is short, so working pieces are delivered iteratively rather than in one big release at the end.
- The work splits cleanly into two halves: Sprint 1 covers requirements and design, Sprint 2 covers build and testing.
- Each sprint ends with a review and retrospective so design issues are caught before coding starts.
- Writing the SRS and design up front in Sprint 1 keeps documentation thorough while still iterating rapidly in Sprint 2.

**Sprints**

| Sprint | Dates | Goal |
|---|---|---|
| Sprint 1 | 19 to 22 Nov | Requirements, design and plan done |
| Sprint 2 | 23 to 26 Nov | Attendance and marks working, tested, report ready |
| Release | 27 Nov | Final check and submission |

## Stage 3: Plan

**Work breakdown structure (WBS)**

| ID | Work package | Sprint | Due | Depends on |
|---|---|---|---|---|
| 1 | Requirements: problem statement, actors, survey, SRS, use cases | 1 | 20 to 21 Nov | none |
| 2 | Planning: process model, WBS, risks | 1 | 20 to 21 Nov | none |
| 3 | Design: ER diagram, architecture, DFDs, data dictionary | 1 | 21 to 22 Nov | 1 |
| 4 | Sequence diagram and student status state diagram | 1 | 22 Nov | 3 |
| 5 | Development: attendance (23 Nov), marks and risk rules (24 Nov) | 2 | 23 to 24 Nov | 3 |
| 6 | Testing: unit/functional tests and test case table | 2 | 25 Nov | 5 |
| 7 | Ethics table and professional responsibility statement | 2 | 24 Nov | 1 |
| 8 | Report, slides, deployment doc | 2 | 25 to 26 Nov | all |

**Milestones**
- **M1:** Requirements and plan done (20 to 21 Nov)
- **M2:** Design frozen, Sprint 1 review (22 Nov)
- **M3:** Attendance module working (23 Nov)
- **M4:** Marks and risk rules working (24 Nov)
- **M5:** Testing done, report and slides final (26 Nov)
- **M6:** Release (27 Nov)

**Risks**

| Risk | Mitigation |
|---|---|
| Scope creep (biometrics, ML) | Stick to the out-of-scope list |
| Diagrams don't match each other | Check DFD balancing and schema alignment at M2 |
| Merge conflicts | One branch per feature; small pull requests |
| Real student data ends up in the repo | Use synthetic data only; check `.gitignore` |
| Threshold rules unclear | Store them in `risk_rules` and document defaults as assumptions |
| Deployment fails on demo day | Run locally and keep a recorded backup |

## Stage 4: Design

### DFD Level 0

The whole system is one process. It exchanges data with five external entities:

| Entity | Sends to the system | Receives from the system |
|---|---|---|
| Admin | Course, user and enrolment data | Confirmations |
| Faculty | Attendance and marks | Class summaries |
| Mentor | Intervention notes | Risk alerts |
| HOD | Threshold settings | Department reports |
| Student | Login and view requests | Their attendance, marks and status |

### DFD Level 1

**Processes**
- 1.0 Manage users and academic setup
- 2.0 Record attendance
- 3.0 Record internal marks
- 4.0 Evaluate academic risk
- 5.0 Manage interventions and notifications
- 6.0 Generate reports

**Data stores**
- D1 Users
- D2 Courses/Enrolments
- D3 Attendance
- D4 Marks
- D5 RiskRules
- D6 RiskFlags
- D7 Interventions/Notifications

### DFD Level 2: process 4.0, Evaluate academic risk

**Inputs**
- A trigger (student and course) from process 2.0 or 3.0
- Data read from D3 (Attendance), D4 (Marks) and D5 (RiskRules)

**Outputs**
- Status written to D6 (RiskFlags)
- A status-change alert sent to process 5.0

**Sub-processes**
- 4.1 Fetch attendance % and marks
- 4.2 Apply the threshold rules
- 4.3 Compare with the current status and update the flag
- 4.4 Raise a status-change alert

**Balancing:** Level 2 has the same inputs and outputs as process 4.0 at Level 1.

### Data dictionary (key entries)
- `attendance_entry = session_id + student_id + status`, where `status = [P|A|L]`
- `attendance_pct = (P + L) / total_sessions × 100`
- `mark_entry = assessment_id + student_id + marks_obtained (0..max_marks)`
- `risk_flag = student_id + course_id + status + reasons{rule_id} + flagged_on`
- `risk_rule = rule_id + metric + operator + threshold + severity`
- `intervention = flag_id + mentor_id + action + notes + date + outcome`

### Sequence diagram: faculty submits attendance

**Participants:** Faculty, UI, AttendanceController, AttendanceService, DB, RiskService, NotificationService, Mentor.

**Normal flow**
1. Faculty selects the session and submits the attendance list.
2. The list is validated and saved.
3. Attendance % is recalculated.
4. The risk rules are checked. A student's status moves from Normal to At-Risk.
5. A flag is saved, and an alert goes to the mentor and the student.
6. Faculty sees a success screen with "2 students newly flagged".

**Error and alternative flows**
- **(a)** The faculty member does not teach this section: the request is refused (403).
- **(b)** Attendance for this session is already recorded: the system offers to edit it within the 48-hour window.
- **(c)** The record is locked (older than 48 hours): the change is rejected and must be requested from the HOD.
- **(d)** The notification fails: the attendance is still saved, and the alert is queued and retried.

### Student status states

| From | To | When |
|---|---|---|
| Normal | Warning | Attendance falls into the 65–74.99% band |
| Warning | At-Risk | Attendance falls below 65% |
| Normal | At-Risk | A marks rule fires (test below 40%, or below a cutoff in 2 or more courses) |
| At-Risk | Under Intervention | The mentor opens a case |
| Under Intervention | Resolved | The student is back above the thresholds, or the mentor closes the case |
| Resolved | Normal | Automatically, once the case is resolved |
| Warning or At-Risk | Normal | Automatically, if the student recovers before any intervention |

### Architecture

| Layer | Technology |
|---|---|
| Presentation | Server-rendered HTML pages with Bootstrap |
| Application | Flask routes and services (attendance, marks, risk, notify) |
| Data | MySQL (SQLite during development) |
| External | SMTP email |

- **Why this stack:** Python and SQL provide a simple, cohesive stack with no frontend build step.
- **Deployment:** run Flask and MySQL on a laptop or lab server, or deploy on Render / PythonAnywhere.
- **Scalability:** one department is easy to handle. To grow, add more app workers and a database index on `(student_id, course_id)`.
- **Bottlenecks:**
  - A single database gets a burst of writes when many faculty submit at the end of a period.
  - Risk rules are recalculated on every submission.
  - Email is sent while the user waits.
  - Later fix: a job queue and caching.

### ER entities
User (role), Student, Faculty, Course, Section, Enrollment, ClassSession, AttendanceRecord, Assessment, MarkEntry, RiskRule, RiskFlag, Intervention, Notification, AuditLog

## Stage 5: Repository

**Folders**
- `/requirements`: SRS, use cases, survey, sequence & state diagrams, ethics
- `/planning`: project plan, WBS, risks
- `/diagrams`: `.drawio` source files, `.png` exports, and `data_dictionary.md`
- `/src`: app code, tests and `seed_synthetic.sql` (synthetic sample data)
- `/report`: test cases and reports
- `/presentation`: slides
- `README.md`

**Housekeeping:** keep `.env` out of Git and use synthetic data only.

## Stage 6: Ethics

| Issue | Mitigation |
|---|---|
| Privacy of marks and attendance | Access by role: students see only their own data, mentors see only their mentees |
| Consent | Students are mostly adults, so parents are notified only if the student opts in. A privacy notice explains what is collected and why. Collect only what is needed (in line with India's data protection law, the DPDP Act) |
| Security | Passwords hashed with bcrypt; checked against the OWASP Top 10 (e.g. SQL injection, broken access control); no secrets in the repo; HTTPS |
| Stigma of the "At-Risk" label | Only the student, mentor and HOD can see it; supportive wording such as "Needs support"; no public lists |
| Accuracy of risk flags | Fixed rules that state their reason; a mentor reviews before any action; audit log; marks editable only within the edit window |
| Bias (medical leave, disability, sports duty) | Approved leave is not counted as absence; the mentor can override with a recorded reason |
| Accessibility | Labelled forms, keyboard navigation, good contrast; works on low-end phones |
| Data retention | Archive records after the course ends; daily backups |
| Licensing and IP | Only MIT/BSD-licensed libraries (Flask, Bootstrap); cite every source; include a LICENSE file |

**Professional responsibility.** Collect only the data needed and keep it confidential. State the system's limits clearly: the risk rules are fixed thresholds, not predictions, and the thresholds depend on university regulations.

## To verify
- The 75% attendance rule: sources disagree on whether the UGC itself requires it. Many universities allow a 65–74% band to be condoned.
- The 40% internal-marks cutoff: configurable default, no universal standard.
- The details of the DPDP Act: not checked against the official text yet. Seek legal review before production use.

## Sources
- [UGC attendance guidelines](https://bhattandjoshiassociates.com/ugc-guidelines-on-attendance-and-gujarat-universitys-policy/)
- [Kerala University condonation](https://www.keralauniversity.ac.in/pdfs/news/Condonation1602068779.pdf)
- [NIT Kurukshetra attendance rules](https://nitkkr.ac.in/wp-content/uploads/2024/11/Revised-Attendance-Rules-as-per-UG-PG-Regulations-for-appearing-in-the-End-Semester-Examination-from-Academic-Year-2024-25-onwards.pdf)
- [Classter early-warning systems](https://www.classter.com/blog/edtech/early-warning-system-launch/)
- [Ken42: at-risk indicators](https://ken42.com/blog/how-to-identify-at-risk-students-using-predictive-analytics)
- [Transparent early-warning framework (arXiv)](https://arxiv.org/pdf/2412.09483v1.pdf)
- [DPDP Act and student data](https://ksandk.com/md/data-protection-and-data-privacy/dpdp-compliance-for-schools-and-edtech/)
