# Prioritization: Enterprise User Management at Scale

> How I prioritized a platform feature that had no direct revenue impact, no flashy metrics, and no executive sponsor pushing for it, but was quietly consuming the implementation team's capacity and eroding customer trust at the largest accounts.

---

## The Core Tension

This project didn't have the urgency of a regulatory deadline or the revenue pull of a marketplace launch. It had something harder to prioritize: **a slow-burn operational bottleneck that nobody outside the implementation team was screaming about.**

Leadership wasn't asking for it. Customers were frustrated but not churning (yet). The engineering team had a backlog of feature requests with clearer business cases. The challenge was making the invisible visible: proving that the time the implementation team spent on manual user changes was time they weren't spending on new customer onboarding and platform configuration, which had direct revenue impact.

Three competing pressures shaped every decision:

1. **What to build** (a new role with scoped access, not just an admin page) -- the right solution was more complex than the obvious one
2. **How to scope it** (tenant isolation in a multi-tenant environment) -- the constraint that made every design decision harder
3. **How to justify it** (framing operational burden as revenue protection) -- the only way to get it prioritized against feature work

---

## Framework: Operational Burden as Revenue Proxy

Standard RICE doesn't score "implementation team is drowning" well. Reach is internal (implementation team, not customers). Impact is indirect (customers wait longer for onboarding, not for a feature). Confidence is high but the metric is squishy ("fewer tickets" isn't a revenue number).

I reframed the prioritization by connecting the operational burden to revenue:

### The Calculation

| Metric | Value |
|--------|-------|
| Implementation team hours spent on user management requests per week | Tracked for 3 weeks to get a baseline |
| Those same hours valued at: new customer onboarding capacity | Each onboarding generates recurring revenue |
| Number of queued onboardings delayed by implementation team overload | Direct revenue delay |
| Largest customer's escalation frequency on user management | Churn risk signal |

This turned "our implementation team is busy" into "we're delaying X onboardings worth Y revenue per quarter because the implementation team is spending Z hours on requests that customers could handle themselves."

---

## Sequencing: What to Build First

### The Obvious Solution vs. the Right Solution

| Approach | Effort | Risk | Why I chose or rejected it |
|----------|--------|------|---------------------------|
| **Give customers admin access** | Low | Very high -- admin workspace exposes platform config, other tenants, system settings | **Rejected.** Blast radius too large. A mistake in admin access could leak data across tenants. |
| **Build a user management page inside the admin workspace** | Medium | High -- admin workspace is a complex surface; scoping visibility within it is fragile | **Rejected.** Too many edge cases where admin-level controls could be exposed accidentally. |
| **Create a new dedicated User Manager role with its own isolated page** | High | Low -- completely separate from admin workspace, own page boundary, own permission scope | **Chose this.** Higher build effort, but lowest blast radius. If we got the scoping wrong, the worst case was a user management bug, not a platform configuration breach. |

The right solution was the most expensive to build. Justifying that cost required showing that the cheaper options carried unacceptable risk in a multi-tenant environment.

### Feature Sequencing Within the Build

| Capability | Priority | Reasoning |
|-----------|----------|-----------|
| **Add and remove users** | First | Covers 60%+ of all implementation team user management tickets. Highest volume request. |
| **Assign and revoke roles** | First (same phase) | Second most common request. Tightly coupled with add/remove -- doesn't make sense to ship separately. |
| **Bulk operations** (add/remove multiple users at once) | Second | Large organizations with 100K+ users need this, but single-user operations unblock the core pain immediately. |
| **Audit trail** (who changed what, when) | Second | Important for compliance at large enterprises, but not blocking the core self-service capability. |
| **Role template management** (create reusable role bundles) | Third | Nice to have for very large organizations with standardized role structures. Not blocking for initial launch. |
| **API access for user management** (programmatic user provisioning) | Deferred | Some customers wanted to automate user provisioning from their HR systems. Valid use case, but much higher complexity. Deferred to a future phase after self-service UI proved the model. |

### Rollout Sequencing: Which Customers First

| Customer Profile | Rollout Priority | Reasoning |
|-----------------|-----------------|-----------|
| **Large conglomerate (100K+ users)** | Beta (first) | Highest pain, most tickets, most vocal about the problem. Also the hardest test case: if it works for 100K+ users, it works for everyone. |
| **Mid-size customers (5K-50K users)** | Beta (second wave) | Moderate pain, good validation of the solution at different scale. |
| **Small customers (50-500 users)** | GA | Lower urgency (fewer user changes), but benefits from the same self-service capability. Rolled out at general availability. |

Starting with the largest, most complex customer was a deliberate choice. It was the highest-risk beta test, but also the most convincing proof point. If tenant isolation held at 100K+ users in a multi-tenant environment, no one would question it at smaller scale.

---

## The Bigger Prioritization: This Feature vs. the Backlog

The engineering team had a full backlog of feature requests with clearer business cases: new visualization widgets, performance improvements, customer-requested workflow enhancements. Slotting a user management feature into the roadmap required explicit justification.

### How I Made the Case

**Exposed the backlog trade-off to leadership.**
I used the same approach that worked on the platform decomposition project: showed the current backlog with effort assigned to each item, then showed what would need to move to make room. The framing was: "This isn't adding work. This is choosing which work generates the most value per engineering hour."

**Framed it as implementation team capacity recovery.**
The implementation team's time was a finite resource shared between user management requests and new customer onboarding. Every hour spent processing user change tickets was an hour not spent onboarding new customers. I quantified the onboarding delay and attached a revenue number to it.

**Connected it to the largest customer's satisfaction.**
The conglomerate with 100K+ users was a strategic account. Their escalation frequency on user management was increasing. I framed the feature as "revenue protection for our largest account" rather than "internal tooling improvement."

**Positioned it as platform maturity, not just a fix.**
Self-service RBAC is a standard expectation for enterprise SaaS platforms. Not having it was a competitive gap. Framing it as platform maturity (something we'd eventually need anyway) made it easier to prioritize now rather than perpetually deferring it.

---

## Prioritization During Rollout

Even after the build was approved and underway, prioritization decisions continued:

**When the design team proposed a richer UI:**
The design team wanted to build a comprehensive admin-style interface with dashboards, analytics on user activity, and role usage visualization. I scoped it back to the core: add, remove, assign, revoke. The richer UI could come later. Shipping the MVP fast mattered more than shipping a polished experience slowly, because every week of delay was another week of implementation team tickets.

**When engineering found a multi-tenancy edge case:**
During beta testing, engineering discovered a scenario where a user manager with access to a tenant that shared infrastructure with another tenant could, under very specific conditions, see a count (not details, just a number) of users in the adjacent tenant. Not a data leak, but a data signal that shouldn't exist. I prioritized the fix immediately, ahead of the bulk operations feature. Tenant isolation was non-negotiable.

**When the implementation team asked for an override:**
The implementation team wanted to retain the ability to manage users on behalf of customers, even after self-service launched. This was reasonable (for emergencies, for customers who hadn't adopted self-service yet, for complex migrations). I ensured the implementation team's existing access was not removed, and the self-service capability was additive. Both paths coexisted.

---

## What I'd Do Differently

1. **Track implementation team time from the start of my tenure, not when I needed the data.** I tracked user management ticket volume for 3 weeks to build the business case. If I'd been tracking it from day one, the cumulative data would have been more compelling and the business case would have been ready sooner.

2. **Ship bulk operations in the MVP.** I deferred bulk operations (add/remove multiple users at once) to the second phase because single-user operations covered the core pain. But the 100K+ user customer immediately asked for bulk operations during beta. They were adding users in batches of hundreds. Shipping bulk in the MVP would have avoided a second round of implementation for the most important beta customer.

3. **Build the programmatic API earlier.** Several large customers wanted to automate user provisioning from their HR systems (employee joins company, automatically gets provisioned on our platform). I deferred this as "future phase" complexity. In retrospect, even a basic webhook or CSV import would have covered 80% of the automation use case at low effort.

---

*All company names, client names, and partner names have been anonymized. Metrics and decision frameworks are accurate as applied.*
