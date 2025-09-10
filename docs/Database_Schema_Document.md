# MNFSR IT System – Database Schema Document

## Document Information
- **Project**: Digital Transformation of MNFSR
- **Client**: Ministry of National Food Security & Research (MNFSR), Pakistan
- **Version**: 1.0
- **Date**: September 2025
- **Prepared By**: SOFT PYRAMID

---

## 1. Executive Summary
This document defines the logical database design, ERDs, and standards for all six modules, with cross-module relationships, indexing, data types, migration, backup/restore, security, and performance recommendations. SQL Server is the authoritative data store; KMS uses Elasticsearch for search indexing. Power BI consumes curated views.

---

## 2. Conventions & Standards
- Naming: snake_case for tables/columns; singular table names (e.g., commodity, vehicle)
- Keys: surrogate integer/bigint identity keys; natural keys constrained where applicable
- Timestamps: created_at, updated_at (UTC), deleted_at (nullable) for soft-deletes where needed
- Multi-tenancy: ministry/department via department_id where applicable
- Auditing: audit trails via audit_log with entity_id, action, old/new values, actor_id

---

## 3. ERDs by Module

### 3.1 Pakistan Agri Trade Portal (PATP)
```mermaid
erDiagram
    commodity ||--o{ market_price : has
    commodity {
      bigint id PK
      varchar name
      varchar category  // crop, meat, oil, vegetable, fruit, aqua, seed
    }
    market_price {
      bigint id PK
      bigint commodity_id FK
      date price_date
      decimal price_usd
      varchar source   // FAO, UN, ITC, PBS
      varchar market   // domestic/international
    }
    supplier ||--o{ supplier_product : offers
    supplier {
      bigint id PK
      varchar name
      varchar country
    }
    supplier_product {
      bigint id PK
      bigint supplier_id FK
      bigint commodity_id FK
    }
    mill {
      bigint id PK
      varchar type // flour,rice,sugar
      varchar name
      integer storage_capacity_tons
      varchar location
    }
    grievance {
      bigint id PK
      bigint user_id FK
      varchar subject
      text description
      varchar status
    }
    consultation {
      bigint id PK
      bigint user_id FK
      varchar channel // chat,email
      text question
      text answer
    }
```

### 3.2 Knowledge Management System (KMS)
```mermaid
erDiagram
    document ||--o{ document_version : has
    document ||--o{ document_tag : tagged
    tag ||--o{ document_tag : maps
    document {
      bigint id PK
      varchar title
      varchar classification // public, restricted, confidential
      bigint owner_id FK
    }
    document_version {
      bigint id PK
      bigint document_id FK
      integer version_no
      varchar file_uri
      varchar checksum
    }
    tag {
      bigint id PK
      varchar name
    }
    permission {
      bigint id PK
      bigint document_id FK
      bigint principal_id FK
      varchar access // read, write, admin
    }
```

### 3.3 Vehicle Management System (VMS)
```mermaid
erDiagram
    vehicle ||--o{ maintenance : has
    vehicle ||--o{ fuel_log : has
    vehicle ||--o{ allocation : has
    driver ||--o{ allocation : assigned
    vehicle {
      bigint id PK
      varchar reg_no UQ
      varchar make
      varchar model
      integer year
      varchar status
    }
    maintenance {
      bigint id PK
      bigint vehicle_id FK
      date service_date
      varchar type
      decimal cost
    }
    fuel_log {
      bigint id PK
      bigint vehicle_id FK
      date fill_date
      decimal liters
      decimal amount
    }
    allocation {
      bigint id PK
      bigint vehicle_id FK
      bigint driver_id FK
      date from_date
      date to_date
    }
    driver {
      bigint id PK
      varchar name
      varchar license_no
    }
```

### 3.4 Inventory Management System (IMS)
```mermaid
erDiagram
    asset ||--o{ stock_txn : affects
    location ||--o{ stock_txn : occurs
    requisition ||--o{ approval : has
    issue ||--o{ return : has
    asset {
      bigint id PK
      varchar code UQ
      varchar name
      varchar category
      decimal unit_cost
    }
    stock_txn {
      bigint id PK
      bigint asset_id FK
      bigint location_id FK
      varchar type // in,out,transfer,adjust
      integer quantity
      timestamp txn_at
    }
    location {
      bigint id PK
      varchar name
      varchar type // store,office
    }
    requisition {
      bigint id PK
      bigint requestor_id FK
      timestamp requested_at
      varchar status
    }
    approval {
      bigint id PK
      bigint requisition_id FK
      bigint approver_id FK
      timestamp approved_at
      varchar decision
    }
    issue {
      bigint id PK
      bigint requisition_id FK
      timestamp issued_at
    }
    return {
      bigint id PK
      bigint issue_id FK
      timestamp returned_at
      varchar condition
    }
```

### 3.5 PSDP Management System
```mermaid
erDiagram
    project ||--o{ milestone : has
    project ||--o{ budget : has
    project ||--o{ expenditure : has
    project ||--o{ progress_report : has
    project {
      bigint id PK
      varchar code UQ
      varchar title
      varchar status
      date start_date
      date end_date
    }
    milestone {
      bigint id PK
      bigint project_id FK
      varchar name
      date due_date
      varchar status
    }
    budget {
      bigint id PK
      bigint project_id FK
      decimal amount
      varchar fy
    }
    expenditure {
      bigint id PK
      bigint project_id FK
      decimal amount
      date expense_date
      varchar head
    }
    progress_report {
      bigint id PK
      bigint project_id FK
      date report_date
      text summary
    }
```

### 3.6 International Collaboration (IC) Management
```mermaid
erDiagram
    agreement ||--o{ renewal : has
    agreement ||--o{ payment_obligation : has
    agreement ||--o{ stakeholder : involves
    country ||--o{ agreement : parties
    agreement {
      bigint id PK
      varchar ref_no UQ
      varchar type // MOU,MOI,Protocol,JWG,JMC
      bigint country_id FK
      date start_date
      date end_date
      varchar status
    }
    renewal {
      bigint id PK
      bigint agreement_id FK
      date renewal_due
      varchar status
    }
    payment_obligation {
      bigint id PK
      bigint agreement_id FK
      decimal amount
      date due_date
    }
    stakeholder {
      bigint id PK
      bigint agreement_id FK
      varchar name
      varchar role
    }
    country {
      bigint id PK
      varchar name
      varchar iso2 UQ
    }
```

---

## 4. Cross-Module Relationships
- `user` (central identity) references across modules for creator/owner/responsible
- `department` used by VMS/IMS/PSDP/IC for organizational reporting
- `document` repository shared (KMS/DOCS) by PSDP and IC for file storage
- `audit_log` captures actions across services

---

## 5. Keys, Constraints, and Indexing
- Primary keys: bigint identity; composite unique constraints for natural keys (e.g., reg_no, code, iso2)
- Foreign keys: ON UPDATE RESTRICT; ON DELETE CASCADE for children where safe; otherwise SET NULL
- Indexing: B-tree on foreign keys and frequent filters; covering indexes for date ranges (price_date, service_date, txn_at, report_date)
- Partitioning: large fact tables (market_price, stock_txn, expenditure) by month
- Full-text: KMS document metadata fields

---

## 6. Data Types & Validation Rules
- Monetary: decimal(18,2); quantities: integer; dates: date/timestamp with UTC
- Enum-like fields: constrained via lookup tables or CHECK
- Validation: server-side constraints plus application validations; idempotency keys for ingestion

---

## 7. Data Migration Strategy
- Phased migration per module; staging tables; reconciliation reports
- ETL jobs with audit checkpoints; rollback plans
- Data quality: profiling, deduplication, referential integrity enforcement

---

## 8. Backup & Recovery Procedures
- SQL Server: full daily, differential hourly, log backups every 15 min; geo-redundant storage
- Restore drills quarterly; document RPO ≤ 1h, RTO ≤ 4h
- Object storage versioning for documents

---

## 9. Database Security & Access Control
- RBAC at database: least privilege roles (read, write, admin, report)
- Encryption: TDE for databases; column-level for sensitive fields (PII)
- Secrets management via key vault; rotation policy
- Auditing: DDL/DML auditing; privileged access monitoring

---

## 10. Performance Optimization
- Use parameterized queries; avoid RBAR; batch writes
- Precompute aggregates via indexed/materialized views for BI
- Use read replicas for heavy reporting loads
- Monitor with query store and telemetry dashboards

---

## 11. Appendices
- A. Naming standards and sample DDL templates
- B. Data dictionary (to be generated from OpenAPI + migrations)
- C. Power BI model star schemas (high-level)
