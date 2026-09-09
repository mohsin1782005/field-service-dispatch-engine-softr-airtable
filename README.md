# NovaField Operations Portal & Real-Time Dispatch Engine

![NovaField Operations Architecture](Fields_service_operations_Softr_Airtable.jpg)

[![System Architecture: Production](https://img.shields.io/badge/Architecture-Enterprise_B2B-blue.svg)](#system-architecture)
[![Database: Airtable Relational Base](https://img.shields.io/badge/Database-Airtable_Relational-18BFFF.svg)](#relational-database-schema)
[![Portal: Softr Enterprise Engine](https://img.shields.io/badge/Portal-Softr_PWA-FF5A5F.svg)](#technician-mobile-execution-suite)
[![Security: Role-Based Access Control](https://img.shields.io/badge/Security-Strict_RBAC-2ea44f.svg)](#security--role-based-access-control-rbac)

---

## 📌 Overview

**NovaField Operations Portal** is an end-to-end field service management and mobile workforce dispatch system for residential and commercial technical operations. It decouples high-velocity public customer intake from authenticated contractor views to eliminate dispatch friction, enforce data segregation, and give field technicians a low-latency mobile execution suite.

---

## 🏗️ Data Ingestion Pipeline

```mermaid
flowchart TD
    subgraph Public Tier [Public Intake Ingestion]
        A[Customer Submits Ticket via /book-service] -->|HTTPS POST| B[Intake Validation Engine]
        B -->|Payload Normalization| C[(Airtable Master Data Engine)]
    end

    subgraph Operational Data Core [Relational Schema Hub]
        C --> D{Dispatch & Triage Queue}
        D -->|Initial State| E[Status: Pending Dispatch]
        D -->|Foreign Key Mapping| F[Customers ⟷ Work Orders ⟷ Technicians]
    end

    subgraph Authenticated Edge Tier [Technician Mobile PWA]
        G[Technician Session Auth] -->|RBAC Validation| H[Live Dispatch Kanban]
        E -->|Synchronous State Sync| H
        H -->|Drag & Drop State Change| I[In Progress / Completed]
        I -->|Bi-directional Sync| C
        H -->|Field Logging Modal| J[Diagnostic Notes & Resolution Logging]
        J -->|Atomic Mutation| C
    end
```

---

## 🗄️ Database Schema

| Collection | Key Fields |
|---|---|
| **Work Orders** | Work Order ID (PK), Job Type (enum), Status (state machine), Technician Notes, Scheduled Date (ISO-8601), Customer Link (FK), Assigned Technician (FK) |
| **Technicians** | Tech ID, Full Name, Email (SSO/magic-link), Assigned Work Orders (inverse FK array), Active Job Count (roll-up) |
| **Customers** | Customer ID, Full Name & Email, Service Address, Service History |

---

## 🚀 Key Capabilities

**Zero-Friction Customer Intake**
- Public route `/book-service` — unauthenticated webform mapped directly to the work order ingestion pipeline
- Strict payload validation on dates, descriptions, and job classification
- Automated post-submission confirmation + dispatch queueing

**Mobile Field Technician Suite (PWA)**
- Interactive Kanban board for state transitions on mobile
- Atomic modals for Status/Notes updates without raw DB access
- Real-time bidirectional data sync

**Enterprise RBAC**
- Public access limited to `/` and `/book-service`; internal consoles fully gated
- Strict read/write role segregation
- Row-level security scoping technicians to their own assigned queues

---

## 🛠️ Stack

| Layer | Technology | Function |
|---|---|---|
| Front-End Portal | Softr Studio | Intake forms, Kanban board, field update modals |
| Database | Airtable Relational Base | Operational datastore, linked records, state enums |
| Auth | Softr Identity & Access Management | User groups, sessions, route-level access rules |
| Sync | Bidirectional REST Sync Engine | Continuous state sync across forms, DB, and technician views |

---

## 📈 Roadmap

- Automated webhook dispatch notifications (SMS/WhatsApp/email on state transitions)
- Dynamic SLA escalation engine for stale `Pending Dispatch` tickets
- Self-service customer status hub via tokenized tracking URLs
