# Hospital management - DevOps project

**Author**: Ivona Vargek
**Course**: Applied DevOps, Algebra Bernays University

## 1. Project Overview

The project is a small web application used for hospital administration. The application supports patient management, doctor management, appointment scheduling, medical records, authentication and role-based access control (RBAC).

**The application is deliberately simple as the focus is the DevOps pipeline around it.** Every change passes through GitHub Issues, a Pull Request and GitHub actions, and is published as a container image to the GitHub Container Registry. 

**Out of scope functionalities**: billing, insurance, patient portal, public deployment.

**Project board**: https://github.com/users/ivargek23/projects/1

### Important decisions not covered by given scenario before starting the requirements analysis

| # | Decision | Explanation |
|---|----------|-------------|
| 1 | Every user has exactly one role | Keeps the RBAC matrix unambiguous and testable. |
| 2 | Deactivation of patient and doctor records is a soft delete | Preserves data about previous appointments, as required by project requirements. |
| 3 | Medical records are append-only | Preserves the integrity of medical records. |
| 4 | The duration of an appointment is fixed to 30 minutes | Requirement regarding same date and time appointments is therefore easily enforceable by a database unique constraint. |

## 2. Requirement analysis

### 2.1. Functional Requirements

#### 2.1.1. Patients

| ID | Requirement |
|------|-------------|
| FR-01 |	The system shall allow a receptionist or an administrator to register a new patient with their name, date of birth, contact information and address.|
| FR-02 |	The system shall allow a receptionist or an administrator to search for an existing patient. |
| FR-03 |	The system shall allow a receptionist or an administrator to view patient information. |
| FR-04 |	The system shall allow a receptionist or an administrator to update patient information. |
| FR-05 | The system shall allow a receptionist or an administrator to deactivate a patient record. |
| FR-06 | The system shall preserve the appointment history information of deactivated patient records. |

#### 2.1.2. Doctors

| ID | Requirement |
|------|-------------|
| FR-07 | The system shall allow an administrator to register a new doctor with their name, contact details and medical specialization. |
| FR-08 | The system shall allow an administrator to view doctor information. |
| FR-09 | The system shall allow an administrator to update doctor information. |
| FR-10 | The system shall allow an administrator to deactivate a doctor record. |
| FR-11 | The system shall preserve the appointment history information of deactivated doctor records. |
| FR-12 | The system shall not offer deactivated doctors when an appointmeint is being scheduled. |
| FR-13 | The system shall show a receptionist which active doctors are available at a given date and time when scheduling an appointment. |
| FR-14 | The system shall show each doctor's medical specialization when a receptionist selects a doctor for an appointment. |

#### 2.1.3. Appointments

| ID | Requirement |
|------|-------------|
| FR-15 | The system shall allow a receptionist or an administrator to schedule an appointment by selecting a doctor, a patient, specifying date, time and reason for visit. |
| FR-16 | The system shall change the status of every newly created appointment to "Scheduled". |
| FR-17 | The system shall maintain the status of an appointment in one of the following states: Scheduled, Completed, Cancelled. |
| FR-18 | The system shall prevent a doctor from having two non-cancelled appointments scheduled at the same date and time. |
| FR-19 | The system shall not allow an appointment to be scheduled in the past. |
| FR-20 | The system shall not allow an appointment to be scheduled for a deactivated patient. |
| FR-21 | The system shall allow a receptionist or an administrator to view all appointments. |
| FR-22 | The system shall allow a doctor to view all their appointments. |
| FR-23 | The system shall allow a receptionist or an administrator to update a scheduled appointment to a new date and time, applying the double-booking constraint (FR-18). |
| FR-24 | The system shall allow a receptionist or an administrator to cancel a scheduled appointment. |
| FR-25 | The system shall allow a doctor to mark their own scheduled appointment as completed. |

#### 2.1.4. Medical Records

| ID | Requirement |
|------|-------------|
| FR-26 | The system shall allow a doctor to access information about patients who have appointments with them. |
| FR-27 | The system shall allow a doctor to record the date of the visit, a diagnosis and visit notes for a patient after the appointment has been completed. |
| FR-28 | The system shall associate each medical record entry with the patient, the doctor and the appointment. |
| FR-29 | The system shall retain all medical record entries and shall not allow them to be deleted. |
| FR-30 | The system shall not allow an existing medical record entry to be modified. A correction shall be added as a new entry and linked to the original entry. |
| FR-31 | The system shall allow a doctor to view the previous medical record entries of patients they are authorized to access, i.e., patients they have appointments with. |
| FR-32 | The system shall allow a doctor to export the medical record entries of patients they are authorized to access to a CSV file. |

#### 2.1.5. Authentication and Role-Based Access Control (RBAC)

| ID | Requirement |
|------|-------------|
| FR-33 | The system shall require users to log in with a username and password before accessing any protected functionality. |
| FR-34 | The system shall reject login attempts with invalid credentials using generic error message which does not specify whether the username or password is incorrect. |
| FR-35 | The system shall allow a logged-in user to log out, terminating their session. |
| FR-36 | The system shall allow each user to have exactly one role assigned to them. Possible roles are Receptionist, Doctor and Administrator. |
| FR-37 | The system shall allow an administrator to create a user account and assign a role to the user. |
| FR-38 | The system shall allow an administrator to change the role of an existing user. |
| FR-39 | The system shall allow an administrator to deactivate a user account. |
| FR-40 | The system shall not allow a deactivated user account to log in. |
| FR-41 | The system shall restrict access to a certain functionality according to the user's role, as defined by the RBAC matrix. |
| FR-42 | The system shall return authentication error (HTTP 401 Status code) to unauthenticated requests and authorization error (HTTP 403 Status code) to authenticated requests that lack necessary permissions. |
| FR-43 | The system shall not allow a receptionist or an administrator to access medical record entries. |
| FR-44 | The system shall warn a receptionist when a new patient has the same name and date of birth as an existing patient, but shall still allow the receptionist to register the new patient. |
| FR-45 | The system shall show a logged-in doctor a list of their appointments for the current day, ordered by time. |
| FR-46 | The system shall allow an administrator to view the audit of main actions (create, update, deactivate).|

### 2.2. Non-Functional Requirements

| ID | Category | Requirement | Verification method |
|------|-------------|-------------|--------------------|
|NFR-01 |	Security |	Passwords shall be stored only as salted hashes using a modern adaptive algorithm (Argon2id or bcrypt). |	Code review; automated test checks the stored value is not plaintext |
|NFR-02|	Security |	Sessions/tokens shall expire after 30 minutes of inactivity. |	Automated integration test |
|NFR-03|	Security |	Every protected endpoint shall enforce the RBAC matrix (Section 3). |	Automated authorization tests, one per role and endpoint |
|NFR-04|	Security |	The pipeline shall fail on known high or critical vulnerabilities in dependencies and the container image. |	Dependency scan + image scan (e.g. Trivy) in GitHub Actions |
|NFR-05|	Security |	No secrets shall be stored in the repository. |	Secret scanning (GitHub secret scanning / gitleaks) in CI |
|NFR-06|	Auditability |	Every create, update and deactivate action on patients, doctors, appointments and medical records shall be logged with user, timestamp and action. |	Automated integration test checks audit entries |
|NFR-07|	Data integrity |	Double-booking (FR-18) and medical-record immutability (FR-29, FR-30) shall be enforced at the database level, not only in application code. |	Automated tests against a real database (e.g. Testcontainers) |
|NFR-08|	Performance |	95% of API requests shall complete in under 500 ms with 50 concurrent users. |	Load test (k6 or Gatling) in the pipeline |
|NFR-09|	Maintainability |	Automated test line coverage shall be at least 80% for business logic. |	Coverage report (e.g. JaCoCo) with a CI quality gate |
|NFR-10|	Maintainability |	Code shall pass static analysis and formatting checks before merge. |	Linter/static analysis in CI, required status check |
|NFR-11|	Portability |	The application shall run as a container image built by the pipeline and published to GitHub Container Registry. |	CI build and push job; docker run smoke test |
|NFR-12|	Deployability |	The full application with its database shall start locally with a single command. |	docker compose up smoke test in CI |
|NFR-13|	Observability |	The application shall expose a health endpoint and structured (JSON) logs.	Automated test on health endpoint; log format check |
|NFR-14|	Usability |	Error messages shall be clear and shall not expose internal details (stack traces, SQL).	Automated API tests on error responses |

## 3. Roles and Access Control

**Receptionist**: Manages patient data and appointments

**Doctor**: Accesses clinical information, records diagnoses and visit notes

**Administrator**: Manages doctors, users, roles and system-level administration

> Patients are not system users and do not have an RBAC role.

### The RBAC Matrix

#### Patient Management

| Capability | Receptionist | Doctor | Administrator |
|---|:---:|---:|---:|
| Register new patient | Yes | No | Yes |
| Search for existing patient | Yes | Own patients only | Yes |
| View patient information | Yes | Own patients only | Yes |
| Edit patient information | Yes | No | Yes |
| Deactivate patient record | Yes | No | Yes |

#### Decisions on cells not defined by the scenario

| Capability | Decision | Reasoning |
|---|---|---|
| Doctor updates or cancels an appointment | No | The appointment calendar is owned by the front desk, a doctor can ask a receptionist to change it. |
| Administrator reads medical history | No | Administrator is not a clinical role, they have least privilege (FR-43). |
| Administrator exports medical data | No | Same reason as above; export is limited to the treating doctor. |
| Administrator manages patients and appointments | Yes | The administrator can cover front-desk work, but never clinical data. |
| Patient as a system user | Not a role | The scenario gives patients no login because they are treated as data, not users of the system. |

#### Appointment Management

| Capability | Receptionist | Doctor | Administrator |
|---|:---:|---:|---:|
| Schedule appointment | Yes | No | Yes |
| View appointment | All | Own patients only | All |
| Update appointment | Yes | No | Yes |
| Cancel appointment | Yes | No | Yes |

#### Clinical Data

| Capability | Receptionist | Doctor | Administrator |
|---|:---:|---:|---:|
| Read medical history | No | Own patients only | No |
| Record diagnosis | No | Yes | No |
| Record visit notes | No | Yes | No |
| Export authorised medical data | No | Yes | No |

#### Administration

| Capability | Receptionist | Doctor | Administrator |
|---|:---:|---:|---:|
| Register new doctor | No | No | Yes |
| Update doctor information | No | No | Yes |
| Deactivate doctor record | No | No | Yes |
| Manage user accounts | No | No | Yes |
| Manage roles and permissions | No | No | Yes |

## 4. Product Backlog

### 4.1 Backlog order

Story identifiers are stable; the order below is the priority set by the Product Owner and changes during refinement.

| Rank | Story | Reason |
|---:|---|---|
| 1 | US-17 login | Nothing can be demonstrated before login exists. |
| 2 | US-20 enforce-rbac | Highest risk; every later endpoint depends on it. |
| 3 | US-06 register-doctor | Appointments need doctors. |
| 4 | US-01 register-patient | Appointments need patients. |
| 5 | US-03 search-patient | Needed to pick a patient when scheduling. |
| 6 | US-04 schedule-appointment | Core flow and the main business rule. |
| 7 | US-09 view-appointments | A doctor must see an appointment before completing it. |
| 8 | US-12 complete-appointment | Precondition for medical records. |
| 9 | US-13 record-visit | Core clinical value. |
| 10 | US-15 view-medical-history | Clinical access rule, security-relevant. |
| 11 | US-19 manage-users | Until now users are seeded; needed before real use. |
| 12 | US-18 logout | Small; completes the session lifecycle. |
| 13 | US-02 update-patient | |
| 14 | US-07 manage-doctor | |
| 15 | US-10 reschedule-appointment | |
| 16 | US-11 cancel-appointment | |
| 17 | US-05 deactivate-patient | |
| 18 | US-08 deactivate-doctor | |
| 19 | US-14 correct-record | |
| 20 | US-16 export-medical-records | |
| 21 | US-23 view-audit-log | |
| 22 | US-21 duplicate-patient-warning | |
| 23 | US-22 doctor-daily-schedule | |

**Justification of the top three**: Login and role enforcement come first because every other story depends on them, and they carry the highest security risk, so they should be done while there is still time to correct mistakes. Registering doctors comes third because it is the simplest entity that scheduling depends on, and it lets the first sprint deliver a thin end-to-end slice (login, database write, container).

### 4.2 Definition of Done

A story or task is done when:
- [ ] All acceptance criteria are met.
- [ ] The code is merged into `main` through a reviewed pull request, never pushed directly.
- [ ] Unit and integration tests are written and pass in CI.
- [ ] The end-to-end test for the affected flow passes.
- [ ] The container image builds and is published to GHCR.
- [ ] Manual test cases (if any) are executed and documented.
- [ ] The README is updated if a requirement or decision changed.

### Patients

#### US-01: register a new patient
**Description**: As a receptionist, I want to register a new patient with their personal information so that they can be patient's data can be used for future appointments and medical records.
**Covers**: FR-01
**Acceptance Criteria**:
- Name, date of birh, contact information and address are required fields.
- Invalid input (e.g. date of birth set in future) is rejected with a clear error message.ž
- The new patient is saved with status "Active" and is assigned a unique patient ID.

### US-02: Update patient information
**Description**: As a receptionist, I want to update patient information so that the records are accurate and up-to-date.
**Covers**: FR-04
**Acceptance Criteria**:
- All fields enteret at registration can be updated.
- The validation rules for US-01 apply to updates as well.
- The change is recorded in the audit log with user, timestamp and action.

#### US-03 search-patient
**Description**: As a receptionist, I want to search for an existing patient by name or ID and view their information so that I can quickly access their data.
**Covers**: FR-02, FR-03
**Acceptance Criteria**:
- Search by full or partial name, or by exact ID.
- Results show name, date of birth and status (active/deactivated)
- Selecting a result opens the patient's details.

#### US-05 deactivate-patient
**Description**: As a receptionist, I want to deactivate a patient record so that inactive patients are not used for new appointments while their history is preserved.
**Covers**: FR-05, FR-06
**Acceptance Criteria**:
- Deactivation is a soft delete; the record and its appointment history remain.
- A deactivated patient cannot be selected for a new appointment.

### Doctors

#### US-06 register-doctor
**Description**: As an administrator, I want to register a new doctor with their name, contact details and specialization so that they can be assigned appointments.
**Covers**: FR-07
**Acceptance Criteria**:
- Name, contact details and specialization are required.
- The new doctor is saved as active.

#### US-07 manage-doctor
**Description**: As an administrator, I want to view and update doctor information so that doctor data stays accurate.
**Covers**: FR-08, FR-09
**Acceptance Criteria**:
- The administrator can view all doctors and their details.
- All fields entered at registration can be edited with the same validation.

#### US-08 deactivate-doctor
**Description**: As an administrator, I want to deactivate a doctor record so that they are no longer offered for scheduling while their history is preserved.
**Covers**: FR-10, FR-11, FR-12
**Acceptance Criteria**:
- Deactivation is a soft delete; appointment history remains.
- A deactivated doctor is not offered when scheduling.

### Appointments

#### US-04 schedule-appointment
**Description**: As a receptionist, I want to schedule an appointment for a patient with a doctor so that the patient can receive medical care.
**Covers**: FR-13, FR-14, FR-15, FR-16, FR-18, FR-19, FR-20
**Acceptance Criteria**:
- The receptionist selects an active patient, an available active doctor (with specialization shown), a date, a time and a reason.
- Only doctors free at the chosen date and time are offered.
- The appointment is saved with status "Scheduled".
- A doctor cannot have two non-cancelled appointments at the same date and time.
- Appointments in the past and for deactivated patients are rejected.

#### US-09 view-appointments
**Description**: As a receptionist or doctor, I want to view appointments so that I know the schedule.
**Covers**: FR-21, FR-22
**Acceptance Criteria**:
- A receptionist sees all appointments.
- A doctor sees only their own appointments.
- Appointments can be filtered by date and status.

#### US-10 reschedule-appointment
**Description**: As a receptionist, I want to move a scheduled appointment to a new date and time so that changes in plans can be handled.
**Covers**: FR-23
**Acceptance Criteria**:
- Only appointments with status "Scheduled" can be rescheduled.
- The double-booking and no-past-date rules apply to the new slot.

#### US-11 cancel-appointment
**Description**: As a receptionist, I want to cancel a scheduled appointment so that the slot becomes free.
**Covers**: FR-24, FR-17
**Acceptance Criteria**:
- Status changes to "Cancelled"; the appointment is not deleted.
- The freed slot can be booked again.

#### US-12 complete-appointment
**Description**: As a doctor, I want to mark my appointment as completed so that I can record the visit.
**Covers**: FR-25, FR-17
**Acceptance Criteria**:
- Only the assigned doctor can complete the appointment.
- Only "Scheduled" appointments can be completed.

### Medical Records

#### US-13 record-visit
**Description**: As a doctor, I want to record the visit date, diagnosis and notes after a completed appointment so that the patient's medical history is documented.
**Covers**: FR-27, FR-28
**Acceptance Criteria**:
- An entry can be added only for a completed appointment of the logged-in doctor.
- The entry is linked to the patient, the doctor and the appointment.

#### US-14 correct-record
**Description**: As a doctor, I want to add a correction to an existing medical record entry so that mistakes are fixed without losing the original.
**Covers**: FR-29, FR-30
**Acceptance Criteria**:
- Existing entries cannot be edited or deleted.
- A correction is a new entry linked to the original.

#### US-15 view-medical-history
**Description**: As a doctor, I want to view my patients' information and previous medical records so that I can make informed decisions.
**Covers**: FR-26, FR-31, FR-43
**Acceptance Criteria**:
- A doctor can see only patients who have appointments with them.
- Receptionists and administrators cannot access medical records.

#### US-16 export-medical-records
**Description**: As a doctor, I want to export my patients' medical records to CSV so that I can use them outside the system.
**Covers**: FR-32
**Acceptance Criteria**:
- The export contains only records the doctor is authorized to access.
- The CSV is valid UTF-8 and opens correctly in a spreadsheet tool.

### Authentication and RBAC

#### US-17 login
**Description**: As a user, I want to log in with my username and password so that I can access the system.
**Covers**: FR-33, FR-34, FR-40
**Acceptance Criteria**:
- Valid credentials of an active user log them in.
- Invalid credentials return a generic error message.
- Deactivated users cannot log in.

#### US-18 logout
**Description**: As a user, I want to log out so that nobody else can use my session.
**Covers**: FR-35
**Acceptance Criteria**:
- After logout, the session is invalid and protected requests return 401.

#### US-19 manage-users
**Description**: As an administrator, I want to create, change the role of and deactivate user accounts so that access to the system is controlled.
**Covers**: FR-36, FR-37, FR-38, FR-39
**Acceptance Criteria**:
- Every user has exactly one role: Receptionist, Doctor or Administrator.
- A deactivated user cannot log in (see US-17).

#### US-20 enforce-rbac
**Description**: As an administrator, I want access to be restricted by role so that users can perform only actions they are permitted to.
**Covers**: FR-41, FR-42, FR-43
**Acceptance Criteria**:
- Unauthenticated requests return 401.
- Authenticated requests without permission return 403.
- Every rule in the RBAC matrix is enforced.

#### US-21 duplicate-patient-warning
**Description**: As a receptionist, I want to be warned when I register a patient who may already exist, so that the same person does not end up with two records and a split history.
**Covers**: FR-44
**Acceptance Criteria**:
- Given a patient with the same name and date of birth exists, when I register a new one, then the system shows a warning with a link to the existing record.
- I can still confirm the registration if it is a different person.

#### US-22 doctor-daily-schedule
**Description**: As a doctor, I want to see today's appointments as soon as I log in, so that I can prepare for each patient without searching.
**Covers**: FR-45
**Acceptance Criteria**:
- After login, a doctor sees only their own appointments for the current day, ordered by time.
- Cancelled appointments are not shown.

#### US-23 view-audit-log
**Description**: As an administrator, I want to view who changed which record and when, so that I can investigate mistakes and misuse.
**Covers**: FR-46, NFR-06
**Acceptance Criteria**:
- Each entry shows user, timestamp, action and the affected record.
- The log can be filtered by user and date.
- Medical record content is not shown in the log.

## 5. Process and Ceremonies

The project is divided into seven sprints of two weeks each. Two weeks is long enough to deliver a meaningful vertical slice of the system, and short enough to inspect and adapt seven times before the deadline. The work is done by one person, who acts as Product Owner, Developer and Scrum Master.

### 5.1 Sprint outline

| Sprint | Weeks | Name |
|---|---|---|
| 1 | 1–2 | Walking skeleton |
| 2 | 3–4 | Core records |
| 3 | 5–6 | Appointments |
| 4 | 7–8 | Clinical records and observability |
| 5 | 9–10 | Integration and testing |
| 6 | 11–12 | Continuous delivery |
| 7 | 13–14 | Performance and final release |

Sprint goals and backlogs are in Section 8.

### 5.2 Sprint ceremonies

#### Sprint planning
- **When**: first day of each sprint, 30–60 minutes.
- **What**: review the ordered backlog, select the top items that fit the sprint capacity, break them into tasks and estimate them. Highest-priority items come first, and the workload is based on velocity measured in previous sprints.
- **Output**: sprint backlog with tasks and estimates, and a one-sentence sprint goal.

#### Sprint review
- **When**: last day of each sprint, around 30 minutes.
- **What**: demonstrate the working software (run from the published container image) to a classmate or the teaching assistant. Compare the result with the sprint goal and acceptance criteria.
- **Output**: feedback, an updated and re-ordered product backlog, and the measured velocity (completed story points).

#### Sprint retrospective
- **When**: right after the sprint review, around 20 minutes.
- **What**: answer three questions in writing: what went well, what did not go well, and what to change. The answers are committed to `docs/retrospectives/sprint-N.md`.
- **Output**: one concrete improvement to try in the next sprint, which is checked at the next retrospective.

#### Daily scrum
With a single developer there is nobody to synchronize with, so the daily scrum is replaced with a short, dated work log in `docs/worklog.md`. Each working day gets one entry: what was done, what is next, and anything blocking. The commit history and the project board confirm the log.

### 5.3 Backlog refinement

Refinement is continuous. About 10% of each sprint is reserved for it, so that the top of the backlog is always ready to be pulled into planning. Refinement is triggered by:

1. **A new or changed requirement**: the backlog does not cover a requested capability.
2. **A story that is too large**: it is estimated at 13 points or cannot be finished in one sprint, so it is split.
3. **Missing acceptance criteria**: a story without them is not ready for planning.
4. **Spillover**: an unfinished item is re-estimated and re-ordered instead of being silently moved to the next sprint.
5. **A discovery during implementation**: a technical constraint or hidden dependency changes what is realistic.
6. **Feedback from the sprint review**: the demonstration changes what should come next.

## 6. Task decomposition

### 6.1 Epics

| Epic | Name | Stories |
|---|---|---|
| E1 | Authentication and access | US-17, US-18, US-19, US-20, US-23 |
| E2 | Patient management | US-01, US-02, US-03, US-05, US-21 |
| E3 | Doctor management | US-06, US-07, US-08 |
| E4 | Appointment scheduling | US-04, US-09, US-10, US-11, US-12, US-22 |
| E5 | Medical records | US-13, US-14, US-15, US-16 |
| E6 | Platform and pipeline | PL-01 - PL-08 |

### 6.2 Platform and pipeline tasks (E6)

The pipeline represents a unit work, just as any other feature. For that reason, pipeline tasks are estimated and planned in sprints.

| ID | Item | Blocked by |
|---|---|---|
| PL-01 | app-skeleton: runnable Spring Boot application with database connection | none |
| PL-02 | ci-pipeline: build, test and coverage on every push and pull request | PL-01 |
| PL-03 | container-publish: image built, scanned and published to GHCR | PL-02 |
| PL-04 | security-scans: dependency and secret scanning in CI | PL-02 |
| PL-05 | e2e-pipeline: Playwright end-to-end tests in CI | PL-03 |
| PL-06 | monitoring: metrics and dashboard | PL-03 |
| PL-07 | delivery-pipeline: automatic deployment of the published image | PL-03 |
| PL-08 | load-tests: performance test against NFR-08 in the pipeline | PL-07 |

### 6.3 Story dependencies

| Story | Blocked by |
|---|---|
| US-17 login | PL-01 |
| US-20 enforce-rbac | US-17 |
| US-18 logout | US-17 |
| US-19 manage-users | US-20 |
| US-06 register-doctor | US-20 |
| US-07 manage-doctor | US-06 |
| US-08 deactivate-doctor | US-06 |
| US-01 register-patient | US-20 |
| US-02 update-patient | US-01 |
| US-03 search-patient | US-01 |
| US-05 deactivate-patient | US-01 |
| US-21 duplicate-patient-warning | US-01 |
| US-04 schedule-appointment | US-01, US-03, US-06 |
| US-09 view-appointments | US-04 |
| US-10 reschedule-appointment | US-04 |
| US-11 cancel-appointment | US-04 |
| US-12 complete-appointment | US-09 |
| US-22 doctor-daily-schedule | US-09 |
| US-13 record-visit | US-12 |
| US-14 correct-record | US-13 |
| US-15 view-medical-history | US-13 |
| US-16 export-medical-records | US-15 |
| US-23 view-audit-log | US-01, US-04, US-06, US-19 |

### 6.4 Decomposed items

Test task notation: `[TEST][auto]` is an automated code test run in CI; `[TEST][manual]` is a manual test case from the checklist in `docs/manual-tests.md`.

#### PL-01 app-skeleton
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-PL01.1 | Generate the Spring Boot project (Java 21, Gradle, Web, Security, JPA, Thymeleaf, Flyway, Actuator) | Code | – |
| T-PL01.2 | `docker-compose.yml` with PostgreSQL for local development | Code | T-PL01.1 |
| T-PL01.3 | Database configuration and Flyway baseline migration | Code | T-PL01.2 |
| T-PL01.4 | Actuator health endpoint and structured JSON logging | Code | T-PL01.1 |
| T-PL01.5 | [TEST][auto] Application context test against PostgreSQL in Testcontainers | Test | T-PL01.3 |

#### PL-02 ci-pipeline
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-PL02.1 | GitHub Actions workflow: build and run tests on push and pull request | Pipeline | PL-01 |
| T-PL02.2 | Gradle dependency caching in the workflow | Pipeline | T-PL02.1 |
| T-PL02.3 | JaCoCo coverage report and 80% quality gate (NFR-09) | Pipeline | T-PL02.1 |
| T-PL02.4 | Branch protection on `main`: pull request and passing CI required | Config | T-PL02.1 |
| T-PL02.5 | [TEST][manual] Open a pull request with a failing test and verify merge is blocked | Test | T-PL02.4 |

#### PL-03 container-publish
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-PL03.1 | Multi-stage `Dockerfile` with a JRE base image, running as a non-root user | Code | PL-02 |
| T-PL03.2 | Workflow job: build and push the image to GHCR on merge to `main`, tagged with version and commit SHA | Pipeline | T-PL03.1 |
| T-PL03.3 | Trivy image scan; fail on HIGH and CRITICAL vulnerabilities (NFR-04) | Pipeline | T-PL03.2 |
| T-PL03.4 | [TEST][auto] Smoke test: start the image with compose and check the health endpoint | Test | T-PL03.2 |

#### US-17 login
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-17.1 | User entity, role enum and migration | Code | PL-01 |
| T-17.2 | Seed the initial administrator from environment variables on first start | Code | T-17.1 |
| T-17.3 | Spring Security form login with `UserDetailsService`, Argon2 password encoder, deactivated users rejected | Code | T-17.1 |
| T-17.4 | Login page with a generic error message | Code | T-17.3 |
| T-17.5 | Session timeout of 30 minutes (NFR-02) | Config | T-17.3 |
| T-17.6 | [TEST][auto] Integration tests: valid login, invalid credentials, deactivated user, password stored as hash | Test | T-17.3 |
| T-17.7 | [TEST][auto] Playwright E2E: log in and reach the home page | Test | T-17.4 |

#### US-20 enforce-rbac
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-20.1 | URL authorization rules per role in `SecurityFilterChain` | Code | US-17 |
| T-20.2 | Method-level security (`@PreAuthorize`) on service methods | Code | T-20.1 |
| T-20.3 | 401 for unauthenticated and 403 for unauthorized requests, with a 403 error page | Code | T-20.1 |
| T-20.4 | [TEST][auto] Parameterized RBAC test: every role against every endpoint, matching the matrix in Section 3 | Test | T-20.2 |

#### US-06 register-doctor
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-06.1 | Doctor entity (specialization, active flag) and migration | Code | US-20 |
| T-06.2 | Service method to register a doctor with validation | Code | T-06.1 |
| T-06.3 | Register-doctor form and controller | Code | T-06.2 |
| T-06.4 | [TEST][auto] Unit tests for doctor validation | Test | T-06.2 |
| T-06.5 | [TEST][auto] MockMvc integration test, including 403 for a receptionist | Test | T-06.3 |

#### US-01 register-patient
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-01.1 | Patient entity (name, date of birth, contact, address, active flag) and migration | Code | US-20 |
| T-01.2 | Service method to register a patient with Bean Validation (date of birth not in the future) | Code | T-01.1 |
| T-01.3 | Register-patient form and controller | Code | T-01.2 |
| T-01.4 | [TEST][auto] Unit tests for validation, including boundary dates | Test | T-01.2 |
| T-01.5 | [TEST][auto] MockMvc integration test for registration | Test | T-01.3 |
| T-01.6 | [TEST][manual] Validation message wording and form layout | Test | T-01.3 |

#### US-03 search-patient
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-03.1 | Paginated repository query by partial name (case-insensitive) or ID | Code | US-01 |
| T-03.2 | Search page with result list and patient details page | Code | T-03.1 |
| T-03.3 | [TEST][auto] Repository test in Testcontainers: partial name, ID, no results | Test | T-03.1 |
| T-03.4 | [TEST][manual] Empty-state screen and long result list | Test | T-03.2 |

#### US-04 schedule-appointment
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-04.1 | Appointment entity, status enum and migration with a partial unique index on (doctor, start time) for non-cancelled appointments (FR-18, NFR-07) | Code | US-01, US-06 |
| T-04.2 | Query for active doctors free at a given slot | Code | T-04.1 |
| T-04.3 | Service method to schedule: not in the past, active patient and doctor, 30-minute slot alignment, status "Scheduled", constraint violation mapped to a clear error | Code | T-04.1 |
| T-04.4 | Booking form: patient picker, doctor picker with specialization, date, time and reason | Code | T-04.2, T-04.3, US-03 |
| T-04.5 | [TEST][auto] Unit tests for scheduling rules and slot boundaries | Test | T-04.3 |
| T-04.6 | [TEST][auto] Testcontainers test: two concurrent bookings for the same slot, only one succeeds | Test | T-04.3 |
| T-04.7 | [TEST][auto] Playwright E2E: book an appointment, then a clashing booking is refused | Test | T-04.4 |
| T-04.8 | [TEST][manual] Error message wording for refused bookings | Test | T-04.4 |

#### US-09 view-appointments
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-09.1 | Queries: all appointments for receptionist and administrator, own appointments for doctor, filtering by date and status | Code | US-04 |
| T-09.2 | Appointment list page | Code | T-09.1 |
| T-09.3 | [TEST][auto] Integration test: a doctor sees only their own appointments, a receptionist sees all | Test | T-09.1 |

#### US-12 complete-appointment
| ID | Task | Kind | Blocked by |
|---|---|---|---|
| T-12.1 | Service method to complete an appointment: assigned doctor only, "Scheduled" status only | Code | US-09 |
| T-12.2 | "Mark as completed" action in the doctor's appointment list | Code | T-12.1 |
| T-12.3 | [TEST][auto] Unit tests for allowed and rejected status transitions | Test | T-12.1 |
| T-12.4 | [TEST][auto] Integration test: another doctor gets 403 | Test | T-12.2 |

The remaining stories and pipeline items are decomposed during backlog refinement before the sprint they are planned for.

## 7. Estimation

### 7.1 Story point scale

Story points are a relative measure of size, not hours. They account for the amount of work, technical complexity, uncertainty, dependencies and testing effort.

| Points | Meaning |
|---:|---|
| 1 | **Trivial**: fully understood, no new technology, one or two files, barely any test. |
| 2 | **Small**: routine work on something that already exists, such as a form field or a list view. |
| 3 | **Standard**: several tasks and tests, nothing new. The anchor story is a 3. |
| 5 | **Substantial**: multiple layers, a business or security rule that needs thorough testing. |
| 8 | **Large or unfamiliar**: new technology, security, concurrency, or something that needs research first. |
| 13 | **Too big**: not an estimate but a signal; the item is split before it enters a sprint. |

**Anchor story**: US-01 register-patient = **3**. Every other item is estimated by comparing it to registering a patient.

### 7.2 Estimates

| ID | Item | Points | What drives the number |
|---|---|---:|---|
| US-17 | login | 8 | Security, sessions, password hashing, seeding; first use of Spring Security. |
| US-20 | enforce-rbac | 5 | Touches every endpoint; full role matrix test. |
| US-18 | logout | 1 | Mostly provided by the framework. |
| US-19 | manage-users | 5 | Changes the permission model again; deactivation interacts with login. |
| US-06 | register-doctor | 2 | Simple create form on a new entity. |
| US-07 | manage-doctor | 2 | Routine view and edit. |
| US-08 | deactivate-doctor | 2 | Soft delete plus exclusion from scheduling. |
| US-01 | register-patient | 3 | **Anchor**: several fields, validation, tests. |
| US-02 | update-patient | 2 | Reuses validation from US-01. |
| US-03 | search-patient | 2 | One query, one list view. |
| US-05 | deactivate-patient | 2 | Soft delete, history preserved. |
| US-21 | duplicate-patient-warning | 2 | One query plus a confirmation step. |
| US-04 | schedule-appointment | 8 | Two foreign keys, availability query, partial unique index, concurrency test, E2E test. |
| US-09 | view-appointments | 3 | Role-dependent queries and filters. |
| US-10 | reschedule-appointment | 3 | Reuses scheduling rules, adds status checks. |
| US-11 | cancel-appointment | 2 | Status change; slot becomes free again. |
| US-12 | complete-appointment | 2 | One status transition with an ownership check. |
| US-22 | doctor-daily-schedule | 1 | Filter on an existing query. |
| US-13 | record-visit | 3 | Comparable to the anchor, plus an ownership check. |
| US-14 | correct-record | 3 | Immutability enforced in the database, linked entries. |
| US-15 | view-medical-history | 5 | Authorization filter across patients is easy to get wrong. |
| US-16 | export-medical-records | 3 | Simple output, but the same authorization filter applies. |
| US-23 | view-audit-log | 5 | Cross-cutting: every write must produce an entry. |
| PL-01 | app-skeleton | 3 | Known tools, several pieces to wire together. |
| PL-02 | ci-pipeline | 3 | First workflow, coverage gate, branch protection. |
| PL-03 | container-publish | 5 | Dockerfile, registry authentication, image scanning. |
| PL-04 | security-scans | 2 | Mostly configuration. |
| PL-05 | e2e-pipeline | 5 | Running the application and a browser inside CI. |
| PL-06 | monitoring | 5 | New tools: metrics export and a dashboard. |
| PL-07 | delivery-pipeline | 8 | Most unfamiliar item: deployment target, secrets, rollback. |
| PL-08 | load-tests | 5 | New tool, a realistic dataset, threshold in CI. |
| | **Total** | **110** | |

## 8. Sprint Backlogs

**Capacity**: the first sprint is planned at about 15 points. From Sprint 2 onward, the plan uses the velocity measured at the previous sprint review, and this outline is updated during refinement.

| Sprint | Goal | Items | Points |
|---|---|---|---:|
| 1 | A user can log in to an application skeleton that CI builds and tests. | PL-01, PL-02, US-17 | 14 |
| 2 | Role-protected registration of doctors and patients, published as a container image to GHCR. | US-20, PL-03, US-06, US-01 | 15 |
| 3 | A receptionist can find a patient and book an appointment without double-booking; a doctor can complete it. | US-03, US-04, US-09, US-12 | 15 |
| 4 | Doctors record visits and read their patients' history; the application is scanned and monitored. | US-13, US-15, PL-04, PL-06 | 15 |
| 5 | End-to-end tests run in the pipeline; users and records can be managed. | PL-05, US-19, US-18, US-02, US-07 | 15 |
| 6 | Every merge to `main` is deployed automatically; appointments can be changed and patients deactivated. | PL-07, US-10, US-11, US-05 | 15 |
| 7 | Performance is verified and the remaining required features are complete, ready for the defence. | PL-08, US-08, US-14, US-16, US-22 | 14 |

**Stretch (not planned)**: US-23 view-audit-log (5) and US-21 duplicate-patient-warning (2). These are extra stories beyond the scenario. They are pulled in if measured velocity allows. Audit entries are still written from the start (NFR-06); only the viewing screen is deferred.

## 9. Tools and Technology Stack

| Area | Choice | Justification |
|---|---|---|
| Language | Java 25 (LTS) | Current long-term support release; the language I know best. |
| Framework | Spring Boot 4 (Web MVC, Security, Data JPA, Validation, Actuator) | Covers web, security and persistence in one consistent framework with first-class container support. |
| User interface | Thymeleaf + Bootstrap 5 | Server-rendered pages keep the system as one application, one image and one pipeline. |
| Build tool | Gradle (Kotlin DSL) | Fast incremental builds and caching in CI; type-safe build scripts. |
| Database | PostgreSQL | Official container image; partial unique indexes enforce the double-booking rule (FR-18). |
| Database migrations | Flyway | Versioned schema changes stored in the repository and applied automatically on startup. |
| Unit testing | JUnit 5, Mockito, AssertJ | Standard Java testing tools, run headless by Gradle in CI. |
| Integration testing | Spring Boot Test (MockMvc), Testcontainers | Tests run against a real PostgreSQL instead of an in-memory substitute. |
| End-to-end testing | Playwright for Java | Same language as the application; runs headless in GitHub Actions. |
| Coverage | JaCoCo | Coverage report and quality gate in CI (NFR-09). |
| Code quality | Spotless (google-java-format), SpotBugs | Formatting and static analysis enforced before merge (NFR-10). |
| Load testing | Gatling (Java DSL) | Performance tests written in Java and run from Gradle (NFR-08). |
| Containerization | Docker (multi-stage build, Eclipse Temurin JRE base), Docker Compose | Small runtime image running as non-root; whole system starts with one command (NFR-12). |
| CI/CD | GitHub Actions | Integrated with Issues, Projects and pull requests, as required by the project. |
| Artifact registry | GitHub Container Registry (GHCR) | Required by the project; images linked to the repository. |
| Security scanning | Trivy, Dependabot, GitHub secret scanning | Vulnerable dependencies, image vulnerabilities and leaked secrets fail or flag the build (NFR-04, NFR-05). |
| Monitoring | Micrometer, Prometheus, Grafana | Spring Boot exposes metrics natively; Prometheus collects them and Grafana displays the dashboard. |
| Logging | Spring Boot structured logging (JSON) | Built in; logs can be parsed by monitoring tools without extra libraries (NFR-13). |
| Diagrams | draw.io (`.drawio` files in `docs/`) | XML source that Git can version and diff. |

## 10. Branching and commit conventions

### Branch names

`<issue_number>-<item-id>-<short-description>`, lowercase, words separated by hyphens.

| Example | Issue |
|---|---|
| `37-t-pl01-1-spring-boot-skeleton` | #37 T-PL01.1 |
| `39-us-18-logout` | #39 US-18 |

**Rules**:
- One branch per Task, or per small User Story without sub-tasks.
- Always created from an up-to-date `main`, preferably from the issue (**Development → Create a branch**), so the branch is linked to the issue.
- The issue number comes first, so automation can read it from the branch name.
- Nobody commits directly to `main`. Changes reach it only through a pull request with `Closes #<issue-number>` in the description.

### Commit messages
- Imperative mood, one logical change per commit: `Add patient entity and migration (#73)`.
- Every commit references its issue number, so it appears on the issue timeline.

