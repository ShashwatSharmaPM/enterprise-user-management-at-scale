# Enterprise User Management at Scale

> Designing self-service RBAC for a multi-tenant SaaS platform with 100K+ users, turning an internal bottleneck into a customer-facing capability.

## Context

**Company type**: Global supply chain SaaS platform (demand and supply planning)
**My role**: Lead Product Manager, APIs and Platform
**Environment**: Multi-tenant platform serving enterprises from retail to FMCG to logistics

---

## The Problem

As the platform scaled and onboarded increasingly large, complex organizations, user management became an unsustainable bottleneck.

One customer alone, a large conglomerate, had over 100,000 users on the platform. These users spanned retail stores, distribution centers, regional hubs, and headquarters. People joined, people left, roles changed. And every single user change required a request to our implementation team.

The implementation team was stretched thin. They were handling user provisioning, role changes, and access requests alongside their core work of configuring the platform for new customers. The queue was growing, response times were slipping, and large customers were frustrated that they couldn't manage their own people on a platform they were paying significant fees for.

---

## The Constraints

This wasn't a simple "add a user management page" feature. Several factors made it complex:

**Multi-tenancy**: The platform served multiple customers on shared infrastructure. Any user management capability had to be airtight about tenant isolation. A user manager at Organization A must never see, access, or even know about Organization B's users or data.

**Granular access control**: We couldn't just give customers admin access to the platform. The user management capability needed its own dedicated role that could only see and modify user-related settings, nothing else. If the same person had other roles assigned, those should work independently.

**Varied organizational structures**: Some customers had flat structures (50 users, one location). Others had deep hierarchies (100K+ users across dozens of entities, regions, and functional areas). The solution needed to scale across both without forcing a one-size-fits-all model.

---

## What I Did

### Defining the Problem and the Boundary

My role was to frame the problem, define the use cases, and establish what success looked like:

- **Who needs this?** Large, self-reliant organizations with high user turnover
- **What should they be able to do?** Add users, remove users, assign/revoke roles, all within their own tenant boundary
- **What should they NOT be able to do?** See anything outside their tenant, modify platform-level configurations, or accidentally grant permissions beyond what the organization has licensed

### Cross-Functional Design

I worked with three teams:

- **Engineering team**: How do we implement tenant-isolated RBAC? Do we give user managers access to the existing admin workspace, or create a separate role with its own dedicated interface? (We chose the latter: a dedicated "User Manager" role that only surfaced the user management page.)

- **Design team**: Where does this page live in the platform navigation? How should it look and feel? What's the interaction model for adding/removing users in bulk? (Design created a dedicated section accessible only to users with the User Manager role.)

- **Implementation team**: What's the current pain, specifically? What are the most common requests? What error patterns do you see? (This grounded the feature in real operational data, not assumptions.)

### Building With Feature Flags

We used the platform's existing feature flag system (alpha, beta, production tiers) to roll this out gradually:

1. **Pre-production testing**: Implementation teams and customer-side account managers tested the feature in a staging environment
2. **Beta in production**: Enabled via configuration flag for select customers, allowing real-world usage while monitoring for edge cases
3. **General availability**: Rolled out to all qualifying customers after beta validation

### Artifacts I Produced

- **PRD**: Problem statement, use cases, success criteria, effort estimation
- **Figma collaboration**: Worked with the design team on interaction patterns and page placement
- **Technical design input**: Contributed to the discussion on whether to use existing admin roles or create a new role type (advocated for the new role to minimize blast radius)
- **Demo document**: Click-through walkthrough with screenshots of every UI element, button, and workflow, what it does, what it means, and what the user should expect. Updated every time even a small change was made.

---

## Outcomes

- **Implementation team load reduced significantly**: The most common user management requests no longer required internal team involvement
- **Scalable for organizations of any size**: Worked for 50-user organizations and 100K+ user organizations without architectural changes
- **Tenant isolation maintained**: Zero cross-tenant data leaks during or after rollout
- **Customer satisfaction**: Large enterprises could now manage their own users on their own timeline, without waiting in a support queue

---

## What I Learned

1. **The best platform features are the ones that eliminate internal dependencies.** This feature didn't add a new capability for end users. It transferred an existing capability (user management) from our team to the customer. The value wasn't in what it did. The value was in who could now do it.

2. **Scoping RBAC is a product decision, not just a technical one.** The question of "what should a User Manager role be able to see?" is fundamentally a product question about trust, liability, and customer expectation. Engineering can implement any permission model. Deciding which model to implement is the PM's job.

3. **Multi-tenancy makes everything harder, and you have to respect that.** Every feature on a multi-tenant platform has an implicit requirement: "and it must not leak data across tenants." This sounds obvious but has deep implications for how you design, test, and roll out features.

4. **Demo docs are underrated.** The click-through demo documents I created for every feature became the most-referenced artifacts in the organization. They reduced support questions, accelerated customer onboarding, and gave the implementation team a tool they could hand directly to customers.

---

*All company names, client names, and partner names have been anonymized. Metrics and technical details are accurate as described.*
