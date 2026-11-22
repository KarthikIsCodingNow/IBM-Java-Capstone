# User Stories - Smart Clinic Management System

## Project Overview
The **Smart Clinic Management System** is a full-stack healthcare web application designed to digitize and streamline administrative, medical, and patient interactions. The system provides role-based access for three primary user personas: **Admin**, **Doctor**, and **Patient**.

---

## User Personas

1. **Admin**: Responsible for onboarding medical personnel, managing doctor profiles and availability schedules, and monitoring clinical analytics.
2. **Doctor**: Medical practitioner who logs in to inspect scheduled appointments, review patient details, update appointment status, and issue digital prescriptions.
3. **Patient**: Clinic patient who registers, authenticates, browses available doctors by specialty and slot time, books appointments, and views medical records and prescriptions.

---

## Epic 1: Admin Management & System Governance

### US 1.1: Admin Authentication
**As an** Admin,  
**I want to** log in with my username and password,  
**So that I can** access the administrative dashboard securely and manage clinic operations.
- **Acceptance Criteria:**
  - `POST /admin` authenticates username and password against the `admin` table.
  - Successful authentication returns a valid signed JWT token (HTTP 200).
  - Invalid credentials return HTTP 401 Unauthorized with appropriate error messages (`"Password does not match"` or `"invalid email id"`).
  - The frontend redirects authenticated admins to `/adminDashboard/{token}`.

### US 1.2: Add Doctor Profile & Availability
**As an** Admin,  
**I want to** register new doctors with their specialties and available time slots,  
**So that** patients can book appointments with them.
- **Acceptance Criteria:**
  - `POST /doctor/{token}` accepts doctor details including `name`, `specialty`, `email`, `phone`, `password`, and `availableTimes` list.
  - Verifies that the requesting token belongs to an authorized `admin`.
  - Validates constraints (valid email format, 10-digit phone number, name between 3-100 characters).
  - Returns HTTP 201 Created on success or HTTP 409 Conflict if the doctor's email is already registered.

### US 1.3: Update Doctor Details
**As an** Admin,  
**I want to** modify a doctor's contact information, specialty, or consultation hours,  
**So that** the doctor's directory remains accurate.
- **Acceptance Criteria:**
  - `PUT /doctor/{token}` updates existing doctor records by ID.
  - Verifies admin authorization token.
  - Returns HTTP 200 OK on success or HTTP 404 Not Found if the doctor does not exist.

### US 1.4: Remove Doctor Profile
**As an** Admin,  
**I want to** delete a doctor from the system,  
**So that** former or inactive doctors no longer appear in the directory.
- **Acceptance Criteria:**
  - `DELETE /doctor/{id}/{token}` deletes the doctor and associated appointments in cascading order.
  - Returns HTTP 201 Created on successful deletion or HTTP 404 Not Found.

### US 1.5: System Reporting & Analytics
**As an** Admin,  
**I want to** run stored procedure reports on appointment statistics and doctor workloads,  
**So that I can** make data-driven staffing and operational decisions.
- **Acceptance Criteria:**
  - Execute `GetDailyAppointmentReportByDoctor` to view daily appointment schedules for any doctor.
  - Execute `GetDoctorWithMostPatientsByMonth` and `GetDoctorWithMostPatientsByYear` to identify top-performing doctors based on patient counts.

---

## Epic 2: Doctor Schedule & Consultation Management

### US 2.1: Doctor Authentication
**As a** Doctor,  
**I want to** sign in to my account with my registered email and password,  
**So that I can** view my patient appointments securely.
- **Acceptance Criteria:**
  - `POST /doctor/login` validates credentials against the `doctor` table.
  - Returns a JWT token on success or HTTP 401 Unauthorized on failure.
  - Doctor is navigated to `/doctorDashboard/{token}` upon successful login.

### US 2.2: View Assigned Appointments & Daily Schedule
**As a** Doctor,  
**I want to** view appointments booked by patients for a selected date,  
**So that I can** plan my clinical consultations.
- **Acceptance Criteria:**
  - `GET /appointment/{date}/{patientName}/{token}` retrieves appointments filtered by doctor ID, date, and optional patient name search.
  - Validates doctor authentication token.
  - Displays patient name, scheduled time, and status (`0` = Scheduled, `1` = Completed).

### US 2.3: Check Real-time Slot Availability
**As a** Doctor,  
**I want to** see which of my consultation slots are free or booked on a given date,  
**So that I can** avoid double-booking.
- **Acceptance Criteria:**
  - `GET /doctor/availability/{user}/{doctorId}/{date}/{token}` calculates remaining available slots by removing booked appointments from `availableTimes`.
  - Returns an array of unbooked time slot strings (e.g., `["09:00-10:00", "14:00-15:00"]`).

### US 2.4: Issue Digital Prescription
**As a** Doctor,  
**I want to** create and attach a digital prescription to an appointment,  
**So that** the patient has a verifiable record of prescribed medication and dosages.
- **Acceptance Criteria:**
  - `POST /prescription/{token}` saves a prescription document into MongoDB (`prescriptions` collection).
  - Validates required fields: `patientName`, `appointmentId`, `medication`, and `dosage`.
  - Returns HTTP 201 Created on successful creation.

### US 2.5: Mark Appointment Status
**As a** Doctor,  
**I want to** update the status of an appointment to completed,  
**So that** the appointment history reflects treated visits.
- **Acceptance Criteria:**
  - `PUT /appointment/{token}` updates appointment status code (e.g., status changed to `1` for completed).

---

## Epic 3: Patient Onboarding, Search & Booking

### US 3.1: Patient Registration (Sign Up)
**As a** new Patient,  
**I want to** register an account with my personal and contact information,  
**So that I can** access the clinic's digital services.
- **Acceptance Criteria:**
  - `POST /patient` accepts `name`, `email`, `password`, `phone`, and `address`.
  - Validates email format, minimum 6-character password, and 10-digit phone number.
  - Verifies that email and phone number are unique in the database.
  - Returns HTTP 201 Created upon successful registration.

### US 3.2: Patient Login & Session
**As a** Patient,  
**I want to** log in with my email and password,  
**So that I can** access my appointments and booking dashboard.
- **Acceptance Criteria:**
  - `POST /patient/login` authenticates credentials and returns a JWT token.
  - The client stores the token in `sessionStorage` and navigates to `/pages/loggedPatientDashboard.html`.

### US 3.3: Browse & Filter Doctors
**As a** Patient,  
**I want to** filter doctors by specialty, consultation time (AM/PM), or name,  
**So that I can** find the most suitable doctor for my medical needs.
- **Acceptance Criteria:**
  - `GET /doctor` returns a list of all active doctors.
  - `GET /doctor/filter/{name}/{time}/{speciality}` supports combined or standalone filtering (e.g., `/doctor/filter/null/am/Cardiology`).
  - Passwords and sensitive fields are excluded from the response payload (`@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)`).

### US 3.4: Book an Appointment
**As a** Patient,  
**I want to** select an available doctor time slot and book an appointment,  
**So that I can** secure consultation time with the physician.
- **Acceptance Criteria:**
  - `POST /appointment/{token}` verifies that the selected `appointmentTime` is in the future.
  - Checks doctor slot availability to prevent conflicting bookings.
  - Persists the appointment in MySQL with initial status `0` (Scheduled).
  - Returns HTTP 201 Created on success.

### US 3.5: View Booked Appointments & Medical Prescriptions
**As a** Patient,  
**I want to** view my upcoming and historical appointments and associated prescriptions,  
**So that I can** track my clinical visits and medication history.
- **Acceptance Criteria:**
  - `GET /patient/{id}/patient/{token}` returns an array of `AppointmentDTO` items for the patient.
  - `GET /patient/filter/{condition}/{name}/{token}` filters appointments by `past` or `future` status.
  - `GET /prescription/{appointmentId}/{token}` fetches prescription details from MongoDB for the selected appointment.
