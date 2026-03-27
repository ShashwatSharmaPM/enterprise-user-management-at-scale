# Before state: manual user management at 100K+ scale

> Every user change required a ticket to the implementation team. One conglomerate alone had 100,000+ users across retail stores, distribution centers, and regional hubs. Personnel joining, leaving, changing roles daily. Queue growing, response times slipping.

```mermaid
graph TD
    subgraph ORG[" Large enterprise customer — 100K+ users "]
        direction TB
        RETAIL[Retail store employees]
        DC[Distribution center staff]
        REGIONAL[Regional hub planners]
        HQ[Headquarters analysts]
    end

    ORG -->|User needs to be added,<br/>removed, or role changed| TICKET[Support ticket created]
    TICKET --> IMPL[Implementation team<br/>processes request manually]
    IMPL --> PLATFORM[Platform updated]

    classDef org fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef users fill:#a3dcc8,color:#0a2a1e,stroke:#5dbfa0
    classDef ticket fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef impl fill:#d94f4f,color:#fff,stroke:#b33a3a
    classDef plat fill:#777,color:#fff,stroke:#555

    class ORG org
    class RETAIL,DC,REGIONAL,HQ users
    class TICKET ticket
    class IMPL impl
    class PLATFORM plat
```

### Why the implementation team was drowning

```mermaid
graph TD
    CUST_A[Customer A<br/>100K+ users] -->|User change requests| QUEUE[Implementation team queue]
    CUST_B[Customer B<br/>5K users] -->|User change requests| QUEUE
    CUST_C[Customer C<br/>50K users] -->|User change requests| QUEUE

    QUEUE --> BACKLOG[Queue growing<br/>response times slipping]
    BACKLOG --> FRUSTRATED[Large customers frustrated<br/>can't manage own people]

    QUEUE --> COMPETING[Competing with core work<br/>new customer onboarding,<br/>platform configuration]

    classDef cust fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef queue fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef problem fill:#e8a0a0,color:#2a0a0a,stroke:#c27070
    classDef bad fill:#d94f4f,color:#fff,stroke:#b33a3a

    class CUST_A,CUST_B,CUST_C cust
    class QUEUE queue
    class BACKLOG,COMPETING problem
    class FRUSTRATED bad
```

### Why "just add an admin page" wasn't the answer

```mermaid
graph TD
    REQ[Customer needs user management] --> OPTION{Give them admin access?}

    OPTION -->|Full admin access| RISK_1[Can see platform-level config<br/>not just user management]
    OPTION -->|Full admin access| RISK_2[Can modify settings<br/>beyond their scope]
    OPTION -->|Full admin access| RISK_3[In multi-tenant environments<br/>could leak data across tenants]

    RISK_1 --> NOGO[Too risky<br/>need a scoped solution]
    RISK_2 --> NOGO
    RISK_3 --> NOGO

    classDef req fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef option fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef risk fill:#e8a0a0,color:#2a0a0a,stroke:#c27070
    classDef bad fill:#d94f4f,color:#fff,stroke:#b33a3a

    class REQ req
    class OPTION option
    class RISK_1,RISK_2,RISK_3 risk
    class NOGO bad
```

### The multi-tenancy constraint

```mermaid
graph TD
    ENV[Shared multi-tenant environment] --> T_A[Tenant A<br/>100K+ users]
    ENV --> T_B[Tenant B<br/>different organization]
    ENV --> T_C[Tenant C<br/>different organization]

    T_A --> WALL{Tenant isolation<br/>must be airtight}
    T_B --> WALL
    T_C --> WALL

    WALL --> RULE[User manager at Tenant A<br/>must never see, access, or know about<br/>Tenant B or Tenant C data]

    classDef env fill:#777,color:#fff,stroke:#555
    classDef tenant fill:#4a8bc2,color:#fff,stroke:#3670a0
    classDef wall fill:#c68a2e,color:#fff,stroke:#a06e1e
    classDef rule fill:#d94f4f,color:#fff,stroke:#b33a3a

    class ENV env
    class T_A,T_B,T_C tenant
    class WALL wall
    class RULE rule
```

## Pain points summary

| Problem | Impact |
|---------|--------|
| **Every user change requires a ticket** | Adding, removing, or re-roling a user means filing a request to the implementation team. At 100K+ users with daily personnel changes, this is unsustainable. |
| **Implementation team stretched thin** | User management requests compete with core work: new customer onboarding, platform configuration, bug triage. Queue grows, response times slip. |
| **Large customers frustrated** | Enterprises paying significant platform fees cannot manage their own people without waiting in a support queue. |
| **Admin access is too broad** | Giving customers full admin access exposes platform-level configuration, settings beyond user management, and in multi-tenant environments, risks cross-tenant data leaks. |
| **Multi-tenancy makes scoping hard** | Any self-service user management must be airtight about tenant isolation. A user manager at Organization A must never see Organization B's data. |
| **Varied organizational structures** | Some customers have 50 users in one location. Others have 100K+ across retail, distribution, regional, and HQ levels. Solution must scale across both. |
