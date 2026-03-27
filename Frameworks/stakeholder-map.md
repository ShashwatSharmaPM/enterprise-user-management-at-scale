# Stakeholder Map: Enterprise User Management at Scale

> This feature transferred an existing capability from an internal team to the customer. That transfer had implications for every team that touched user management: the implementation team losing work from their queue, the engineering team building something without a flashy business case, the design team scoping a new interface, and customers gaining responsibility they'd never had before.

---

## Why This Project Had Unusual Stakeholder Complexity

Most feature builds have clear external demand: a customer asks for something, you build it, they use it. This project's demand was internal-first: the implementation team was drowning, and the customers were frustrated but hadn't framed "let us manage our own users" as a feature request. They framed it as "why does it take so long to add a user?"

That meant:
- The business case had to be constructed, not reported. Nobody was asking for "self-service RBAC." They were asking for "faster user changes."
- The implementation team was simultaneously the biggest beneficiary and the most concerned stakeholder. Self-service meant fewer tickets (good), but also less control and a new surface area for customer mistakes they'd have to clean up (concerning).
- Engineering had to build something that was architecturally harder than it looked (multi-tenant isolation, role independence, dedicated page boundary) for a feature that didn't have a revenue line item.
- Customers gaining self-service meant they also gained responsibility. Some welcomed it. Others weren't sure they wanted it.

---

## Stakeholder Map

### Implementation Team

**What they cared about**: Reducing their ticket queue, maintaining quality of customer configurations, not losing the ability to intervene when customers make mistakes.

**Their concern with this project**: Dual and contradictory. On one hand: "Please build this, we're drowning in user change requests." On the other: "If customers manage users themselves and break something, we'll be the ones cleaning it up. And we'll have less visibility into what's happening in each tenant."

**How I managed them**:
- Made them the primary input source for the problem definition. Instead of guessing what the pain was, I asked: what are the most common requests? How much time does each one take? What errors do customers make when they describe what they want? This grounded the feature in real operational data, not assumptions.
- Ensured the self-service capability was additive, not replacement. The implementation team retained full ability to manage users on behalf of customers. Self-service was a new path alongside the existing one, not a replacement for it. This was critical for their buy-in.
- Invited them into pre-production testing before any customer saw the feature. They tested the same workflows they'd been doing manually, now through the self-service interface. Their feedback caught edge cases that engineering hadn't anticipated (e.g., what happens when a user manager tries to assign a role the organization hasn't licensed).
- The demo document I created became their primary tool for customer training. Instead of explaining the feature verbally on every customer call, they could share the click-through walkthrough. This turned a concern ("now I have to teach customers how to use this") into a solved problem.

**What they needed from me**: Retained override capability, involvement in testing, a training tool they could hand to customers, and confidence that customer self-service wouldn't create more cleanup work than it saved.

**Alignment mechanism**: Problem definition interviews, pre-production testing participation, demo document as enablement tool, retained manual path.

---

### Engineering Team

**What they cared about**: Building something architecturally sound, not creating a security liability in a multi-tenant environment, getting clear requirements.

**Their concern with this project**: "This looks simple on the surface, but multi-tenant RBAC with role independence is genuinely complex. We need to get tenant isolation right, or this is a data leak waiting to happen. And this doesn't have the same business excitement as the marketplace or the plugin. Is this actually going to be prioritized long enough for us to build it properly?"

**How I managed them**:
- Involved engineering in the key architecture decision: dedicated User Manager role with its own page boundary (my recommendation) vs. extending the admin workspace (the simpler option). I presented the trade-offs (effort vs. blast radius) and let them weigh in on the technical feasibility. They agreed that a dedicated role was the right approach and were motivated by the clean architecture.
- The PRD clearly defined the scope boundary: what the User Manager role could and could not do. Engineering didn't have to guess about edge cases. Every boundary was documented upfront.
- The feature flag rollout (alpha, beta, GA) gave engineering confidence that they wouldn't ship a bug to 100K+ users on day one. Beta in production with select customers meant real-world testing with rollback capability.
- When the multi-tenancy edge case surfaced during beta (user count signal visible across tenants), I prioritized the fix immediately and didn't treat it as a low-priority cosmetic issue. Engineering saw that tenant isolation was genuinely non-negotiable, not just a talking point.

**What they needed from me**: Clear scope, a defensible architecture decision, phased rollout to reduce risk, and immediate priority for any tenant isolation issue.

**Alignment mechanism**: Architecture decision session, PRD with boundary definitions, feature flag rollout plan, zero-tolerance priority for isolation bugs.

---

### Design Team (UX)

**What they cared about**: Building a user experience that was intuitive, consistent with the rest of the platform, and appropriate for the task.

**Their concern with this project**: "Where does user management live in the platform? How do we make it discoverable for user managers but invisible to everyone else? And how do we handle the interaction design for organizations with 100K+ users without building a full admin console?"

**How I managed them**:
- Brought them in at the PRD stage, not after engineering had already built something. They participated in the go/no-go review and had a voice in the scope decision from the beginning.
- Defined the use cases and expected behavior (my role), then let the design team decide how it should look, feel, and behave in the interface (their role). Clear role separation. I didn't design screens. They didn't define business rules.
- The key design decisions: dedicated section in navigation visible only to User Manager role, bulk operations as a second-phase enhancement (MVP was single-user operations), and interaction patterns that worked for both 50-user and 100K+ user organizations.
- Figma designs were produced as formal artifacts alongside the PRD and technical design document, shared with the full cross-functional team in the go/no-go review.

**What they needed from me**: Early involvement, clear use cases, defined boundaries (what the feature should and shouldn't do), and respect for their ownership of the interaction design.

**Alignment mechanism**: Go/no-go review participation, use case documentation, shared Figma workspace, joint review sessions.

---

### Product Leadership (VPs of Product)

**What they cared about**: Roadmap coherence, revenue impact, competitive positioning, not spending engineering cycles on internal tooling that doesn't move customer-facing metrics.

**Their concern with this project**: "This is basically an internal ops tool. Our engineering team should be building features that drive customer growth, not replacing the implementation team's workflow."

**How I managed them**:
- Reframed the feature as revenue protection and platform maturity, not internal tooling. The pitch: "Our largest account is escalating weekly on user management delays. Every hour the implementation team spends on user change tickets is an hour they're not spending onboarding new customers. Self-service RBAC is table stakes for enterprise SaaS platforms. Not having it is a competitive gap."
- Showed the revenue proxy calculation: implementation team hours spent on user management tickets, translated to delayed onboarding capacity, translated to delayed recurring revenue.
- Positioned it within the broader platform narrative: the monolith-to-microservices migration made the platform modular, the API marketplace made it purchasable, and self-service RBAC makes it operationally scalable. Each piece builds on the last.
- Used the existing backlog exposure approach: showed what was in flight, what effort was assigned, and what would move to make room. No vague "we need to build this." Concrete trade-offs.

**What they needed from me**: A revenue connection (even indirect), proof that the largest customers needed this, and confidence that engineering cycles spent here generated more value than the alternative uses.

**Alignment mechanism**: Revenue proxy calculation, strategic account escalation data, backlog trade-off presentation, platform maturity narrative.

---

### Account Management Team

**What they cared about**: Customer satisfaction, managing expectations, not being surprised by changes that affected their accounts.

**Their concern with this project**: "My largest customer has been complaining about user management for months. When can I tell them this is fixed? And when it launches, will it create confusion or new problems I'll have to manage?"

**How I managed them**:
- Briefed account managers on the rollout timeline and what to communicate to customers at each phase. Before beta: "We're building self-service user management, targeted for [timeline]." During beta: "Your organization has been selected for early access. Here's what the User Manager role can do." At GA: "Self-service is live for all qualifying customers."
- Shared the demo document with account managers so they could walk customers through the feature in quarterly business reviews. The visual walkthrough was more effective than a verbal explanation.
- Involved account managers in the beta customer selection. They knew which customers were most frustrated and most likely to adopt self-service positively.

**What they needed from me**: Rollout timeline, customer communication guidance, the demo document as a training tool, and input into beta customer selection.

**Alignment mechanism**: Rollout briefings, demo document sharing, beta customer selection collaboration.

---

### Enterprise Customers (Especially the 100K+ Conglomerate)

**What they cared about**: Managing their own users without waiting in a support queue. Maintaining security. Not breaking anything.

**Their concern with this project**: "We want to manage our own users. But we have 100,000+ people across retail stores, distribution centers, and regional hubs. What if someone grants the wrong access? What if we accidentally delete users? What safeguards exist?"

**How I managed them**:
- The beta rollout started with the 100K+ conglomerate specifically because they had the highest pain and the most complex use case. If the feature worked for them, it worked for everyone.
- Customer-side account managers tested in pre-production before it went to the customer's own team. This meant the first people using the feature were familiar faces who could report issues in context.
- The dedicated User Manager role was the key trust-builder: it could only see and manage users, nothing else. The customer's IT team could be confident that a user manager couldn't accidentally modify platform configuration, access another tenant's data, or grant permissions beyond what the organization had licensed.
- For the concern about mistakes: the implementation team retained override capability. If a customer's user manager made an error, the implementation team could fix it through the existing path. Self-service was a new option, not the only option.

**What they needed from me**: A feature that was safe to use at 100K+ scale, clear boundaries on what the User Manager role could and couldn't do, and a fallback path if mistakes happened.

**Alignment mechanism**: Beta testing with customer-side account managers, pre-production validation, demo document for customer training, implementation team as safety net.

---

## How Stakeholder Dynamics Shifted Over the Project Lifecycle

| Phase | Who Had the Loudest Voice | My Focus |
|-------|--------------------------|----------|
| **Problem definition** | Implementation team (operational pain) | Quantifying ticket volume, mapping common requests, building revenue proxy |
| **Getting approval** | Product leadership (roadmap justification) | Revenue framing, platform maturity argument, backlog trade-off |
| **Architecture decision** | Engineering team (tenant isolation design) | Dedicated role vs. admin workspace, blast radius analysis |
| **Design and build** | Design team + engineering | Use case definition, Figma collaboration, PRD boundary clarity |
| **Beta rollout** | Enterprise customer (100K+ conglomerate) + implementation team | Pre-production testing, demo document, monitoring for edge cases |
| **GA and adoption** | Account managers + all customers | Communication guidance, training enablement, demo document distribution |

---

## Key Takeaway

The defining challenge of this project was making the invisible visible. Nobody was asking for "self-service RBAC." The implementation team was asking for "fewer tickets." Customers were asking for "faster user changes." Leadership was asking for "more customer onboarding capacity."

The PM job was to recognize that all three of these were the same problem, then build a solution that addressed all three while navigating the constraint that made it genuinely hard: multi-tenant isolation in a shared infrastructure environment. The feature wasn't technically glamorous, but it was the kind of platform investment that separates a product that scales from one that drowns in operational overhead.

---

*All company names, client names, and partner names have been anonymized. Stakeholder dynamics and management approaches are accurate as experienced.*
