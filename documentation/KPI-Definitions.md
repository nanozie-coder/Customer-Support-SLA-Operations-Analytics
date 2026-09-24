# KPI Definitions

## Customer Support SLA & Operations Analytics

This document defines the principal Power BI measures used in the **Customer Support SLA & Operations Analytics** dashboard.

The definitions distinguish between ticket status, historical events, SLA outcomes and workforce metrics to ensure consistent interpretation across the report.

The dataset is evaluated using a reporting snapshot date of **4 July 2026**.

---

## 1. Ticket Metrics

### Ticket Count

**Business definition:** Total distinct tickets in the selected filter context.

```DAX
Ticket Count =
DISTINCTCOUNT(FCT_TICKETS[TICKET_ID])
```

**Validated result:** 3,500

**Used for:** Overall ticket volume, trends, category analysis and workload reporting.

---

### Completed Tickets

**Business definition:** Tickets whose current status is either `Resolved` or `Closed`.

```DAX
Completed Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[SLA STATUS] IN {"Resolved", "Closed"}
)
```

**Validated result:** 3,301

---

### Active Workload

**Business definition:** Tickets that remain operationally active at the reporting snapshot.

```DAX
Active Workload =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[SLA STATUS] IN {
        "Open",
        "In Progress",
        "Pending Client",
        "Escalated"
    }
)
```

**Validated result:** 199

**Reconciliation:**

**3,301 Completed Tickets + 199 Active Workload = 3,500 Total Tickets**

---

### Ticket Completion Rate %

**Business definition:** Percentage of all tickets that have reached either Resolved or Closed status.

```DAX
Ticket Completion Rate % =
DIVIDE(
    [Completed Tickets],
    [Ticket Count],
    0
)
```

**Validated result:** 94.31%

---

## 2. SLA Metrics

### SLA Breached Tickets

**Business definition:** Completed tickets identified by the source system as having breached the overall SLA.

```DAX
SLA Breached Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[SLA_BREACHED] = "true",
    FCT_TICKETS[SLA STATUS] IN {"Resolved", "Closed"}
)
```

**Validated result:** 677

---

### SLA Met Tickets

**Business definition:** Completed tickets identified as having met the overall SLA.

```DAX
SLA Met Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[SLA_BREACHED] = "false",
    FCT_TICKETS[SLA STATUS] IN {"Resolved", "Closed"}
)
```

**Validated result:** 2,624

---

### SLA Compliance %

**Business definition:** Percentage of completed tickets that met the overall SLA.

```DAX
SLA Compliance % =
DIVIDE(
    [SLA Met Tickets],
    [Completed Tickets],
    0
)
```

**Validated result:** 79.49%

---

### SLA Breach %

**Business definition:** Percentage of completed tickets that breached the overall SLA.

```DAX
SLA Breach % =
DIVIDE(
    [SLA Breached Tickets],
    [Completed Tickets],
    0
)
```

**Validated result:** 20.51%

**Validation:**

**79.49% SLA Compliance + 20.51% SLA Breach = 100%**

---

### Response SLA Met Tickets

**Business definition:** Number of tickets identified by the source system as having met their first-response SLA.

```DAX
Response SLA Met Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[RESPONDED_WITHIN_SLA] = "true"
)
```

**Validated result:** 2,825

---

### Response SLA Compliance %

**Business definition:** Percentage of tickets with a recorded response-SLA result that met their response SLA.

```DAX
Response SLA Compliance % =
DIVIDE(
    [Response SLA Met Tickets],
    CALCULATE(
        DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
        NOT(ISBLANK(FCT_TICKETS[RESPONDED_WITHIN_SLA]))
    ),
    0
)
```

---

### Avg First Response Time (Minutes)

**Business definition:** Average source-calculated first-response duration in minutes.

```DAX
Avg First Response Time (Minutes) =
AVERAGE(FCT_TICKETS[FIRST_RESPONSE_MINUTES])
```

**Validated result:** 303 minutes

The source response-duration metric is retained rather than reconstructed from timestamps because validation showed that the source calculation incorporates logic not reproduced by a simple timestamp difference.

---

## 3. Escalation Metrics

### Escalated Tickets

**Business definition:** Distinct tickets that experienced an escalation at any point in their history.

```DAX
Escalated Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[ESCALATED_FLAG] = "true"
)
```

**Validated result:** 464

`ESCALATED_FLAG` is deliberately used instead of `SLA STATUS = "Escalated"` because a previously escalated ticket can subsequently move to another status such as Resolved or Closed.

---

### Escalation Rate %

**Business definition:** Percentage of all tickets that experienced escalation.

```DAX
Escalation Rate % =
DIVIDE(
    [Escalated Tickets],
    [Ticket Count],
    0
)
```

**Validated result:** 13.3%

---

## 4. Overdue Workload

### SLA Remaining Hour

`SLA Remaining Hour` is a **calculated column**, rather than a measure.

```DAX
SLA Remaining Hour =
VAR SnapshotDate =
    DATE(2026, 7, 4)
VAR ReferenceDateTime =
    IF(
        NOT ISBLANK(FCT_TICKETS[RESOLVED_AT]),
        FCT_TICKETS[RESOLVED_AT],
        SnapshotDate
    )
RETURN
    DATEDIFF(
        ReferenceDateTime,
        FCT_TICKETS[SLA_DUE_AT],
        HOUR
    )
```

**Interpretation:**

- Positive value = SLA time remaining
- Negative value = SLA deadline has passed

---

### Overdue Active Tickets

**Business definition:** Active tickets whose SLA deadline had passed at the reporting snapshot.

```DAX
Overdue Active Tickets =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[TICKET_ID]),
    FCT_TICKETS[SLA STATUS] IN {
        "Open",
        "In Progress",
        "Pending Client",
        "Escalated"
    },
    FCT_TICKETS[SLA Remaining Hour] < 0
)
```

**Validated result:** 199

At the **4 July 2026** reporting snapshot, all 199 active tickets were overdue. This result was separately validated against the underlying `SLA_DUE_AT` timestamps.

---

## 5. Workforce & Capacity Metrics

### Active Agents

**Business definition:** Distinct agents currently flagged as active.

```DAX
Active Agents =
CALCULATE(
    DISTINCTCOUNT(DIM_AGENTS[AGENT_ID]),
    DIM_AGENTS[IS_ACTIVE] = TRUE()
)
```

**Validated result:** 114

---

### Inactive Agents

**Business definition:** Distinct agents currently flagged as inactive.

```DAX
Inactive Agents =
CALCULATE(
    DISTINCTCOUNT(DIM_AGENTS[AGENT_ID]),
    DIM_AGENTS[IS_ACTIVE] = FALSE()
)
```

---

### Agents with Active Workload

**Business definition:** Distinct agents assigned at least one currently active ticket.

```DAX
Agents with Active Workload =
CALCULATE(
    DISTINCTCOUNT(FCT_TICKETS[AGENT_ID]),
    FCT_TICKETS[SLA STATUS] IN {
        "Open",
        "In Progress",
        "Escalated",
        "Pending Client"
    }
)
```

**Validated result:** 86

---

### Avg Tickets per Agent

**Business definition:** Overall ticket volume divided by the number of agents currently flagged as active.

```DAX
Avg Tickets per Agent =
DIVIDE(
    [Ticket Count],
    [Active Agents],
    0
)
```

**Validated result:** 30.70, displayed as 31.

> **Interpretation note:** This divides the selected historical ticket population by agents currently flagged as active. It should therefore be interpreted as a portfolio workload indicator rather than a point-in-time staffing productivity measure.

---

### Underutilisation Threshold

**Business definition:** First quartile (25th percentile) of ticket workload among active agents.

```DAX
Underutilisation Threshold =
VAR ActiveAgentWorkload =
    CALCULATETABLE(
        ADDCOLUMNS(
            VALUES(DIM_AGENTS[AGENT_ID]),
            "@TicketCount", CALCULATE([Ticket Count])
        ),
        REMOVEFILTERS(DIM_AGENTS),
        DIM_AGENTS[IS_ACTIVE] = TRUE()
    )
RETURN
    PERCENTILEX.INC(
        ActiveAgentWorkload,
        [@TicketCount],
        0.25
    )
```

**Validated threshold:** 2 tickets

---

### Underutilised Agents

**Business definition:** Active agents whose assigned-ticket workload is at or below the first-quartile workload threshold.

```DAX
Underutilised Agents =
VAR Threshold = [Underutilisation Threshold]
VAR AgentWorkload =
    ADDCOLUMNS(
        FILTER(
            VALUES(DIM_AGENTS[AGENT_ID]),
            CALCULATE(
                SELECTEDVALUE(DIM_AGENTS[IS_ACTIVE])
            ) = TRUE()
        ),
        "@Tickets",
            CALCULATE(
                DISTINCTCOUNT(FCT_TICKETS[TICKET_ID])
            )
    )
RETURN
    COUNTROWS(
        FILTER(
            AgentWorkload,
            [@Tickets] <= Threshold
        )
    )
```

**Validated result:** 29 agents

---

## 6. Agent SLA Performance

### Agent SLA Compliance %

**Business definition:** SLA compliance calculated within the current agent filter context.

```DAX
Agent SLA Compliance % =
VAR Completed =
    [Completed Tickets]
VAR Breached =
    [SLA Breached Tickets]
RETURN
    IF(
        Completed > 0,
        DIVIDE(
            Completed - Breached,
            Completed
        ),
        BLANK()
    )
```

---

### Agent Eligible for SLA Ranking

**Business definition:** Identifies agents with a sufficient completed-ticket population for the SLA ranking analysis.

```DAX
Agent Eligible for SLA Ranking =
IF(
    [Completed Tickets] >= 5,
    1,
    0
)
```

The agent SLA ranking visual applies:

`Agent Eligible for SLA Ranking = 1`

This means an agent must have at least **5 completed tickets** to be included in the SLA ranking. The rule reduces distortion caused by agents with very small completed-ticket samples.

---

## 7. Reporting Snapshot Rule

All unresolved-ticket calculations in this portfolio project are evaluated as at:

**4 July 2026**

The fixed reporting snapshot is intentional. Using `NOW()` would cause historical results to change whenever the Power BI report was opened in the future.

---

## 8. Key Validation Checks

The dashboard was validated to confirm that:

- Total Tickets = **3,500**
- Completed Tickets = **3,301**
- Active Workload = **199**
- Completed Tickets + Active Workload = **3,500**
- SLA Met Tickets = **2,624**
- SLA Breached Tickets = **677**
- SLA Met Tickets + SLA Breached Tickets = **3,301**
- SLA Compliance = **79.49%**
- SLA Breach Rate = **20.51%**
- SLA Compliance + SLA Breach Rate = **100%**
- Response SLA Met Tickets = **2,825**
- Escalated Tickets = **464**
- Escalation Rate = **13.3%**
- Active Agents = **114**
- Agents with Active Workload = **86**
- Underutilisation Threshold = **2 tickets**
- Underutilised Agents = **29**
- All **199 active tickets** were overdue at the reporting snapshot

---

## Measure Organisation

For model maintainability, reporting measures are centralised in the `_Measures` table and organised into logical display folders covering:

- Agent Performance
- SLA
- Ticket Metrics

This separates reusable analytical measures from source columns and improves report-model usability.
