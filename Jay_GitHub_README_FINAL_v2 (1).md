# AWS Cloud Financial Governance & Automated Budget Monitoring System

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws&logoColor=white)
![FinOps](https://img.shields.io/badge/FinOps-Cost%20Governance-blue)
![IAM](https://img.shields.io/badge/IAM-Enforcement-red?logo=amazonaws)
![SNS](https://img.shields.io/badge/SNS-Alerting-yellow?logo=amazonaws)
![Status](https://img.shields.io/badge/Status-Deployed%20%26%20Validated-brightgreen)

> **Read the full project write-up on Medium:** [link]
> **Connect on LinkedIn:** [link]

---

## TL;DR

A five-layer cloud cost governance framework built entirely on native AWS tools. The system monitors spend, sends real-time alerts, and automatically enforces IAM restrictions when budget thresholds are crossed — no manual intervention required. Built to reflect how FinOps and Cloud Platform teams manage financial control in production AWS environments.

---

## What This Project Demonstrates

- Designing a layered cloud financial governance system from scratch using native AWS tooling
- Automating budget enforcement with IAM policy actions — not just passive alerting
- Configuring IAM trust relationships and execution roles for budget-triggered enforcement
- Applying FinOps principles: cost visibility, proactive monitoring, active control, and accountability
- Production-oriented thinking: the system responds to cost events, it does not just report them

---

## Prerequisites

Before deploying this project, you will need:

- An AWS account with an admin IAM user created
- IAM billing access activated by the root user (Account Settings → IAM User and Role Access to Billing Information → Activate)
- The `Billing` managed policy attached to your admin IAM user
- An email address to receive SNS budget alert notifications

---

## Architecture Overview

The governance framework evolves across five implementations, each adding a new control layer:

```
AWS Account Spend
        │
        ▼
┌─────────────────────────────┐
│         AWS Budgets         │
│  Budget: monthly-cost-      │
│  monitor | $3.00 fixed      │
│  Threshold: 80% actual cost │
└────────────┬────────────────┘
             │  Threshold Breached
             │
    ┌────────┴────────────┐
    │                     │
    ▼                     ▼
SNS Topic            Budget Action
cost-governance-    (Automated IAM
alerts               Enforcement)
    │                     │
    ▼                     ▼
Email Alert        cost-control-deny-ec2
(Immediate)        policy applied via
                   budget-enforcement-role
                   (ec2:RunInstances denied)
```

**Full governance lifecycle:**

```
Implementation 1 → Budget + SNS Alert        (reactive alerting)
Implementation 2 → Forecast-Based Alerts     (proactive alerting)
Implementation 3 → Automated IAM Enforcement (active cost control)
Implementation 4 → Cost Explorer Analysis    (visibility & optimisation)
Implementation 5 → Tag Governance            (resource accountability)
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| AWS Budgets | Budget thresholds, alert configuration, automated enforcement actions |
| Amazon SNS | Real-time email notifications on threshold breach |
| AWS IAM | Custom deny policies and budget execution roles |
| AWS Cost Explorer | Spend analysis, trend identification, cost optimisation |
| Amazon S3 | Tag governance demonstration |
| AWS Billing & Cost Allocation Tags | Resource-level cost accountability |

---

## Budget Configuration

| Setting | Value |
|---|---|
| Budget Name | `monthly-cost-monitor` |
| Budget Type | Cost Budget |
| Period | Monthly, Recurring |
| Budgeting Method | Fixed |
| Budget Amount | $3.00 |
| Budget Scope | All AWS Services |
| Alert Threshold | 80% of actual spend ($2.40) |
| Notification Channel | SNS → Email |
| SNS Topic Name | `cost-governance-alerts` |
| SNS Type | Standard |

---

## IAM Enforcement Configuration — Implementation 3

This is the most technically significant part of the project. Instead of relying on a human to respond to an alert, the system automatically applies a restrictive IAM policy the moment the budget threshold is crossed.

### Deny Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyEC2RunInstances",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*"
    }
  ]
}
```

Policy name: `cost-control-deny-ec2`

### Budget Execution Role

| Setting | Value |
|---|---|
| Role Name | `budget-enforcement-role` |
| Trusted Entity | AWS Service: Budgets |
| Attached Policies | `cost-control-deny-ec2`, `IAMFullAccess` |
| Execution Mode | Automatically applied at 80% threshold |

### How Enforcement Works

When actual monthly spend crosses $2.40 (80% of the $3.00 budget):

1. AWS Budgets detects the threshold breach
2. Budgets assumes the `budget-enforcement-role` execution role
3. The `cost-control-deny-ec2` deny policy is applied to the IAM user
4. Any attempt to launch a new EC2 instance returns an `AccessDenied` error
5. Spend escalation is stopped automatically — no human intervention required

This is the difference between monitoring and governance.

---

## Implementation Breakdown

### Implementation 1 — Budget & SNS Notification System

Creates the SNS topic (`cost-governance-alerts`) with a confirmed email subscription, then configures the monthly AWS Budget (`monthly-cost-monitor`) with an 80% actual cost alert connected to the SNS ARN.

**Validated:** Email notification received from AWS Budgets confirming threshold breach and successful SNS delivery.

---

### Implementation 2 — Forecast-Based Monitoring

Extends the budget to add forecast-based alerts. Rather than alerting only when actual spend crosses a threshold, the system now alerts when projected spend is on track to exceed the budget — catching the problem before it happens.

This shifts cost governance from reactive to proactive.

---

### Implementation 3 — Automated IAM Enforcement

A custom IAM deny policy (`cost-control-deny-ec2`) is created and attached to a budget execution role (`budget-enforcement-role`). This role is linked to the 80% budget alert as an automated budget action. When the threshold is crossed, Budgets assumes the role and enforces the deny policy without manual input.

**Validated:** After enforcement triggered, an attempted EC2 instance launch returned `AccessDenied` — confirming `ec2:RunInstances` was successfully restricted by the automated policy.

---

### Implementation 4 — Cost Analysis via AWS Cost Explorer

Spend patterns are analysed by service, time period, and usage type to identify cost drivers and optimisation opportunities. Cost Explorer requires activation in Billing settings and can take up to 24 hours to populate data after enabling.

---

### Implementation 5 — Tag Governance & Cost Accountability

Cost Allocation Tags are enabled and applied to S3 resources to demonstrate resource-level spend tracking. Tags are assigned by environment, project, and owner to show how spend accountability is enforced across a cloud environment — a standard FinOps requirement in production teams.

---

## Key Design Decisions

**Why native AWS governance tools instead of custom Lambda pipelines?**
Enterprise cloud teams standardise on platform-native tooling for cost governance because it is auditable, reliable, and requires no custom code to maintain. This project reflects that pattern deliberately.

**Why automate enforcement rather than just alert?**
A notification-only system still depends on a human responding in time. Attaching IAM enforcement to the budget action removes the response gap entirely. Spend is controlled whether anyone is watching or not.

**Why a layered approach across five implementations?**
Reactive alerting alone is not sufficient governance. Each layer adds a new control — forecast alerting, active enforcement, spend analysis, resource accountability — building a complete lifecycle that mirrors how FinOps practices mature in real organisations.

---

## Skills Demonstrated

- Cloud Financial Management (FinOps)
- AWS Budgets configuration and enforcement actions
- IAM policy design and trust relationship configuration
- Event-driven automation using native AWS services
- Cost visibility and spend analysis with AWS Cost Explorer
- Tag-based cost accountability and resource governance
- Production-oriented architecture design

---

## Related

- Medium article: [link]
- LinkedIn: [link]
