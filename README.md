# SWEDTEL Cloud Management Platform - Enterprise Web SaaS Architecture

**SWEDTEL Cloud Management Platform** is a full-stack, multi-tenant enterprise Cloud Management Platform (CMP) and FinOps optimization engine tailored for Nordic and global enterprise customers managing hybrid multi-cloud environments (**Google Cloud Platform**, **AWS**, and **Microsoft Azure**).

---

## 1. Executive Technical Summary

* **Target Architecture**: Standard 3-Tier Enterprise Cloud SaaS (Web Frontend → Secure REST API Backend → Managed PostgreSQL Database → Cloud Provider Adapter Layer).
* **Primary Cloud Integration Target**: **Google Cloud Platform (GCP)** using **Workload Identity Federation (WIF)** for keyless authentication.
* **Server-Side AI Engine**: **Google Gemini Models** (`gemini-3.1-pro` for deep FinOps reasoning and compliance; `gemini-3.1-flash-lite` for sub-second SRE incident triage runbooks) invoked exclusively from backend microservices with zero client API key exposure.
* **Multi-Tenant Isolation**: Server-side enforced authorization context across all SQL queries, API endpoints, alerts, FinOps cost records, and support ticket internal notes.
* **Compliance Baseline**: **CIS Google Cloud Foundation Benchmark v2.0** and continuous **SOC 2 Type II** audit logging.

---

## 2. Directory Structure

```
.
├── server/                         # Backend REST API (Node.js / Express / TypeScript)
│   ├── migrations/                 # PostgreSQL DDL & Production Seed Migrations
│   │   ├── 001_initial_schema.sql  # Enterprise multi-tenant relational schema
│   │   └── 002_seed_data.sql       # Demo seed data (Nordic Health, Scania, Telia)
│   ├── src/
│   │   ├── ai/                     # Server-Side Gemini Advisor & SRE Runbook Engine
│   │   ├── cloud/                  # Cloud Provider Adapter Abstractions (GCP, AWS, Azure)
│   │   ├── config/                 # Secret Manager & Environment Variable Loader
│   │   ├── controllers/            # REST API Endpoint Business Logic
│   │   ├── db/                     # PostgreSQL Connection Pool & In-Memory Store
│   │   ├── middleware/             # Auth, Multi-Tenant Boundary & RBAC Middlewares
│   │   ├── routes/                 # Express Route Declarations
│   │   ├── types/                  # Domain Models, Enums & Interfaces
│   │   └── index.ts                # Server Entrypoint
│   ├── package.json
│   └── tsconfig.json
│
├── web/                            # Enterprise Web Frontend (React 18 / TypeScript / Vite / Tailwind)
│   ├── src/
│   │   ├── api/                    # Typed API Client with dynamic tenant headers
│   │   ├── components/             # Navbar, Sidebar, AI Drawer, Safety Modals
│   │   ├── context/                # AuthContext & Multi-Tenant Role Switcher
│   │   ├── views/                  # FinOps, Infrastructure, Security, Automation Views
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── vite.config.ts
│   └── tailwind.config.js
│
└── infra/                          # Production Deployment Blueprints
    ├── terraform/                  # Terraform for GCP (Cloud Run, Cloud SQL, WIF, Armor)
    │   └── main.tf
    └── docker/                     # Multi-stage Containerfiles & Compose
        ├── Dockerfile.api
        ├── Dockerfile.web
        └── docker-compose.yml
```

---

## 3. Security & Multi-Tenant Isolation Model

1. **Server-Side Tenant Derivation**:
   * Tenant isolation is enforced in `server/src/middleware/tenant.ts`.
   * For customer roles (`CUSTOMER_ADMIN`, `CUSTOMER_VIEWER`), the `customerId` is strictly bound to the authenticated user token. Attempts to query or modify data belonging to other tenants result in immediate `403 CROSS_TENANT_VIOLATION` responses.
2. **Zero Client-Side Secret Leakage**:
   * The web browser **NEVER** receives or holds GCP Service Account JSON keys, AWS Secret Keys, Azure client secrets, or Gemini API keys.
   * All credentials are encrypted in **Google Secret Manager** and accessed exclusively via IAM `roles/secretmanager.secretAccessor` assigned to the Cloud Run service identity.
3. **Keyless Cloud Federation (WIF)**:
   * Google Cloud connections use Workload Identity Federation, eliminating static long-lived keys.
4. **Automated SOC 2 Audit Trail**:
   * All administrative actions, tenant switches, security remediations, and automation workflow executions are written to an immutable audit log repository.

---

## 4. Google Cloud Production Deployment Guide

### Deploying via Terraform & Google Cloud CLI

```bash
# 1. Authenticate with Google Cloud
gcloud auth login
gcloud config set project swedtel-cloud-platform-prod

# 2. Provision Infrastructure via Terraform
cd infra/terraform
terraform init
terraform apply -var="project_id=swedtel-cloud-platform-prod"

# 3. Build & Deploy Backend Container to Google Cloud Run
cd ../../server
gcloud builds submit --tag europe-north1-docker.pkg.dev/swedtel-cloud-platform-prod/swedtel-containers/api:v1.0.0 .
gcloud run deploy swedtel-backend-api \
  --image europe-north1-docker.pkg.dev/swedtel-cloud-platform-prod/swedtel-containers/api:v1.0.0 \
  --region europe-north1 \
  --platform managed
```

---

## 5. Feature Implementation & Real vs. Mock Status

| Feature Area | Status | Implementation Details |
| :--- | :--- | :--- |
| **Web SaaS Architecture** | **FULLY FUNCTIONAL** | Modern React 18, Tailwind CSS, Nordic tech visual branding. |
| **Backend REST API** | **FULLY FUNCTIONAL** | Node.js / Express / TypeScript API with full schema validation. |
| **Relational Database** | **FULLY FUNCTIONAL** | PostgreSQL 16 schema + automated fallback in-memory store. |
| **Server-Side Auth & RBAC** | **FULLY FUNCTIONAL** | JWT auth + Super Admin, Engineer, FinOps, Customer Admin roles. |
| **Multi-Tenant Isolation** | **FULLY FUNCTIONAL** | Server-side query filtering and boundary enforcement. |
| **Server-Side Gemini AI** | **FULLY FUNCTIONAL** | Server-to-server calls to `gemini-3.1-pro` and `gemini-3.1-flash-lite`. |
| **Google Cloud Integration** | **LIVE ARCHITECTURE** | Workload Identity Federation & Cloud Asset Inventory adapter. |
| **AWS & Azure Integration** | **MOCK / SIMULATION** | Safe simulation adapter for demo accounts without destructive calls. |
| **FinOps Cost Intelligence** | **FULLY FUNCTIONAL** | Spend analytics, budget variance tracking, anomaly detection. |
| **Automation & Governance** | **FULLY FUNCTIONAL** | Two-man rule approval modals for destructive cloud operations. |
| **Support Helpdesk** | **FULLY FUNCTIONAL** | Real-time ticket threads with private engineering notes. |
| **SOC 2 Audit Trail** | **FULLY FUNCTIONAL** | Immutable audit log store with IP and identity tracking. |
