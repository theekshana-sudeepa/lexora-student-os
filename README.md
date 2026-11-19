<div align="center">

# Lexora Student OS

**An intelligent student workspace and productivity operating system featuring hardware-enforced license verification, interactive focus timers, dynamic exam countdowns, and academic performance predictors.**

[![Next.js](https://img.shields.io/badge/Next.js-16.1.6-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.2-blue?style=for-the-badge&logo=react)](https://react.dev/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-green?style=for-the-badge&logo=mongodb)](https://www.mongodb.com/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-CSS%20v4-38bdf8?style=for-the-badge&logo=tailwindcss)](https://tailwindcss.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](#license)

</div>

---

## 📌 Executive Overview

**Lexora Student OS** is a unified digital productivity suite tailored for modern academic workflows. Engineered with Next.js App Router and React 19, the platform combines client-side utility applications with a server-side license entitlement verification subsystem. 

### Key Capabilities
- **Hardware-Enforced Licensing:** Multi-device restriction enforcing a maximum device quota (default: 2 active devices) mapped to persistent client hardware UUIDs.
- **Administrative Issuance API:** Protected endpoint guarded by secure administrative secret tokens (`x-admin-secret`) to programmatically issue unique cryptographic license credentials.
- **Focus Pomodoro Timer:** Dual-mode productivity timer supporting focused study blocks (25 min) and restorative breaks (5 min) with active session state controllers.
- **Exam War Room:** Real-time deadline countdown tracker with dynamic urgency color gradients (Critical, Warning, Relaxed) and persistent local storage caching.
- **GPA & Grade Predictor:** Weighted assessment average calculator that forecasts cumulative performance metrics based on dynamic course module weightings.
- **Device Management & Session Deactivation:** User-controlled device unbinding interface for managing active workstation sessions.

---

## 🏗️ System Architecture

```mermaid
graph TD
    subgraph ClientLayer ["Client Browser / Workstation"]
        UI["Student Dashboard & Utility Apps"]
        Guard["LicenseGuard Wrapper"]
        DevID["UUID Device Fingerprint"]
        Storage[("Local Cache / Storage")]
    end

    subgraph ServerLayer ["Next.js Serverless Runtime"]
        VerifyAPI["/api/verify (POST)"]
        AdminAPI["/api/admin/create-license (POST)"]
        SeedAPI["/api/seed (GET)"]
        DBConn["Mongoose Connection Pool"]
    end

    subgraph DataLayer ["Database Tier"]
        Mongo[("MongoDB Atlas")]
    end

    UI --> Guard
    Guard --> DevID
    Guard --> Storage
    Guard -- "Verify Credentials (Key + DeviceID)" --> VerifyAPI
    AdminAPI -- "Secret Header Check (x-admin-secret)" --> DBConn
    VerifyAPI --> DBConn
    SeedAPI --> DBConn
    DBConn --> Mongo
```

---

## 🔄 License Verification & Device Binding Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Student Workstation
    participant Guard as LicenseGuard (Client)
    participant API as /api/verify
    participant DB as MongoDB Atlas

    User->>Guard: Open Protected Module (/timer, /exams, /grades)
    Guard->>Guard: Check localStorage for active session
    alt Session Validated Locally
        Guard-->>User: Render Protected Application
    else Unverified Device
        Guard-->>User: Display Security Verification Modal
        User->>Guard: Submit License Key
        Guard->>API: POST { licenseKey, deviceId }
        API->>DB: Query License Record
        alt License Missing or Inactive
            DB-->>API: Not Found / Suspended
            API-->>Guard: 400/403 Error Response
            Guard-->>User: Display Error Banner
        else Device Already Registered
            DB-->>API: Match Found in devices[]
            API-->>Guard: 200 OK (Welcome Back)
            Guard-->>User: Grant Access & Cache Session
        else New Device (Under Limit)
            API->>DB: Push deviceId & Save
            DB-->>API: Saved
            API-->>Guard: 200 OK (New Device Bound)
            Guard-->>User: Grant Access & Cache Session
        else Device Limit Exceeded
            API-->>Guard: 403 Forbidden (Quota Exceeded)
            Guard-->>User: Display Limit Reached Notice
        end
    end
```

---

## 📂 Repository Directory Layout

```
lexora-student-os/
├── public/                     # Static media and vector icons
│   ├── next.svg
│   └── vercel.svg
├── src/
│   ├── app/                    # Next.js App Router routes & endpoints
│   │   ├── api/
│   │   │   ├── admin/
│   │   │   │   └── create-license/  # Protected license creation endpoint
│   │   │   ├── seed/                # Development fixture seeder
│   │   │   └── verify/              # Device binding & verification endpoint
│   │   ├── exams/              # Exam War Room countdown page
│   │   ├── grades/             # Weighted GPA calculator page
│   │   ├── settings/           # Device manager and session settings
│   │   ├── timer/              # Standalone protected Pomodoro timer
│   │   ├── globals.css         # Tailwind CSS styling and theme definitions
│   │   ├── layout.js           # Root layout and font configuration
│   │   └── page.js             # Welcome screen and security check entrance
│   ├── components/
│   │   ├── FocusTimer.js       # Pomodoro countdown widget
│   │   └── LicenseGuard.js     # Client authorization higher-order wrapper
│   ├── lib/
│   │   └── db.js               # Cached MongoDB connection manager
│   ├── models/
│   │   └── License.js          # Mongoose schema for license validation
│   └── utils/
│       └── device.js           # Client device identifier generator
├── .env.example                # Safe environment configuration template
├── .gitignore                  # Production exclusion rules
├── eslint.config.mjs           # Flat ESLint configuration
├── next.config.mjs             # Next.js runtime configuration
├── package.json                # Project dependencies and script declarations
└── README.md                   # Enterprise architecture and project guide
```

---

## ⚙️ Environment Configuration

Create a `.env.local` file in the root directory by duplicating `.env.example`:

```bash
cp .env.example .env.local
```

Populate the required environment variables:

| Variable | Description | Example / Default |
| :--- | :--- | :--- |
| `MONGODB_URI` | MongoDB Atlas or localized MongoDB connection string | `mongodb+srv://admin:pass@cluster.mongodb.net/lexora_student_os` |
| `ADMIN_SECRET` | Header secret for programmatic license creation | `your_secure_admin_token` |
| `NEXT_PUBLIC_APP_URL` | Base host URL for deployment environments | `http://localhost:3000` |

---

## 🚀 Quick Start Guide

### Prerequisites
- **Node.js**: v18.x or v20+ (Node v22+ recommended)
- **npm** or **pnpm**
- **MongoDB**: Active MongoDB database instance

### 1. Installation
Clone the repository and install project dependencies:

```bash
git clone https://github.com/theekshana-sudeepa/lexora-student-os.git
cd lexora-student-os
npm install
```

### 2. Run Development Server
Start the Turbopack-powered Next.js development server:

```bash
npm run dev
```

Navigate to `http://localhost:3000` in your browser.

### 3. Build & Production Run
To compile and launch the optimized production build:

```bash
npm run build
npm run start
```

---

## 🔒 Security Best Practices

1. **Zero Client Secret Exposure:** Database connection strings (`MONGODB_URI`) and admin secrets (`ADMIN_SECRET`) remain isolated to serverless route handlers and are never bundled into client scripts.
2. **Device Quota Enforcement:** Client UUID fingerprints prevent unauthorized license proliferation beyond user limits.
3. **Environment Isolation:** All sensitive credentials are excluded from version control via `.gitignore`.

---

## 📄 License

This project is licensed under the MIT License.

---

## 👨‍💻 Author & Maintainer

**Theekshana Sudeepa**
- LinkedIn: [linkedin.com/in/theekshana-sudeepa](https://www.linkedin.com/in/theekshana-sudeepa/)
- GitHub: [github.com/theekshana-sudeepa](https://github.com/theekshana-sudeepa)
- Organization: [LexoraTech](https://lexoratech.com)
