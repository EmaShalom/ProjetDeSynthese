# FULL-STACK DEVELOPMENT PROMPT — COORHOS
## Intelligent Hospital Coordination Platform
### CISSS de l'Outaouais — Complete Front-end + Back-end + Database Application

---

## PROJECT CONTEXT & MISSION

You must develop a complete hospital coordination web application for the CISSS de l'Outaouais (Quebec, Canada). A fully functional HTML/CSS/JavaScript prototype already exists (3,309 lines, 11 interfaces, 35 functions). Your mission is to transform this prototype into a production-ready application with:

- A **robust REST API** (Node.js + Express)
- A **persistent database** (PostgreSQL + Prisma)
- A **modern React front-end** faithful to the prototype
- **Real-time WebSockets** (Socket.io)
- An **AI microservice** (Python + FastAPI)

The application is called **COORHOS**. It centralizes bed management, patient tracking, admissions, inter-facility transfers, room hygiene, and AI-assisted decision-making.

---

## REQUIRED TECH STACK

### Back-end
- **Runtime**: Node.js 20+ with Express.js
- **Database**: PostgreSQL 15+
- **ORM**: Prisma with automatic migrations
- **Real-time**: Socket.io (replacing prototype's setInterval polling)
- **Authentication**: JWT + refresh tokens + RBAC (5 roles)
- **Validation**: Zod for all input schemas
- **PDF Generation**: PDFKit for all 12 reports
- **Logging**: Winston
- **Testing**: Jest + Supertest
- **Password hashing**: bcrypt

### Front-end
- **Framework**: React 18 + Vite
- **Routing**: React Router v6 with role-protected routes
- **Global state**: Zustand
- **UI**: Tailwind CSS — exact color palette from prototype:
  - Primary blue: `#1D6FA4`
  - Green: `#16A34A`
  - Red: `#DC2626`
  - Amber: `#D97706`
  - Purple: `#7C3AED`
  - Dark background (floor display): `#0d1117`
- **Components**: Shadcn/ui as base
- **Charts**: Recharts
- **Forms**: React Hook Form + Zod
- **API queries**: TanStack Query v5
- **Real-time**: Socket.io-client
- **PDF**: triggered via back-end API

### AI (separate microservice)
- **Framework**: Python 3.11 + FastAPI
- **ML**: scikit-learn (RandomForestRegressor + RandomForestClassifier)
- **Dataset**: HDHI (Hindustan Dataset Hospital India — 15,757 patients, 56 variables, available on Kaggle)

### Infrastructure
- **Containerization**: Docker + Docker Compose
- **Environment variables**: dotenv
- **CORS**: configured for front-end origin

---

## FOLDER STRUCTURE

```
cissscoord/
├── backend/
│   ├── src/
│   │   ├── routes/
│   │   │   ├── auth.routes.js
│   │   │   ├── patients.routes.js
│   │   │   ├── beds.routes.js
│   │   │   ├── hygiene.routes.js
│   │   │   ├── transfers.routes.js
│   │   │   ├── dashboard.routes.js
│   │   │   ├── reports.routes.js
│   │   │   ├── station.routes.js
│   │   │   └── ai.routes.js
│   │   ├── controllers/        # Business logic
│   │   ├── middleware/
│   │   │   ├── auth.middleware.js
│   │   │   ├── rbac.middleware.js
│   │   │   └── validate.middleware.js
│   │   ├── services/
│   │   │   ├── pdf.service.js
│   │   │   ├── ai.service.js
│   │   │   └── socket.service.js
│   │   └── socket/
│   │       └── events.js
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── seed.js
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   ├── DashboardPage.jsx
│   │   │   ├── UrgencyPage.jsx
│   │   │   ├── FloorPage.jsx
│   │   │   ├── FloorDisplayPage.jsx
│   │   │   ├── AdmissionsPage.jsx
│   │   │   ├── CoordinatorPage.jsx
│   │   │   ├── TransfersPage.jsx
│   │   │   ├── HygienePage.jsx
│   │   │   ├── MorningStationPage.jsx
│   │   │   ├── AIPage.jsx
│   │   │   └── ReportsPage.jsx
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── Sidebar.jsx
│   │   │   │   └── Topbar.jsx
│   │   │   ├── shared/
│   │   │   │   ├── BedGrid.jsx
│   │   │   │   ├── PatientModal.jsx
│   │   │   │   ├── RiskBadge.jsx
│   │   │   │   └── StatusBadge.jsx
│   │   │   └── ui/             # Shadcn components
│   │   ├── stores/
│   │   │   ├── authStore.js
│   │   │   ├── patientStore.js
│   │   │   └── socketStore.js
│   │   ├── hooks/
│   │   │   ├── useSocket.js
│   │   │   └── useRBAC.js
│   │   ├── api/
│   │   │   └── index.js        # Axios instance + all API calls
│   │   └── lib/
│   │       ├── diagUnitMap.js
│   │       └── riskScore.js
│   └── package.json
├── ai-service/
│   ├── main.py
│   ├── models/
│   │   ├── risk_score.py
│   │   ├── length_of_stay.py
│   │   └── readmission.py
│   └── requirements.txt
└── docker-compose.yml
```

---

## DATABASE — COMPLETE PRISMA SCHEMA

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─────────────────────────────────────────
// USERS & ROLES
// ─────────────────────────────────────────

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String   // bcrypt hash
  name      String
  role      Role
  unit      String?  // "2N" | "2S" | "3N" | "3S" — null = global access
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  auditLogs       AuditLog[]
  stationComments StationComment[]
  hygieneUpdates  HygieneUpdate[]
  attributions    BedAttribution[]
}

enum Role {
  COORDINATOR         // IDENTICAL permissions to BED_MANAGER (business rule)
  BED_MANAGER         // Super user — full access to everything
  UNIT_CHIEF          // Own unit + morning station
  NURSE               // Read-only on own unit + floor display
  HYGIENE             // Hygiene module only
}

// ─────────────────────────────────────────
// PATIENTS
// ─────────────────────────────────────────

model Patient {
  id        String @id @default(cuid())
  mrdNumber Int    @unique  // Medical Record Number (e.g., 316881)

  // Identity
  firstName   String?
  lastName    String?
  dateOfBirth DateTime?
  gender      String    // "M" | "F" | "O"
  ramq        String?   // Quebec health insurance number
  language    String    @default("French")

  // Address
  address    String?
  city       String?
  postalCode String?
  phone      String?
  province   String   @default("Quebec")
  ruralUrban String?  // "R" | "U"

  // Emergency contact
  emergencyContactName   String?
  emergencyContactLink   String?   // Relationship (spouse, child, etc.)
  emergencyContactPhone  String?
  emergencyContactPhone2 String?

  // Clinical data (from HDHI dataset)
  age           Int
  admissionType String  // "E" (Emergency) | "O" (OPD outpatient)

  // Primary diagnosis
  diagnosis String

  // 26 binary diagnosis variables (from HDHI dataset columns)
  stableAngina            Boolean @default(false)
  acs                     Boolean @default(false)
  stemi                   Boolean @default(false)
  atypicalChestPain       Boolean @default(false)
  heartFailure            Boolean @default(false)
  hfref                   Boolean @default(false)
  hfnef                   Boolean @default(false)
  valvular                Boolean @default(false)
  chb                     Boolean @default(false)
  sss                     Boolean @default(false)
  aki                     Boolean @default(false)
  cvaInfract              Boolean @default(false)
  cvaBleed                Boolean @default(false)
  af                      Boolean @default(false)
  vt                      Boolean @default(false)
  psvt                    Boolean @default(false)
  congenital              Boolean @default(false)
  uti                     Boolean @default(false)
  neuroCardiogenicSyncope Boolean @default(false)
  dvt                     Boolean @default(false)
  cardiogenicShock        Boolean @default(false)
  shock                   Boolean @default(false)
  pulmonaryEmbolism       Boolean @default(false)
  chestInfection          Boolean @default(false)
  anaemia                 Boolean @default(false)
  severeAnaemia           Boolean @default(false)

  // Comorbidities
  dm       Boolean @default(false)  // Diabetes mellitus
  htn      Boolean @default(false)  // Hypertension
  cad      Boolean @default(false)  // Coronary artery disease
  ckd      Boolean @default(false)  // Chronic kidney disease
  priorCmp Boolean @default(false)
  smoking  Boolean @default(false)
  alcohol  Boolean @default(false)
  raisedCardiacEnzymes Boolean @default(false)

  // Biological data (from HDHI dataset)
  hb         Float?   // Hemoglobin g/dL
  tlc        Float?   // White blood cell count x1000
  platelets  Int?     // Platelets x1000/µL
  glucose    Int?     // mg/dL
  urea       Int?     // mg/dL
  creatinine Float?   // mg/dL
  bnp        Float?   // pg/mL
  ef         Float?   // Ejection fraction %

  // Stay information
  admitDate          DateTime  @default(now())
  dischargeDate      DateTime?
  predictedDischarge DateTime?  // AI prediction
  stayDays           Int?
  icuDays            Int        @default(0)
  attendingPhysician String?
  careLevel          String?    // "Standard" | "Intensive" | "Enhanced" | "Palliative"
  clinicalNotes      String?

  // Status & outcome
  status  PatientStatus @default(WAITING_EMERGENCY)
  outcome String?       // "DISCHARGE" | "DAMA" | "EXPIRY"

  // Floor location
  unit      String?  // "2N" | "2S" | "3N" | "3S"
  bedNumber Int?

  // Admission source
  admissionSource     String?  // "emergency" | "repatriation" | "direct" | "elective"
  linkedMrdEmergency  Int?     // If internal transfer from emergency
  originFacility      String?  // If repatriation from another hospital

  // Emergency triage
  triageLevel     Int?      // 1-5 (P1 = resuscitation, P5 = non-urgent)
  arrivalDatetime DateTime?
  chiefComplaint  String?

  // AI scores
  riskScore       Int    @default(0)
  riskLevel       String @default("LOW")  // "LOW" | "MEDIUM" | "HIGH"
  readmissionRisk Float?  // 0.0 - 1.0

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  bedHistory   BedHistory[]
  hygieneUpdates HygieneUpdate[]
  auditLogs    AuditLog[]
  transfers    Transfer[]
  attributions BedAttribution[]
}

enum PatientStatus {
  WAITING_EMERGENCY  // On stretcher, waiting for a bed
  ADMITTED           // Hospitalized on a floor unit
  DISCHARGED         // Left the hospital
  TRANSFERRED        // Transferred to another facility
}

// ─────────────────────────────────────────
// BEDS
// ─────────────────────────────────────────

model Bed {
  id        String    @id @default(cuid())
  unit      String    // "2N" | "2S" | "3N" | "3S"
  bedNumber Int
  status    BedStatus @default(AVAILABLE)

  @@unique([unit, bedNumber])

  history        BedHistory[]
  hygieneUpdates HygieneUpdate[]
  attributions   BedAttribution[]
}

enum BedStatus {
  AVAILABLE    // Ready for a new patient
  OCCUPIED     // Patient currently in bed
  TO_CLEAN     // Patient just discharged — awaiting cleaning
  CLEANING     // Cleaning in progress
  READY        // Cleaned and ready
  ATTRIBUTED   // Assigned to incoming emergency patient (not yet arrived)
}

model BedHistory {
  id        String   @id @default(cuid())
  bedId     String
  patientId String?
  action    String   // "ADMITTED" | "DISCHARGED" | "ATTRIBUTED" | "CLEANED"
  timestamp DateTime @default(now())
  userId    String?

  bed     Bed      @relation(fields: [bedId], references: [id])
  patient Patient? @relation(fields: [patientId], references: [id])
}

// ─────────────────────────────────────────
// BED ATTRIBUTIONS (real-time core feature)
// ─────────────────────────────────────────
// Created when coordinator assigns a bed to an emergency patient.
// Triggers Socket.io notification on floor display.
// Deleted when patient is officially admitted to the floor.

model BedAttribution {
  id           String   @id @default(cuid())
  patientId    String
  unit         String
  bedNumber    Int
  stretcherNum Int      // Originating stretcher number
  assignedAt   DateTime @default(now())
  assignedBy   String   // userId of coordinator

  patient        Patient @relation(fields: [patientId], references: [id])
  bed            Bed     @relation(fields: [bedId], references: [id])
  bedId          String
  assignedByUser User    @relation(fields: [assignedBy], references: [id])
}

// ─────────────────────────────────────────
// HYGIENE
// ─────────────────────────────────────────

model HygieneUpdate {
  id        String    @id @default(cuid())
  bedId     String
  patientId String?
  status    BedStatus
  comment   String?   // Visible to super users only
  updatedBy String
  updatedAt DateTime  @default(now())

  bed     Bed      @relation(fields: [bedId], references: [id])
  patient Patient? @relation(fields: [patientId], references: [id])
  user    User     @relation(fields: [updatedBy], references: [id])
}

// ─────────────────────────────────────────
// TRANSFERS & REPATRIATIONS
// ─────────────────────────────────────────

model Transfer {
  id                   String       @id @default(cuid())
  type                 TransferType

  patientId            String?
  patientExternalId    String?  // MRD at origin facility

  facility             String   // e.g., "CH Gatineau", "CHUM Montreal"

  targetUnit           String?  // Destination unit (incoming repatriation)
  currentUnit          String?  // Origin unit (outgoing transfer)
  currentBed           Int?
  reservedBed          Int?

  scheduledDatetime    DateTime
  actualDatetime       DateTime?

  diagnosis            String?
  reason               String?

  status               TransferStatus @default(PLANNED)
  bedReservationStatus String?  // "reserved" | "none" | "planned"

  contactPhone         String?
  notes                String?

  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt
  createdBy  String?

  patient Patient? @relation(fields: [patientId], references: [id])
}

enum TransferType {
  REPATRIATION_IN   // Incoming repatriation from another facility
  TRANSFER_OUT      // Outgoing transfer to another facility
}

enum TransferStatus {
  PLANNED     // Scheduled
  CONFIRMED   // Transport confirmed
  IN_TRANSIT  // Currently in transit
  COMPLETED   // Arrived / Departed
  CANCELLED   // Cancelled
}

// ─────────────────────────────────────────
// MORNING STATION (daily coordination meeting)
// ─────────────────────────────────────────

model StationComment {
  id       String   @id @default(cuid())
  unit     String   // "2N" | "2S" | "3N" | "3S" | "GLOBAL"
  content  String
  date     DateTime @default(now())
  authorId String

  author User @relation(fields: [authorId], references: [id])
}

// ─────────────────────────────────────────
// AUDIT LOG
// ─────────────────────────────────────────

model AuditLog {
  id        String   @id @default(cuid())
  userId    String?
  patientId String?
  action    String   // "BED_ATTRIBUTED" | "PATIENT_ADMITTED" | "PATIENT_DISCHARGED"
                     // | "TRANSFER_CREATED" | "RECORD_UPDATED" | etc.
  details   Json?    // Before/after data snapshot
  ipAddress String?
  timestamp DateTime @default(now())

  user    User?    @relation(fields: [userId], references: [id])
  patient Patient? @relation(fields: [patientId], references: [id])
}
```

---

## REST API — ALL ENDPOINTS

### Authentication
```
POST   /api/auth/login           Body: { email, password } → { token, refreshToken, user }
POST   /api/auth/logout
POST   /api/auth/refresh         Body: { refreshToken } → { token }
GET    /api/auth/me              → current user
```

### Patients
```
GET    /api/patients             Filters: unit, status, risk, search (MRD or name)
GET    /api/patients/:id         Full record with all relations
POST   /api/patients             New emergency admission
PUT    /api/patients/:id         Update record (full access based on role)
DELETE /api/patients/:id         Archive (admin only)

GET    /api/patients/waiting     Emergency stretcher patients awaiting a bed
                                 Includes: assignedBed and assignedAt if attributed
GET    /api/patients/unit/:unit  Admitted patients in a specific unit
POST   /api/patients/:id/admit-floor   { unit, bedNumber, attendingPhysician, careLevel }
POST   /api/patients/:id/discharge     { outcome, dischargeDate }
                                       → automatically sets bed to TO_CLEAN
POST   /api/patients/:id/transfer-out  { transferId }
```

### Beds & Attributions
```
GET    /api/beds                 Filters: unit, status
GET    /api/beds/unit/:unit      Beds + occupying patients + active attributions
GET    /api/beds/availability    Summary per unit: { unit, total, occupied, free, attributed }

POST   /api/beds/attribute       CORE REAL-TIME FEATURE — BED ATTRIBUTION
                                 Body: { patientId, unit, bedNumber }
                                 → creates BedAttribution record
                                 → emits Socket.io event "bed:attributed"
                                 → updates waiting patient (assignedBed, assignedAt)
                                 → creates AuditLog

DELETE /api/beds/attribution/:id Cancel an attribution
PUT    /api/beds/:unit/:number/status  { status }  ← Used by hygiene team
```

### Hygiene
```
GET    /api/hygiene              All beds with TO_CLEAN, CLEANING, READY status + patient info
PUT    /api/hygiene/:bedId       Body: { status, comment }
                                 → emits Socket.io event "hygiene:updated"
```

### Transfers
```
GET    /api/transfers            Filters: type, status, date
GET    /api/transfers/today      Today's movements (for morning station)
POST   /api/transfers            Create transfer or repatriation
PUT    /api/transfers/:id        Update
PUT    /api/transfers/:id/status Body: { status }
DELETE /api/transfers/:id
```

### Dashboard
```
GET    /api/dashboard/kpis            { occupancyRate, waitingCount, highRiskCount,
                                         dischargesThisWeek, avgStayDays }
GET    /api/dashboard/occupancy       Occupancy rate per unit
GET    /api/dashboard/admissions-week Admissions per day (last 7 days)
GET    /api/dashboard/diagnostics     Top diagnoses distribution
GET    /api/dashboard/outcomes        DISCHARGE / DAMA / EXPIRY breakdown
```

### Morning Station
```
GET    /api/station/summary      Everything needed for morning coordination meeting
GET    /api/station/comments     Comments of the day (role-restricted)
POST   /api/station/comments     Body: { unit, content }
PUT    /api/station/comments/:id Body: { content }
```

### AI & Predictions
```
GET    /api/ai/predictions       Predictions for all active patients
GET    /api/ai/patient/:id       Predictions for a specific patient
GET    /api/ai/saturation        48h occupancy projection per unit
POST   /api/ai/risk-score        Body: { ...patientData } → { score, level, factors }
```

### PDF Reports
```
GET    /api/reports/pdf/beds           Daily bed status report
GET    /api/reports/pdf/admissions     Admissions report
GET    /api/reports/pdf/discharges     Discharges report
GET    /api/reports/pdf/waiting        Waiting patients report
GET    /api/reports/pdf/transfers      Transfers & repatriations report
GET    /api/reports/pdf/critical       Critical patients report
GET    /api/reports/pdf/weekly         Weekly occupancy report
GET    /api/reports/pdf/hygiene        Hygiene & sanitation report
GET    /api/reports/pdf/ai             AI & predictions report
GET    /api/reports/pdf/station        Morning station report
GET    /api/reports/pdf/dama           DAMA & readmissions report
GET    /api/reports/pdf/complete       Full weekly report
```

---

## SOCKET.IO EVENTS — REAL-TIME

### Server → Clients (emit)
```javascript
// Bed assigned to an emergency patient
// Triggers floor display notification instantly
socket.to(`unit:${unit}`).emit('bed:attributed', {
  patientId, mrdNumber, unit, bedNumber, stretcherNum,
  diagnosis, risk, age, gender, assignedAt, assignedByName
});

// Patient officially admitted to floor
socket.emit('patient:admitted', { patientId, unit, bedNumber });

// Patient discharged → bed goes to TO_CLEAN
socket.emit('patient:discharged', { patientId, unit, bedNumber });

// Hygiene status updated
socket.to(`unit:${unit}`).emit('hygiene:updated', { bedId, unit, bedNumber, status });

// New transfer or repatriation created
socket.emit('transfer:created', { transfer });

// AI alert (imminent saturation)
socket.emit('ai:alert', { unit, type: 'SATURATION', message, severity: 'HIGH' });
```

### Client → Server (listen)
```javascript
// Join a unit room to receive targeted notifications
socket.emit('join:unit', { unit: '2N' });

// Floor display subscribes to attributions for its unit
socket.on('bed:attributed', (data) => {
  // Show notification card in right column
  // Update bed grid (bed turns purple/pulsing)
  // Update free beds list
});
```

---

## CRITICAL BUSINESS RULES — IMPLEMENT SERVER-SIDE

These rules are coded in the prototype and must be enforced by the API — they cannot be bypassed:

### 1. Diagnosis → Unit Matching (MANDATORY)
```javascript
const DIAG_UNIT_MAP = {
  // 2nd Floor North — Cardiology
  'ACS':'2N', 'STEMI':'2N', 'STABLE ANGINA':'2N', 'AF':'2N', 'VT':'2N',
  'VALVULAR':'2N', 'ATYPICAL CHEST PAIN':'2N', 'CHB':'2N', 'SSS':'2N',
  'PSVT':'2N', 'HEART FAILURE':'2N',
  // 2nd Floor South — Intensive Care
  'HFREF':'2S', 'HFNEF':'2S', 'CARDIOGENIC SHOCK':'2S', 'SHOCK':'2S',
  'PULMONARY EMBOLISM':'2S', 'DVT':'2S', 'INFECTIVE ENDOCARDITIS':'2S',
  // 3rd Floor North — Nephrology
  'AKI':'3N', 'UTI':'3N',
  // 3rd Floor South — General Medicine
  'CVA INFRACT':'3S', 'CVA BLEED':'3S', 'CHEST INFECTION':'3S',
  'NEURO CARDIOGENIC SYNCOPE':'3S', 'ANAEMIA':'3S', 'CONGENITAL':'3S', 'Other':'3S'
};
// Validate in POST /api/beds/attribute AND POST /api/patients/:id/admit-floor
```

### 2. RBAC — Role Permissions
```javascript
const PERMISSIONS = {
  COORDINATOR:  ['all'],           // IDENTICAL to BED_MANAGER — explicit business rule
  BED_MANAGER:  ['all'],           // Super user
  UNIT_CHIEF:   ['unit:own', 'station:read', 'station:comment', 'reports:read'],
  NURSE:        ['unit:own:read',  'floor-display:read'],
  HYGIENE:      ['hygiene:read',   'hygiene:update'],
};
// COORDINATOR and BED_MANAGER have exactly the same rights — this is by design
```

### 3. Automatic Bed Lifecycle
```
Patient discharged → BedStatus automatically set to TO_CLEAN
TO_CLEAN → CLEANING   (by hygiene team)
CLEANING → READY      (by hygiene team)
READY    → AVAILABLE  (automatic or confirmed)
NEVER:  OCCUPIED → AVAILABLE directly (strict rule — always goes through cleaning)
```

### 4. Real-Time Attribution Flow
```
POST /api/beds/attribute :
  1. Verify bed is not OCCUPIED, TO_CLEAN, CLEANING, or ATTRIBUTED
  2. Create BedAttribution record
  3. Set BedStatus → ATTRIBUTED
  4. Update Patient.assignedBed and assignedAt
  5. Emit socket event "bed:attributed" to target unit room
  6. Create AuditLog entry
  Response time target: < 200ms
```

### 5. Repatriation Saturation Alert
```
On POST /api/transfers where type = REPATRIATION_IN:
  If target unit occupancy > 80%:
    → Add warning object in response body
    → Emit socket event "ai:alert" with severity "HIGH"
```

### 6. AI Risk Score Algorithm
```javascript
// This exact algorithm is validated and used in the prototype
function calculateRiskScore(patient) {
  let score = 0;
  const factors = [];
  if (patient.cardiogenicShock)       { score += 3; factors.push('Cardiogenic shock (+3)'); }
  if (patient.raisedCardiacEnzymes)   { score += 2; factors.push('Raised cardiac enzymes (+2)'); }
  if (patient.shock)                  { score += 2; factors.push('Shock (+2)'); }
  if (patient.dm)                     { score += 1; factors.push('Diabetes (+1)'); }
  if (patient.htn)                    { score += 1; factors.push('Hypertension (+1)'); }
  if (patient.ckd)                    { score += 1; factors.push('Chronic kidney disease (+1)'); }
  if (patient.age > 70)               { score += 1; factors.push('Age > 70 (+1)'); }
  const level = score >= 4 ? 'HIGH' : score >= 2 ? 'MEDIUM' : 'LOW';
  return { score, level, factors };
}
// Calculated at admission and recalculated on every comorbidity update
```

---

## AI MICROSERVICE — PYTHON / FASTAPI

```python
# ai-service/main.py

from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np

app = FastAPI()

class PatientData(BaseModel):
    age: int
    dm: bool; htn: bool; cad: bool; ckd: bool
    cardiogenic_shock: bool; raised_cardiac_enzymes: bool; shock: bool
    hb: float = None; glucose: int = None
    creatinine: float = None; ef: float = None
    # + all binary diagnosis variables from HDHI dataset

@app.post("/predict/risk-score")
def predict_risk(patient: PatientData):
    # Rule-based algorithm (see business rules section)
    score = 0; factors = []
    if patient.cardiogenic_shock:        score += 3; factors.append("Cardiogenic shock (+3)")
    if patient.raised_cardiac_enzymes:   score += 2; factors.append("Raised enzymes (+2)")
    if patient.shock:                    score += 2; factors.append("Shock (+2)")
    if patient.dm:                       score += 1; factors.append("Diabetes (+1)")
    if patient.htn:                      score += 1; factors.append("Hypertension (+1)")
    if patient.ckd:                      score += 1; factors.append("CKD (+1)")
    if patient.age > 70:                 score += 1; factors.append("Age > 70 (+1)")
    level = "HIGH" if score >= 4 else "MEDIUM" if score >= 2 else "LOW"
    return {"score": score, "level": level, "factors": factors}

@app.post("/predict/length-of-stay")
def predict_los(patient: PatientData):
    # RandomForestRegressor trained on HDHI dataset
    # Features: age, dm, htn, cad, ckd, cardiogenic_shock,
    #           raised_cardiac_enzymes, hb, glucose, creatinine, ef
    #           + all 26 binary diagnosis columns
    # Target: DURATION OF STAY (HDHI column name)
    model = joblib.load("models/los_model.pkl")
    features = extract_features(patient)
    prediction = model.predict([features])[0]
    return {"predicted_days": round(float(prediction), 1)}

@app.post("/predict/readmission-risk")
def predict_readmission(patient: PatientData):
    # Particularly relevant for DAMA (left against medical advice) patients
    model = joblib.load("models/readmission_model.pkl")
    features = extract_features(patient)
    prob = model.predict_proba([features])[0][1]
    return {"probability": round(float(prob), 3), "high_risk": prob > 0.35}

@app.get("/predict/saturation")
def predict_saturation(beds_state: dict):
    # Return 24h and 48h occupancy projections per unit
    # Based on predicted discharges and incoming transfers
    return {
        "units": {
            "2N": {"current_pct": 60, "h24_pct": 65, "h48_pct": 70},
            "2S": {"current_pct": 66, "h24_pct": 73, "h48_pct": 80},
            "3N": {"current_pct": 80, "h24_pct": 87, "h48_pct": 93},  # ← Alert
            "3S": {"current_pct": 66, "h24_pct": 66, "h48_pct": 60},
        }
    }
```

---

## FRONT-END PAGES — 11 REACT PAGES

### Shared Layout (all pages)
```jsx
// Sidebar (220px, background #334155)
// - CisssCoord logo
// - Current week display
// - Navigation groups: Dashboards / Clinical Interfaces / AI / Reports
// - User info (name + role badge) at bottom

// Topbar (56px)
// - Current page title
// - Urgent alerts badge (red, clickable)
// - Live date and time

// All routes protected by role via useRBAC hook
// Unauthenticated users redirected to /login
```

### Page 1 — DashboardPage
5 KPI cards in grid (occupancy rate, waiting patients, high risk count, discharges this week, avg stay days). 7-day admissions bar chart (Recharts). Per-unit occupancy horizontal bars. Top diagnoses list. Outcomes pie/donut. Auto-refresh every 30s via TanStack Query.

### Page 2 — UrgencyPage
Stretcher card grid, auto-sorted by priority (HIGH → MEDIUM → LOW). Each card: MRD, age/gender, diagnosis, target unit, wait time (live counter in red if > 12h), risk badge. **If `assignedBed` exists on patient: show green badge "✅ Bed X assigned (HH:MM)" instead of "Assign" button.** Full sortable list table below.

### Page 3 — FloorPage
4 unit tabs. Per-unit KPIs. 5×3 grid of 15 beds with color coding (blue=stable, red=high risk, amber=medium risk, green=available, purple pulsing=attributed). Click bed → full patient modal. **"Patients incoming from emergency" section** fed by Socket.io `bed:attributed` event. Patient table with "Record" button.

### Page 4 — FloorDisplayPage (nursing station wall display)
**Dark theme required** (`background: #0d1117`). 3-column layout:
- **Left**: 15-bed grid — free (green), occupied (blue/red/amber), attributed (purple pulsing + "ATTRIBUTED" badge + patient info)
- **Center**: numbered list of free beds, updates live
- **Right**: **EMPTY by default** → purple notification card appears instantly via Socket.io event `bed:attributed` showing: 🔔 large bed number + MRD + age + gender + diagnosis + risk level + attribution time
Unit selector buttons at bottom (2N, 2S, 3N, 3S). 30s countdown visible. Live clock.

### Page 5 — AdmissionsPage
3 sub-tabs:
- **Emergency Admission**: first name, last name, DOB, gender, RAMQ + full address (street, city, postal, phone) + emergency contact (name, relationship, phone x2) + notes. Submit → POST /api/patients → auto-generates MRD.
- **Floor Admission**: diagnosis → auto-fills unit → available beds list + visual bed location grid (appears on bed selection, highlights chosen bed in green). Confirm → POST /api/patients/:id/admit-floor.
- **Edit Record**: search bar (MRD, name, unit, status) → results table → 4-tab form (Identity & Contact, Clinical & Diagnosis, Stay & Status, Biological Data). Save → PUT /api/patients/:id → real-time propagation via Socket.io.

### Page 6 — CoordinatorPage
4 clickable unit cards (→ FloorPage with pre-selected tab). Integrated transfers/repatriations module (same content as TransfersPage).

### Page 7 — TransfersPage
2 tabs. Incoming repatriations: detailed cards (facility, diagnosis, arrival time, bed status in color). Outgoing transfers: destination, reason, departure time. "+ New movement" button → full modal form → POST /api/transfers.

### Page 8 — HygienePage
3 counters (To Clean / In Progress / Ready). List with inline dropdown selects. Update → PUT /api/hygiene/:bedId → Socket.io event → real-time update for bed manager.

### Page 9 — MorningStationPage
Blue header with 4 key figures. 4 unit availability cards (⚠️ if > 80%). Today's transfers/repatriations list. Previous day report. Per-unit issues. Comments zone (role-restricted — COORDINATOR, BED_MANAGER, UNIT_CHIEF only).

### Page 10 — AIPage
4 AI KPI cards. Critical alerts (red). Recommendations (green). High-risk patient cards. 48h occupancy projection progress bars (green < 70%, amber < 85%, red ≥ 85%).

### Page 11 — ReportsPage
12 report cards in 3-column grid. Each card: icon + title + short description + "👁 Preview" button (modal) + "⬇ PDF" button (GET /api/reports/pdf/:type → triggers download). PDFs generated server-side via PDFKit.

---

## DATABASE SEED

```javascript
// prisma/seed.js

const users = [
  { email: 'coordinator@cisss.qc.ca',  password: 'Admin2025!', name: 'Marie Dupont',    role: 'COORDINATOR' },
  { email: 'bedmanager@cisss.qc.ca',   password: 'Admin2025!', name: 'Jean Tremblay',   role: 'BED_MANAGER' },
  { email: 'chief2n@cisss.qc.ca',      password: 'Admin2025!', name: 'Dr. Cote',        role: 'UNIT_CHIEF', unit: '2N' },
  { email: 'chief2s@cisss.qc.ca',      password: 'Admin2025!', name: 'Dr. Leblanc',     role: 'UNIT_CHIEF', unit: '2S' },
  { email: 'chief3n@cisss.qc.ca',      password: 'Admin2025!', name: 'Dr. Martin',      role: 'UNIT_CHIEF', unit: '3N' },
  { email: 'chief3s@cisss.qc.ca',      password: 'Admin2025!', name: 'Dr. Tremblay',    role: 'UNIT_CHIEF', unit: '3S' },
  { email: 'nurse2n@cisss.qc.ca',      password: 'Admin2025!', name: 'Julie Bergeron',  role: 'NURSE', unit: '2N' },
  { email: 'hygiene@cisss.qc.ca',      password: 'Admin2025!', name: 'Sophie Lavoie',   role: 'HYGIENE' },
];

// 60 patients from HDHI dataset + 12 emergency stretcher patients
// Import from week_simulation.json (available in prototype)
// Distribute across 4 units: 2N(9), 2S(10), 3N(12), 3S(10) — 19 free beds total

// 60 beds (15 per unit × 4 units) initialized with matching statuses

// Sample transfers: 3 incoming repatriations + 2 outgoing transfers
```

---

## DOCKER COMPOSE

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: cissscoord
      POSTGRES_USER: cisss
      POSTGRES_PASSWORD: cisss2025
    ports: ["5432:5432"]
    volumes: ["postgres_data:/var/lib/postgresql/data"]

  backend:
    build: ./backend
    ports: ["3001:3001"]
    environment:
      DATABASE_URL: postgresql://cisss:cisss2025@postgres:5432/cissscoord
      JWT_SECRET: cissscoord_jwt_secret_2025
      AI_SERVICE_URL: http://ai-service:8000
      PORT: 3001
    depends_on: [postgres]

  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    environment:
      VITE_API_URL: http://localhost:3001
      VITE_SOCKET_URL: http://localhost:3001
    depends_on: [backend]

  ai-service:
    build: ./ai-service
    ports: ["8000:8000"]

volumes:
  postgres_data:
```

---

## TOP PRIORITY IMPLEMENTATION NOTES

**1. Real-time bed attribution is the heart of this system.**
The Socket.io event `bed:attributed` must simultaneously update:
- The emergency stretcher card (replace "Assign" button with green badge)
- The floor display right column (show notification card)
- The floor bed grid (bed turns purple/pulsing)
- The free beds list (remove that bed)
Target latency: < 500ms end-to-end.

**2. COORDINATOR = BED_MANAGER — no hierarchy between them.**
These two roles have 100% identical permissions. This is an explicit business rule from on-the-ground observation. Do not create any permission difference between them.

**3. Bed lifecycle is strict — never skip cleaning.**
A bed can NEVER go from OCCUPIED to AVAILABLE directly.
Always: OCCUPIED → TO_CLEAN → CLEANING → READY → AVAILABLE.
Enforce this as middleware on the bed status update endpoint.

**4. Floor display right column starts empty.**
The notification column shows cards ONLY when a bed has been attributed from the emergency interface. Never show patients who are still on stretchers without an attributed bed. This is the key design decision that eliminates inter-unit phone calls.

**5. HDHI Dataset.**
Available on Kaggle (search "HDHI Admission data"). It contains 15,757 rows and 56 columns. Key columns for ML: `AGE`, `GENDER`, `DURATION OF STAY`, `D.N.A` (outcome — DISCHARGE/DAMA/EXPIRY), `TYPE OF ADMISSION-EMERGENCY/OPD`, and the 26 binary diagnosis columns. Train the RandomForest models on this data for length-of-stay prediction and DAMA readmission risk.

**6. Security requirements.**
All endpoints except `/api/auth/login` require a valid JWT. Unit-specific endpoints (floor, hygiene) verify that the authenticated user has access to the requested unit (except COORDINATOR and BED_MANAGER who have global access). All mutations create an AuditLog entry.

**7. Audit everything.**
Every bed attribution, admission, discharge, transfer creation, and record modification must create an AuditLog with: userId, patientId, action string, JSON snapshot of before/after data, IP address, and timestamp.

---

## DELIVERABLES CHECKLIST

- [ ] `docker-compose up` starts the full stack in a single command
- [ ] `npx prisma db seed` initializes DB with 8 users + simulation data
- [ ] All API endpoints auto-documented via Swagger/OpenAPI
- [ ] React front-end visually faithful to prototype (colors, layout, behaviors)
- [ ] Socket.io working: emergency attribution → floor notification in < 500ms
- [ ] All 12 PDF reports downloadable
- [ ] AI microservice returning risk scores and length-of-stay predictions
- [ ] Jest tests for critical endpoints (attribution, admission, discharge)
- [ ] README with setup instructions
