# Tech Architecture Reference

## Table of Contents
1. [Architecture Patterns](#architecture-patterns)
2. [Writing Readable Code](#writing-readable-code)
3. [Coding Patterns](#coding-patterns)
4. [API Contracts](#api-contracts)
5. [Data and Analytics](#data-and-analytics)
6. [Architecture Best Practices](#architecture-best-practices)
7. [Tools and Technology Radar](#tools-and-technology-radar)
8. [DevOps](#devops)
9. [Testing](#testing)
10. [Source Control](#source-control)
11. [Production Escalations](#production-escalations)
12. [Security and Compliance](#security-and-compliance)

---

## Architecture Patterns

### Domain-Driven Design (DDD)
Core concepts:
- **Domain model**: Business concepts as objects in your system
- **Ubiquitous language**: Consistent vocabulary across the company
- **Bounded context**: Boundary within which the domain model applies

### Monolithic Architecture
All code runs as a single process. Information moves in memory via function calls.

Key features:
- Single deployment unit, single repository
- Scaled as a single unit
- DDD and clear information flow are NOT enforced by the system — depends on engineer discipline

Success requires careful data flow design. When a developer changes functionality, it should be obvious WHERE in the monolith to work. Each additional area needing changes adds complexity.

### Service-Oriented Architecture (SOA) / Microservices
System where information moves between parts over a network.

Key features:
- Independent deployment and scaling per service
- Single repo (monorepo) or many repos
- Information moves via HTTP, RPC, or queuing
- Data contracts must be intentionally designed as APIs

You do NOT need thousands of microservices — even 4-5 well-designed services can provide major improvement.

### Choosing: Monolith vs. SOA
**Monolith first** for the vast majority of problems on day one. Simpler setup, fewer logistics. A well-disciplined team can scale a monolith forever.

Move to SOA when:
- Elements need independent scaling (one feature is CPU-heavy)
- Functionality needs its own independent API with exclusive data domain
- You need a different programming language for a specific problem
- Deploying the monolith is overly slow/risky

### The Distributed Monolith (Anti-pattern)
Multiple services that are NOT independently deployable. Worst of both worlds. Red flags: coordinating releases between services, reasoning about cross-service effects. Fix: pay down API contract and data handling debt.

### Monorepo vs. Manyrepo
**Monorepo**: Easy dependency management, good discoverability, but CI may not support multiple packages natively.
**Manyrepo**: Requires central package manager with versioning (overhead), integrates cleanly with CI/CD.

Advice: Keep simple. Small-to-medium → monorepo. 50+ developers with a dedicated platform team → consider manyrepo.

---

## Writing Readable Code

**Golden rule of programming**: The principal audience for code is the developer who reads it next, not the computer.

### Language Choice Considerations
- Talent pool familiarity (especially in startup ecosystem)
- Existing implementations/starting points
- Performance/scaling requirements
- Framework availability (e.g., React Native requires JS/TS)
- Prefer static type systems (Go, TypeScript, Rust) — compiler does more heavy lifting for correctness

### Code Style and Formatting
- Use published standards (PEP8) or configurable tools (ESLint, Prettier, ReSharper)
- 100% of codebase formatted by same rules
- Auto-format on save in IDE
- Enforce in CI — but ONLY after setting up auto-formatting (otherwise frustrating)

### Static Code Analysis
Tools identify security gaps, bugs, stylistic issues. Good signal-to-noise ratio. Integrate early. Use language-specific tools (ESLint) AND generic platforms (SonarCloud, Code Climate).

### Greenfield vs. Brownfield
**Greenfield**: New environment, free tool choice. Risk: poor decisions with too much choice, underestimated bootstrap cost.
**Brownfield**: Existing legacy systems. Risk: "Not Invented Here" syndrome. Make explicit space for teams to READ and UNDERSTAND existing systems before modifying them.

---

## Coding Patterns

**Object-Oriented Programming (OOP)**: Models real-world nouns/verbs. Languages: Python, Ruby, C#, Java.

**Purity**: No external dependencies or side effects. Same inputs → same outputs. Easily testable, easier to read. Where possible, write pure code.

**Functional Programming**: Functions as first-class citizens. Composed from tiny pieces. Benefits: purity, testability, formal reasoning. Risk: verbose/hard-to-read chains of composed functions.

**Test-Driven Development (TDD)**: Tests written BEFORE functional code. Also: Behavior-Driven (BDD), Acceptance-Test Driven (ATDD).

**Dependency Injection**: Pass dependencies in rather than instantiating. Benefits: decreased coupling, documented interfaces, mockable in tests. Use established frameworks for your language.

**Domain-Driven Design**: Model your business domain consistently. With complex domains, sit down as a team and agree on consistent terminology across the entire system.

---

## API Contracts

### API Design Governance
Consistency, predictability, and correctness across the organization. Range from documented guidelines to formal review boards. Larger teams need more governance.

### HTTP-Based APIs

**REST**: JSON over HTTP. Uses HTTP verbs (GET/PUT/POST/DELETE) on noun endpoints. Most common, broad tool ecosystem.

**GraphQL**: JSON over HTTP but everything is POST. Structured schema of queries/mutations. Benefits: types, self-documenting schema, federation. Tradeoffs: no GET caching, tooling still catching up. Strongly consider for internal use cases.

**SOAP/XML**: Out of fashion. Don't build new SOAP APIs. Still prevalent in legacy enterprise systems.

### Non-HTTP APIs

**Queuing systems**: Inbox-based with delivery guarantees (FIFO/LIFO, at-least-once/at-most-once). AWS SQS, Google Cloud Tasks.

**Pub/Sub**: Publishers create messages, subscribers consume. One-to-one, one-to-many, many-to-one, many-to-many. RabbitMQ, AWS SNS, Google Cloud Pub/Sub. Powerful but requires careful attention to acknowledgement and subscriptions.

**Job systems**: Timer-triggered (cron). Best practices: use existing systems, ensure logging, retry configuration, failure notifications, job status UI, config in source control, secret management integration.

**General advice**: If torn between queues/pub-sub/HTTP, keep it simple — go with synchronous HTTP API. Being torn = async guarantees aren't critical = complexity not worth it.

### Idempotency
Same request multiple times = same effect as once. REST: all verbs except POST should be idempotent. For POST/mutations, implement idempotency keys (client-provided strings for de-duplication). Non-trivial to implement — adopt a framework that provides it.

### API Documentation
Must be: always up-to-date, document all inputs/types, document all errors, easy to navigate. Use auto-generation: OpenAPI for REST (+ stoplight.io, readme.com), GraphQL Playground/Apollo Studio for GraphQL. Portable to IDEs like Postman/Insomnia.

---

## Data and Analytics

### Three Data Types

**Transactional**: Powers the application. Low latency, high availability, modest size. Use off-the-shelf SQL/NoSQL (MongoDB Atlas, Google Cloud SQL). Must-haves: point-in-time restore, backups, read replicas, multi-zone, audit logging, monitoring, slow query monitoring.

**Analytical/BI**: Insights from transactional data. Early: query read-only replica. Later: data pipeline to warehouse. Recommendations: Snowflake/Databricks/BigQuery for warehouse, dbt for pipeline-as-code, Looker/Domo/Preset for visualization.

**Behavioral**: How users interact. High volume, limited schema. Use a CDP (Segment, RudderStack) to route to multiple destinations. Behavioral data is lossy (ad blockers, dropped requests) — don't use for exact numbers.

### General Data Advice
Most startups do NOT have "big data." PostgreSQL with reasonable hardware handles tens of millions of rows and hundreds of GB. Don't prematurely optimize database architecture.

---

## Architecture Best Practices

1. **Put business logic in the backend**: Easier to test, thinner clients, single source of logic, tamper-proof
2. **Make services externalizable**: Design internal APIs as if they could be consumed by third parties. Forces good habits (DDD, sensible auth, appropriate abstractions).
3. **Use as few languages as possible**: Each additional language replicates build systems, dependency management, best practices, and interfaces. Justify new languages with bulletproof arguments.

---

## Tools and Technology Radar

### Internal Technology Radar (inspired by Thoughtworks)
Categories: Hold → Assess → Trial → Adopt

Process:
1. Someone proposes a new tool/technique ("assess")
2. Make a written case for material benefit → approved → move to "trial"
3. Use it in a real project. Write follow-up: pros/cons, ecosystem fit.
4. Team decides: "adopt" (available to all) or "hold" (requires new trial to use again)

### "Boring Technology"
Total cost = maintenance cost − velocity benefit. Hidden costs of "not boring": immature docs, incomplete ecosystems, higher defect likelihood, training cost, update burden.

### Tool Cost
Typical SaaS COGS: 10-30% of revenue. Know your spend, use cloud budgeting features, track with SMPs. Factor regular growth into forecasts. Don't avoid adoption for cost alone — consider ROI.

---

## DevOps

### Four Key Metrics (DORA)
From Google's DevOps Research and Assessment program (7+ years of research):
1. **Lead Time**: Commit → production
2. **Deployment Frequency**: How often code reaches production
3. **Mean Time to Recovery (MTTR)**: Service restoration after incident
4. **Change Fail Percentage**: Releases needing hotfix/rollback

High scores require investment in automation, DevOps, testing, and culture. Quick assessment: dora.dev survey. Tools: LinearB, Code Climate.

### Reproducibility
Minimize human error through automation. Two critical tools:

**Containerization (Docker)**:
- Declarative Dockerfile → build into image → run anywhere with identical results
- Design: build once, run anywhere (extract env differences to runtime variables or secret stores)
- Build images in CI
- Use hosted registries (Dockerhub, cloud registries) with vulnerability scanning
- Keep images small (multi-stage builds, clean up in each layer)

**Infrastructure as Code (IaC)**:
- Terraform (HCL) is the leading tool
- Define resources declaratively, manage like code (source control, peer review)
- Generate change plans, apply to cloud providers
- ClickOps = manual UI configuration. Fine for prototypes, not for production.

### Container Orchestration

**Hosted** (recommended for most startups): Heroku, Google App Engine, Elastic Beanstalk, Cloud Run, Vercel. More expensive but dramatically less overhead.

**Self-managed (Kubernetes/K8s)**: Extremely powerful but steep learning curve. Don't learn on the job — invest 1-2 weeks upfront with a book and sandbox. Seek a K8s mentor.

### Continuous Integration (CI)
Automating code incorporation: static analysis, tests, builds, artifacts. Platforms: GitHub Actions, GitLab, Bitbucket Pipelines, Jenkins, CircleCI.

Best practices:
- Team understands CI system and can troubleshoot
- Builds are consistent and deterministic (no flaky builds!)
- Target: <15 min build time
- Use caches, artifacts, parallel jobs
- Keep CI code DRY
- Consistent secret management (cloud secret manager OR build env secrets, not mixed)

### Continuous Deployment
Release every day or every hour. "Frequency Reduces Difficulty" → automate the hard parts.

Benefits: code goes out faster (reduced lead time), deploy more often (increased frequency), improved MTTR. That's 3 of 4 DORA metrics with one initiative.

Every team that invested in CD described it as "transformational" to culture, output, and velocity.

### Feature Branch Environments
Every branch gets its own staging environment. Eliminates single-staging contention. Enables testing without building locally.

Considerations: automate setup, handle data carefully (new DB per branch, seed scripts not production data), plan schema migrations, set up service discovery for SOA.

### Feature Toggles
Switches to change system behavior without changing code. Decouple shipping code from shipping features. Engineering deploys continuously; business controls when features go live (marketing coordination, regulatory approval, support documentation, etc.).

### Managing DNS
Know DNS basics + email DNS (SPF, DKIM/DMARC). Set up DNS with Terraform/IaC. Don't gate DNS changes on a single person's credentials.

### System Monitoring
**APM**: Inside your app, monitors backend performance (resource usage, latency, throughput).
**RUM**: External, monitors from user's perspective.
Choose based on blind spots. Tools: New Relic, Datadog, Akamai mPulse.

---

## Testing

### Philosophy
Goal: confidence that software does what it's intended to do when all tests pass. May be 100% coverage or 30% — determine based on defect rates and team sentiment.

### Testing / QA Teams
Ensure dev and test teams share goals/KPIs, have robust communication, and maintain a healthy relationship. Testers need deep understanding of what code should do.

### Test Quality
**Bad tests**: Cost more to maintain than bugs they catch, high false positive rate, validate wrong behavior, convoluted/unmaintainable, don't instill shipping confidence.

**Good tests**: Easy/fast to run (local + CI), reliable, easy to augment, easy to refactor, appropriately coupled, easy to understand.

**Evaluation**: Ask the team how they feel about tests. Results are binary — either security blanket or productivity drain.

### What to Test
Test the public interface. Public interfaces are stable, well-thought-out, and represent what consumers experience. Tests on public interfaces change little over time.

### Five Testing Paradigms

**Unit tests**: In-memory, no network/external dependencies. Fast, small scope, tightly coupled to code (main downside — refactors require test updates). Don't over-rely on unit tests alone.

**Integration tests**: Test interactions between components. Broader scope, catch interface mismatches. More realistic but slower to run, need external dependencies.

**End-to-end tests**: Test full user workflows. Highest confidence but slowest, most brittle, most expensive to maintain. Use sparingly for critical paths.

**Manual testing**: Human testers explore the application. Catches UI/UX issues automation misses. Expensive, not scalable, but valuable for exploratory testing.

**Semi-automated testing**: Hybrid approach. Automated setup + human verification. Good for visual testing, complex workflows.

### Testing Best Practices
- Write tests that boost confidence while minimizing maintenance burden
- Prefer testing public interfaces over internal implementation
- Use a blend of paradigms — each type adds value in different areas
- Monitor false positive rate — flaky tests are a massive productivity drain
- Code coverage is a means, not an end

---

## Source Control

### Branch Strategy
Use a branching model appropriate to your team size and deployment frequency. Common strategies: trunk-based development (for CD), Git Flow (for releases), GitHub Flow (simple feature branches).

### Code Review
- Every change reviewed by at least one other engineer
- Reviews should be timely (same day ideally)
- Focus on: correctness, readability, test coverage, architecture alignment
- Use review as a teaching/learning opportunity
- Don't let perfect be the enemy of good — approve with suggestions for non-blocking issues

### Pull Request Best Practices
- Keep PRs small (easier to review, fewer defects, faster merges)
- Include context: what changed, why, how to test
- Link to tickets/specs
- Include screenshots for UI changes

---

## Production Escalations

### Incident Response
Have a clear process BEFORE incidents happen:
- Define severity levels
- Define who gets paged and escalation paths
- Establish communication channels (dedicated Slack channel, status page)
- Designate incident commander role

### Post-Incident Reviews (Postmortems)
- Conduct blameless postmortems for every significant incident
- Document: timeline, root cause, impact, resolution, preventive actions
- Focus on systemic improvements, not individual blame
- Track action items to completion
- Share learnings broadly

---

## Security and Compliance

### Security Basics
- Principle of least privilege for all access
- Encrypt data at rest and in transit
- Regular security audits and penetration testing
- Keep dependencies updated (automated vulnerability scanning)
- Implement proper authentication and authorization
- Secret management (don't hardcode secrets, use vault systems)

### Compliance
- Understand regulatory requirements for your industry (SOC 2, GDPR, HIPAA, PCI-DSS)
- Start compliance preparation early — retrofitting is expensive
- Document security practices for due diligence
- Consider compliance automation tools

### IT Considerations
- Device management for remote teams
- SSO and identity management
- Access provisioning/deprovisioning processes
- Data backup and disaster recovery plans
