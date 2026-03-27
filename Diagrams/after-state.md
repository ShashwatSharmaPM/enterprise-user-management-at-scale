# After state: self-service RBAC with tenant-isolated user management

> Dedicated User Manager role scoped to user management only. Works for 50-user organizations and 100K+ user organizations without architectural changes. Zero cross-tenant data leaks. Implementation team freed from the most common support requests.

```mermaid
graph TD
    subgraph ORG[" Large enterprise customer — 100K+ users "]
        direction TB
        ADMIN[Designated user manager<br/>within the organization]
    end

    ADMIN -->|Self-service| UM_PAGE[User management page<br/>dedicated interface]

    UM_PAGE --> ADD[Add users]
    UM_PAGE --> REMOVE[Remove users]
    UM_PAGE --> ROLES[Assign or revoke roles]

    ADD --> SCOPED[All actions scoped<br/>to own tenant only]
    REMOVE --> SCOPED
    ROLES --> SCOPED

    classDef org fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef admin fill:#2d9d78,color:#fff,stroke:#1d7a5c
    classDef page fill:#a3dcc8,color:#0a2a1e,stroke:#5dbfa0
    classDef action fill:#a3dcc8,color:#0a2a1e,stroke:#5dbfa0
    classDef scope fill:#2d9d78,color:#fff,stroke:#1d7a5c

    class ORG org
    class ADMIN admin
    class UM_PAGE page
    class ADD,REMOVE,ROLES action
    class SCOPED scope
```

### How the User Manager role works

```mermaid
graph TD
    PERSON[Person within organization] --> HAS{What roles are assigned?}

    HAS --> UM_ONLY[User Manager role only]
    HAS --> UM_PLUS[User Manager + other roles]

    UM_ONLY --> SEE_UM[Can see: user management page<br/>Cannot see: anything else on the platform]
    UM_PLUS --> SEE_BOTH[Can see: user management page<br/>Plus: whatever other roles grant access to]

    SEE_UM --> ISOLATED[All user data scoped to own tenant<br/>no visibility into other tenants]
    SEE_BOTH --> ISOLATED

    classDef person fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef check fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef role fill:#a3dcc8,color:#0a2a1e,stroke:#5dbfa0
    classDef access fill:#2d9d78,color:#fff,stroke:#1d7a5c

    class PERSON person
    class HAS check
    class UM_ONLY,UM_PLUS role
    class SEE_UM,SEE_BOTH access
    class ISOLATED access
```

### Rollout via feature flags

```mermaid
graph TD
    ALPHA[Alpha<br/>internal testing only] --> BETA[Beta in production<br/>enabled via config flag<br/>for select customers]
    BETA --> PREPROD[Pre-production testing<br/>implementation team + customer<br/>account managers test on staging]
    PREPROD --> GA[General availability<br/>rolled out to all<br/>qualifying customers]

    BETA --> MONITOR[Monitor for edge cases<br/>in real-world usage]
    MONITOR --> REFINE[Refine based on<br/>beta customer feedback]
    REFINE --> GA

    classDef alpha fill:#777,color:#fff,stroke:#555
    classDef beta fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef test fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef ga fill:#2d9d78,color:#fff,stroke:#1d7a5c

    class ALPHA alpha
    class BETA beta
    class PREPROD,MONITOR,REFINE test
    class GA ga
```

### Multi-tenancy: how isolation is maintained

```mermaid
graph TD
    ENV[Shared multi-tenant environment] --> T_A[Tenant A]
    ENV --> T_B[Tenant B]

    T_A --> UM_A[User Manager at Tenant A]
    UM_A --> SEE_A[Sees only Tenant A users<br/>manages only Tenant A roles]

    T_B --> UM_B[User Manager at Tenant B]
    UM_B --> SEE_B[Sees only Tenant B users<br/>manages only Tenant B roles]

    SEE_A --> ZERO[Zero cross-tenant visibility<br/>during and after rollout]
    SEE_B --> ZERO

    classDef env fill:#777,color:#fff,stroke:#555
    classDef tenant fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef um fill:#a3dcc8,color:#0a2a1e,stroke:#5dbfa0
    classDef scope fill:#2d9d78,color:#fff,stroke:#1d7a5c

    class ENV env
    class T_A,T_B,T_C tenant
    class UM_A,UM_B um
    class SEE_A,SEE_B,ZERO scope
```

## Before vs. after comparison

| Dimension | Before | After |
|-----------|--------|-------|
| **User management process** | Every change requires a ticket to the implementation team | Self-service via dedicated User Manager role and interface |
| **Implementation team load** | Drowning in user change requests alongside core work | Most common user management requests eliminated from their queue |
| **Time to make a user change** | Days (waiting in queue) | Minutes (self-service) |
| **Access scope** | Only option was full admin access (too broad) or no access (status quo) | Dedicated User Manager role sees only the user management page, nothing else |
| **Role independence** | N/A | User Manager role works independently of other assigned roles. Having User Manager doesn't grant extra access. Having other roles doesn't grant user management. |
| **Tenant isolation** | Implicit (implementation team handled manually) | Enforced by design. Zero cross-tenant data leaks during or after rollout. |
| **Organizational scale** | Works the same for 50-user and 100K+ user organizations | Same architecture, no modifications needed per customer size |
| **Rollout approach** | N/A | Alpha, beta (feature flag in production), pre-production testing with implementation team and customer, then GA |
| **Customer satisfaction** | Frustrated. Paying significant fees but can't manage own people. | Self-reliant. Manage users on their own timeline without support queue. |

## Key architectural decisions

**Why a new dedicated role, not extending the existing admin workspace:**
The admin workspace had access to platform-level configuration, tenant settings, and system-wide controls. Exposing it to customers, even partially, created too large a blast radius. A dedicated User Manager role with its own page meant: if we got the scoping wrong, the worst case was a user management bug, not a platform configuration breach. Minimizing blast radius was the deciding factor.

**Why role independence matters:**
A person could be both a supply planner (with access to planning dashboards) and a user manager (with access to user administration). These roles had to work independently. Adding User Manager should not grant any planning access. Removing User Manager should not revoke planning access. This sounds simple but required careful RBAC architecture where roles were purely additive and scoped to their own page boundaries.

**Why feature flags, not a hard launch:**
At 100K+ users in multi-tenant environments, a bug in user management could have cascading effects: wrong users getting wrong access, tenant data leaking across boundaries, or the implementation team losing their ability to override. The alpha-beta-GA approach via feature flags meant we could test in production with select customers, monitor for edge cases in real-world usage, and roll back instantly if something went wrong. The cost of a slower rollout was weeks. The cost of a bad launch was customer trust.

**Why demo docs for every change:**
The implementation team was the first line of support for customers using the new self-service feature. They needed to know exactly what every button did, what every configuration option meant, and what the expected behavior was for every workflow. The click-through demo document I created (screenshots, step-by-step walkthrough, updated with every change no matter how small) became the most-referenced artifact in the organization. It reduced implementation team questions, accelerated customer training, and became a tool the implementation team could hand directly to customers.
