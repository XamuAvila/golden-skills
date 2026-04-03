# Technical Team Management Reference

## Table of Contents
1. [Ten Pillars of Tech Culture](#ten-pillars-of-tech-culture)
2. [Tech Debt](#tech-debt)
3. [Technology Roadmap](#technology-roadmap)
4. [Tech Process / Workflow](#tech-process--workflow)
5. [Developer Experience (DX)](#developer-experience-dx)

---

## Ten Pillars of Tech Culture

### 1. Spend Team Time on Things That Matter to the Business
Use this as a decision-making tool. Example: Debating custom backend vs. framework? If the business needs to iterate on frontend, use the framework even if you rewrite in 18 months. Finding product-market fit earns the right to do the rewrite later.

### 2. Use Reliable Tools Where You Can, Innovate Where It Counts
"Don't reinvent the wheel." Use off-the-shelf components (libraries, cloud services, packages) whenever possible. Even if you ultimately customize, the experience with the off-the-shelf tool informs your custom build.

### 3. Automation Unlocks Velocity
Time-consuming tasks that don't add business value (replicating environments, running test suites, provisioning feature branches) impose a constant tax. Encourage the team to document whenever they spend >30 minutes on nonproductive technical work. Automate those tasks so nobody loses that time again.

### 4. Frequency Reduces Difficulty
"If it hurts, do it more often" (Martin Fowler). Painful tasks that are neglected get worse. Painful tasks done frequently get automated and improved. Example: releasing code weekly → painful at first → team automates → eventually continuous deployment with a button push.

### 5. Standardize the RFC Process
RFC = Request for Comment. Powerful tool for pseudo-democratically making technical decisions. Formalize it:
- Template markdown document in source control
- New RFC = PR introducing new markdown file
- "Votes" = PR approvals
- Finalization = PR merge
- Good RFC topics: naming conventions, code formatters, meeting cadences, monorepo vs manyrepo, API design philosophies
- Include a section on how RFC results are institutionalized (engineering guidebook, onboarding docs)

### 6. Make Speed a Goal, Not a Strategy
A goal = destination. A strategy = journey (diagnosis + guiding policy + action plan).
Example strategy for high-velocity engineering:
- **Challenge**: Early-stage startup, need PMF before running out of capital
- **Diagnosis**: Opportunity to minimize complexity. Pre-PMF code has minimal sunk cost value.
- **Action plan**: Hire generalists, reinforce curiosity/scrappiness, invest modestly in long-term planning, focus on shipping MVP experiments.

### 7. Participate in Continuous Education and Technology Conferences
Budget ~$2,000/engineer for conference attendance. Benefits: individual development, team learning (require written summaries), networking, sponsorship/branding opportunities, and recruiting.

### 8. Deploy Rubber Duck Debugging
Speak the problem out loud (to a rubber duck). Often reveals solutions without interrupting a coworker. Saves time and context-switching costs.

### 9. Build an Explainer Video Library
Use Loom or Slack to record 1-minute desktop/IDE screencasts with voiceover. Archive in internal wiki. Poor watch rates initially, but tremendous long-term reference value. Keep videos short, complete, standalone, and consistent.

### 10. Onboarding Is Everybody's Responsibility
Tribal knowledge grows daily. Actively convert siloed knowledge into scalable documentation. Everyone asks: "How can I document what I've just created/learned?" New employees who can't find answers should seek them out AND document them.

---

## Tech Debt

### The Debt Metaphor
Tech debt = financial debt. You take on a mortgage (deliberate tradeoff) knowing the consequences (interest), to enable something now (ship fast). Then pay it down consistently (monthly payments). Key: recognize debt is inevitable, and proactive paydown is a necessary investment.

### Definition
A technical decision, implementation, or nuance that actively reduces the efficiency or effectiveness of the business today or in the future. If ugly code has no business impact and won't need modification, it's NOT costly debt.

Don't fear debt — it serves a purpose. Building V1 with architecture that won't scale past 100 users is fine if it lets you validate the product fast.

### Seven Types of Tech Debt
1. **Architecture/Design Debt**: Design can't meet near-term or future business needs
2. **Code Debt**: Poor practices, hard to understand/maintain
3. **Test Debt**: Insufficient automated tests for confidence
4. **Infrastructure Debt**: Poorly maintained infra, difficulty scaling/deploying, poor uptime
5. **Documentation Debt**: Insufficient or stale docs
6. **Skill Debt**: Team lacks needed skills for maintenance/updates
7. **Process Debt**: Inconsistent problem-solving, causing mistakes/delays

### Tech Debt Bankruptcy
Signs you're bankrupt:
- Regular production outages with material business impact
- Constant pushback/exaggerated timelines on new features
- Team says codebase is "too complex to work in"
- Can't ship new features without breaking old ones

Action: Raise the alarm, reset expectations, devise a paydown plan. Your honesty with peers (see "Delivering Bad News") provides the credibility needed.

### Measuring Debt — The Debt Inventory
Take a survey 1-4 times per year with the team. For each type of debt, rate 1-10 and provide justification. Compare results over time. Don't do this alone — include engineers who interact with the debt daily.

### Strategies for Paying Down Debt
Allocate 10-20% of team time to non-feature engineering (debt paydown, POCs, DX improvement). Higher percentage if debt impact is more severe.

**Just-in-Time Payment**: Pay off debt as part of business-driven projects (adding debt tickets to sprints). Low overhead but risks: underinvestment, debt as secondary objective, perceived as "slowing down."

**Periodic Paydown**: Fixed intervals (day per sprint, 2 days/month, 2 weeks/quarter). Shape Up uses 2-week cooldown after 6-week cycle (25%). Google's "20% time" is a similar concept.

**Continuous Paydown**: Dedicated team (customer crew) with clear, measurable goals. Example: if test debt is highest, measure defect rates and code coverage.

### Communicating Tech Debt
Non-technical leaders don't expect perfection but do expect high performance. Communicate:
- Your strategy for keeping debt manageable
- When debt may block business goals
- Your plan for paying it down

---

## Technology Roadmap

### Three Timeframes

**Short-term / Horizon 1**: What the team is working on NOW. In-flight features, active defects, urgent work. Managed by sprint/workflow process.

**Medium-term / Horizon 2**: Spreadsheet where rows = teams/individuals, columns = time periods (weeks/sprints), cells = high-level work items. Purpose is NOT to predict precise completion dates but to set direction and order of operations. Update durations as engineering progresses. Useful for communication with other departments and as a retrospective tool.

**Long-term / Horizon 3**: Strategic goals. Produce a clear document (deck, video, wiki). Revisit infrequently — frequent direction changes are confusing and demotivating. Provide quarterly progress updates to engineering team and executive leaders.

Long-term initiative examples:
- Architectural debt (framework migration, Kubernetes onboarding)
- Language debt (consolidation, version upgrades)
- Platform adoption (serverless, SSR, edge computing)
- Hiring plans (growing/reorganizing teams)

### Timeline Communication
Provide regular transparency to other departments. Don't be a "black hole." Have a forum/mechanism to take input and incorporate it. Close the loop — communicate where requests are in the development process.

---

## Tech Process / Workflow

### Five Popular Workflow Patterns
1. **Agile**: Iterative development with flexibility
2. **SCRUM**: Prescriptive with sprints, ceremonies, roles
3. **Kanban**: Continuous flow, WIP limits
4. **Waterfall**: Sequential steps (oldest, most rigid)
5. **Shape Up** (Basecamp): 6-week cycles with 2-week cooldowns

The impact of HOW WELL you implement a process dwarfs the differences between processes. Pick one and implement it well.

### Truisms About Software Development
- Nobody can perfectly predict how long any task will take
- Engineering is rarely a straight line (feature X may require solving Y first)
- No specification is perfect — there are always gaps discovered during building

### Workflow Best Practices

**Estimates**: Inherently imprecise. Use relative sizing (story points) rather than absolute time. Never use estimates to punish or pressure teams. Estimates improve with practice but are never guarantees.

**Sprint planning**: Select work items collaboratively. Ensure stories are well-defined with clear acceptance criteria. Include tech debt alongside feature work.

**Retrospectives**: Regularly (every sprint). What went well? What didn't? What to change? Action items with owners. Don't skip retros — they're the primary mechanism for process improvement.

**Standups**: Daily, time-boxed (15 min max). What did you do? What will you do? Any blockers? Not a status report to management — it's team coordination.

### Technical Planning and Specifications

**Technical specs** (distinct from PRDs): Describe the "how." Living documents. Include:
- Problem statement and context
- Proposed solution with alternatives considered
- Data model changes
- API changes
- Migration strategy
- Testing plan
- Rollback plan
- Security considerations

### Cooldown/Innovation Sprints
Allocate time between cycles for: tech debt paydown, experimentation, learning new tools, proof of concepts. Shape Up's model: 2-week cooldown after every 6-week cycle.

---

## Developer Experience (DX)

Developer experience is how productive and satisfied your engineers are with their tools, processes, and environment. Poor DX = slow velocity, low morale, high turnover.

### Key DX Areas
- **Local development**: How fast can a developer go from `git clone` to running the app? Target: <30 minutes for new setup.
- **Build times**: CI should take <15 minutes. Invest in caches, parallel jobs, optimized builds.
- **Testing**: Tests should be fast, reliable, and easy to run locally.
- **Documentation**: Every repo needs README with installation, directory structure, development loop, deployment.
- **Tooling**: Invest in linters, formatters, IDE configs, code generators.
- **Environment parity**: Dev, staging, and production should be as similar as possible.

### Measuring DX
- Regular pulse surveys to the team
- Track cycle time metrics
- Monitor CI build times and flaky test rates
- Count how often developers are blocked by tooling/environment issues
