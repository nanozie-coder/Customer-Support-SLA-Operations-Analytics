# Data Dictionary

## Customer Support SLA & Operations Analytics

This document defines the principal data fields, tables and business terminology used in the Power BI Customer Support SLA & Operations Analytics project.

The dataset is evaluated using a reporting snapshot date of **4 July 2026**.

---

## 1. Fact Tables

### FCT_TICKETS

Primary transactional table containing one record per support ticket.

| Field | Type / Role | Definition |
|---|---|---|
| TICKET_ID | Identifier | Unique identifier for each ticket. |
| TICKET_REFERENCE | Identifier | Business-facing ticket reference. |
| CLIENT_ID | Foreign Key | Links the ticket to DIM_CLIENTS. |
| AGENT_ID | Foreign Key | Identifies the agent assigned to the ticket. |
| CATEGORY_ID | Foreign Key | Links the ticket to DIM_TICKET_CATEGORY. |
| PRIORITY_ID | Foreign Key | Links the ticket to DIM_PRIORITY. |
| CHANNEL | Attribute | Channel through which the ticket was received. |
| CONTRACT_TIER | Attribute | Customer SLA tier: Standard, Premium or Enterprise. |
| CREATED_AT | Date/Time | Timestamp when the ticket was created. |
| FIRST_RESPONSE_AT | Date/Time | Timestamp of the first recorded response. |
| RESOLVED_AT | Date/Time | Timestamp when the ticket was resolved or closed. |
| SLA_DUE_AT | Date/Time | SLA deadline applicable to the ticket. |
| CREATED_DATE_KEY | Date Key | Date-level key used for created-date analysis. |
| FIRST_RESPONSE_DATE_KEY | Date Key | Date-level key for first-response analysis. |
| RESOLVED_DATE_KEY | Date Key | Date-level key for resolution analysis. |
| SLA_DUE_DATE_KEY | Date Key | Date-level key corresponding to the SLA deadline. |
| SLA STATUS | Status | Current ticket status: Closed, Resolved, Open, In Progress, Pending Client or Escalated. |
| FIRST_RESPONSE_HOURS | SLA Target | Permitted first-response time under the applicable SLA. |
| FIRST_RESPONSE_MINUTES | Actual Metric | Source-calculated actual first-response duration in minutes. |
| RESOLUTION_HOURS | SLA Target | Permitted resolution time under the applicable SLA. |
| RESOLUTION_HOURS_ACTUAL | Actual Metric | Actual resolution duration. |
| ESCALATION_TRIGGER_HOURS | SLA Threshold | Time threshold at which escalation is triggered under the applicable SLA policy. |
| RESPONDED_WITHIN_SLA | Source Flag | Indicates whether the response SLA was achieved. |
| RESOLVED_WITHIN_SLA | Source Flag | Indicates whether the resolution SLA was achieved. |
| SLA_BREACHED | Source Flag | Overall source indicator showing whether the ticket breached its SLA. |
| SLA_BREACH_COUNT | Source Metric | Source-provided breach count indicator. |
| ESCALATED_FLAG | Source Flag | Indicates whether the ticket experienced an escalation at any point. |
| TICKET_AGE_DAYS | Snapshot Metric | Age of the ticket according to the source dataset snapshot. |
| SLA Remaining Hour | Calculated Column | Hours remaining before the SLA deadline. Negative values indicate that the SLA deadline has passed. |

> **Important:** `SLA STATUS = "Escalated"` represents the ticket's current status, while `ESCALATED_FLAG` identifies whether a ticket experienced an escalation at any point. Historical escalation analysis therefore uses `ESCALATED_FLAG`.

### FCT_ESCALATIONS

Stores ticket escalation events and supports analysis of escalation activity.

### FCT_TICKET_AUDIT

Audit/history fact table associated with ticket activity.

---

## 2. Dimension Tables

| Table | Purpose |
|---|---|
| DIM_DATE | Calendar dimension supporting time-based analysis and filtering. |
| DIM_AGENTS | Agent master data including identity, role, hub and active/inactive status. |
| DIM_CLIENTS | Client attributes supporting customer segmentation and client-type analysis. |
| DIM_PRIORITY | Ticket-priority classifications used for SLA risk analysis. |
| DIM_TICKET_CATEGORY | Ticket-category classifications used for operational and SLA analysis. |
| DIM_SLA | SLA reference information supporting service-level analysis. |

---

## 3. Core KPI Definitions

| KPI | Business Definition |
|---|---|
| Ticket Count | Distinct count of support tickets. |
| Completed Tickets | Tickets whose current status is Resolved or Closed. |
| Active Workload | Tickets currently Open, In Progress, Pending Client or Escalated. |
| Ticket Completion Rate | Completed Tickets divided by total Ticket Count. |
| SLA Breached Tickets | Completed tickets identified as having breached the overall SLA. |
| SLA Met Tickets | Completed tickets identified as having met the overall SLA. |
| SLA Compliance % | SLA Met Tickets divided by Completed Tickets. |
| SLA Breach % | SLA Breached Tickets divided by Completed Tickets. |
| Response SLA Met Tickets | Tickets that achieved the source-defined response SLA. |
| Escalated Tickets | Tickets that experienced an escalation at any point. |
| Escalation Rate % | Escalated Tickets divided by Ticket Count. |
| Overdue Active Tickets | Active tickets whose SLA deadline has passed. |
| Avg First Response Time | Average source-calculated first-response duration in minutes. |
| Active Agents | Agents currently flagged as active. |
| Agents with Active Workload | Agents assigned at least one currently active ticket. |
| Avg Tickets per Agent | Ticket Count divided by Active Agents. |
| Underutilised Agents | Active agents whose workload is at or below the first-quartile workload threshold. |
| Agent SLA Compliance % | SLA compliance calculated within the individual agent context. |

---

## 4. SLA Policy

| Contract Tier | First Response Target | Resolution Target | Escalation Trigger |
|---|---:|---:|---:|
| Standard | 4 hours | 24 hours | 20 hours |
| Premium | 4 hours | 48 hours | 40 hours |
| Enterprise | 8 hours | 72 hours | 60 hours |

These are source-defined SLA policy values rather than assumptions introduced during dashboard development.

---

## 5. Ticket Status Definitions

| Status | Interpretation |
|---|---|
| Open | Ticket remains open. |
| In Progress | Ticket is actively being worked. |
| Pending Client | Further progress is awaiting client action or information. |
| Escalated | Ticket is currently in an escalated state. |
| Resolved | Ticket has been resolved. |
| Closed | Ticket has reached closed status. |
| Completed | Analytical grouping covering both Resolved and Closed. |
| Active Workload | Analytical grouping covering Open, In Progress, Pending Client and Escalated. |

### Ticket Population Reconciliation

**Completed Tickets (3,301) + Active Workload (199) = Total Tickets (3,500)**

---

## 6. Snapshot Rule

The reporting snapshot date is **4 July 2026**.

For unresolved tickets, time-sensitive calculations use this fixed snapshot date rather than the current system date. This ensures that historical portfolio results remain reproducible when the report is opened at a later date.

For `SLA Remaining Hour`:

- **Positive value** = SLA time remaining
- **Negative value** = SLA deadline has passed

---

## 7. Workforce Utilisation Rule

The underutilisation threshold is calculated as the **25th percentile (Q1)** of ticket workload among active agents.

For this dataset:

**Underutilisation Threshold = 2 tickets**

An active agent with workload at or below this threshold is classified as underutilised for the purposes of this analysis.

---
