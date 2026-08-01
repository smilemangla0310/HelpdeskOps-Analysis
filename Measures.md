# HelpDeskOps – DAX Measures

This document describes the key measures used in the HelpDeskOps Power BI report.  
Names and formulas can be adapted to your environment as needed.
---

## 1. Volume and backlog

### 1.1 Ticket Volume (2016–23)

**Purpose:** Count of tickets created from 2016-01-01 onward.

```DAX
Ticket Volume (2016–23) =
CALCULATE (
    COUNTROWS ( issues ),
    issues[issue_created] >= DATE ( 2016, 1, 1 )
)
```

---

### 1.2 Backlog Tickets

**Purpose:** Count of tickets not in a final closed/resolved status.

```DAX
Backlog Tickets =
CALCULATE (
    COUNTROWS ( issues ),
    issues[issue_status] IN { "open", "in_progress", "waiting", "reopened" }
)
```

---

## 2. Age and resolution

### 2.1 Ticket Age (Days) – column

**Purpose:** Age of each ticket in days; used as proxy for resolution time.

```DAX
Ticket Age (Days) =
DATEDIFF (
    issues[issue_created],
    TODAY (),
    DAY
)
```

---

### 2.2 Avg Ticket Age (Days)

**Purpose:** Average age in days across tickets in the current filter context.

```DAX
Avg Ticket Age (Days) =
AVERAGEX (
    issues,
    [Ticket Age (Days)]
)
```

---

## 3. SLA metrics

### 3.1 Resolved Tickets

**Purpose:** Count of tickets considered resolved/completed.

```DAX
Resolved Tickets =
CALCULATE (
    COUNTROWS ( issues ),
    issues[issue_status] IN { "resolved", "closed", "done" }
)
```

---

### 3.2 SLA Met Tickets

**Purpose:** Count of resolved tickets that met SLA (requires SLA flag/logic column).

```DAX
SLA Met Tickets =
CALCULATE (
    COUNTROWS ( issues ),
    issues[issue_status] IN { "resolved", "closed", "done" },
    issues[SLA_Flag] = "Met"
)
```

Replace `SLA_Flag` and `"Met"` with your actual column and values.

---

### 3.3 SLA Breached Tickets

**Purpose:** Count of resolved tickets that violated SLA.

```DAX
SLA Breached Tickets =
CALCULATE (
    COUNTROWS ( issues ),
    issues[issue_status] IN { "resolved", "closed", "done" },
    issues[SLA_Flag] = "Breached"
)
```

---

### 3.4 SLA Compliance %

**Purpose:** Percentage of resolved tickets that met SLA.

```DAX
SLA Compliance % =
DIVIDE (
    [SLA Met Tickets],
    [Resolved Tickets]
)

---

## 4. Agent metrics

### 4.1 Total Agents

**Purpose:** Count of distinct agents in the dataset.

```DAX
Total Agents =
DISTINCTCOUNT ( issues[issue_assignee] )
```

---

### 4.2 Tickets per Agent (Avg)

**Purpose:** Average number of tickets per agent.

```DAX
Tickets per Agent (Avg) =
DIVIDE (
    [Ticket Volume (2016–23)],
    [Total Agents]
)
```

---

### 4.3 Tickets by Agent – base measure

**Purpose:** Ticket count in current filter context.

```DAX
Tickets =
COUNTROWS ( issues )
```

This is used in visuals such as:

- Tickets Handled by Agent
- Top 10 Agents by Tickets

---

## 5. Top 10 Agents logic (visual-level)

To show Top 10 agents by tickets:

1. Use a bar chart with:
   - Axis: `issues[issue_assignee]`
   - Values: `[Tickets]`
2. In the **Filters on this visual** pane:
   - Set `issue_assignee` to **Top N**.
   - `Show items: 10`.
   - `By value: Tickets`.
   - Apply filter.
---

## 6. Backlog by Priority and Type (used in visuals)

### 6.1 Backlog Tickets by Priority

Use `Backlog Tickets` measure with `issue_priority` on Axis.

### 6.2 Backlog Tickets by Issue Type

Use `Backlog Tickets` measure with `issue_type` on Axis.

---

## 7. SLA and Age by Priority / Type

### 7.1 SLA Compliance by Priority

Use:

- Axis: `issues[issue_priority]`
- Values: `[SLA Compliance %]`

### 7.2 Avg Ticket Age by Issue Type

Use:

- Axis: `issues[issue_type]`
- Values: `[Avg Ticket Age (Days)]`


