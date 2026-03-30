---
name: user-stories-craft
description: "Skill for crafting high-quality user stories, epics, task breakdowns, acceptance criteria, and aligning product work across stakeholders (executives, managers, developers). Use this skill whenever the user mentions user stories, epics, backlog items, story writing, story splitting, INVEST criteria, acceptance tests, story mapping, story workshops, sprint/iteration planning involving stories, product backlog refinement, story points, velocity, release planning, or any task related to defining, estimating, breaking down, or prioritizing product features in an agile context. Also trigger when the user asks about communicating requirements between business and technical teams, writing stories for Linear/Jira/any backlog tool, creating features from vague product ideas, or translating stakeholder requests into actionable development work. Even if the user doesn't say 'user story' explicitly — if they're describing a feature, requirement, or product need that should be structured for agile development, use this skill."
---

# User Stories Craft

A comprehensive skill for writing, splitting, estimating, and managing user stories — from initial product vision down to developer-ready tasks. Grounded in Mike Cohn's *User Stories Applied* methodology and complemented by modern agile practices.

## When You're Asked to Help With Stories

Before jumping into writing, always understand the context by asking:

1. **Who are the users?** Identify user roles and personas. Don't write for a generic "user" — identify real roles (e.g., "Travel Manager", "Guest Traveler", "Finance Admin"). See `references/user-roles.md` for the full role modeling process.

2. **What's the scope?** Is this a brand new product, a new feature area, or a refinement of existing functionality? This determines whether to start with epics or jump to detailed stories.

3. **What's the planning horizon?** Stories should be sized to the horizon — large/vague epics for distant work, detailed stories for the next 1-2 sprints.

4. **Who is the audience?** The communication style changes dramatically:
   - **Executives/Directors**: Focus on themes, business value, strategic alignment, and release-level roadmaps. Use MoSCoW prioritization.
   - **Product Managers/Gerência**: Focus on epics, acceptance criteria, user role coverage, and sprint-level planning.
   - **Developers**: Focus on task breakdowns, technical acceptance tests, story points, dependencies, and implementation notes.

## The Three C's — Card, Conversation, Confirmation

Every user story has three aspects:
- **Card**: A short written description — a *reminder* to have a conversation, not a contract.
- **Conversation**: The collaborative discussion between customer team and developers that fleshes out details.
- **Confirmation**: Acceptance tests/criteria that document expectations and define "done."

The card is the most visible part, but the conversation is the most important part.

## INVEST Criteria

Every well-formed story satisfies INVEST:

- **Independent**: Avoid dependencies between stories. If two stories are coupled, combine them or find a different split dimension.
- **Negotiable**: Stories are not contracts. The card is a reminder to discuss, not a detailed specification. Include just enough detail to resume the conversation.
- **Valuable**: Every story must deliver value to a user or purchaser. Rewrite technical stories to express business value (e.g., "App starts without lag" instead of "Use connection pool").
- **Estimatable**: If developers can't estimate it, it's either too big, too vague, or the team lacks domain knowledge. Break it down or do a spike.
- **Small**: Stories should be completable in half a day to two weeks by one developer or pair. If larger, it's an epic — split it.
- **Testable**: If you can't write an acceptance test for it, it's not a good story. "The software should be easy to use" is not testable; "A new user can complete a purchase in under 3 minutes without help" is.

## Writing Stories — The Template

The Connextra format is the most widely used:

```
As a [user role], I want [function] so that [business value].
```

Guidelines:
- Write for ONE user, not plural ("A Job Seeker can..." not "Job Seekers can...")
- Use active voice ("A recruiter posts a job" not "A job is posted by a recruiter")
- Keep UI details out as long as possible — describe WHAT, not HOW
- Don't number story cards — use short descriptive titles instead

## Epic → Story → Task Decomposition

### Level 1: Themes / Strategic Initiatives
High-level business goals or product areas. Examples:
- "International flight support"
- "B2C self-service booking"
- "Cost center management improvements"

### Level 2: Epics
Large stories that span multiple sprints. An epic is simply a story too big to fit in one iteration. Examples:
- "A travel manager can book international flights with passport validation"
- "A guest traveler can search and book flights without a company account"

### Level 3: User Stories
Stories sized for a single sprint (INVEST-compliant). Each should be a "full slice of cake" — cutting through all layers (UI, API, DB). Examples:
- "A travel manager can enter passport details for a traveler"
- "A guest traveler can search for domestic round-trip flights"

### Level 4: Tasks
Developer-level implementation work for a single story. Tasks are technical and owned by individuals, unlike stories which are owned by the team. Examples:
- "Create passport validation endpoint in .NET API"
- "Add passport fields to traveler profile schema"
- "Write integration tests for passport validation"

### Splitting Strategies

When a story is too big, split it along these dimensions (in order of preference):
1. **By workflow step**: Registration → Search → Book → Pay → Confirm
2. **By business rule variation**: "Pay with credit card" vs "Pay with invoice"
3. **By data variation**: "Search by city" vs "Search by airport code"
4. **By interface**: "Book via web" vs "Book via WhatsApp"
5. **By operation (CRUD)**: Create, Read, Update, Delete as separate stories
6. **By performance**: "Basic search" vs "Search with sub-second response"
7. **By priority within the story**: Must-have subset vs nice-to-have subset

**Never split by architectural layer** (don't create separate "UI story", "API story", "DB story"). Each story must deliver end-to-end value.

## Acceptance Criteria & Testing

Write acceptance criteria as concrete, testable scenarios. Use the format:

```
Given [precondition]
When [action]
Then [expected result]
```

Or as simple test reminders:
- Test with valid passport number (pass)
- Test with expired passport (fail)
- Test with passport from restricted country (show warning)
- Test with missing passport when route requires it (block booking)

Write tests EARLY — before or at the start of the sprint. Tests communicate expectations and save developer time by surfacing edge cases upfront.

## Estimation — Story Points

- Estimate as a team using Planning Poker (Wideband Delphi)
- Use relative sizing: a 4-point story is roughly twice a 2-point story
- Use a constrained scale: 1/2, 1, 2, 3, 5, 8, 13, 20, 40, 80
- Triangulate: compare each new estimate against previously estimated stories
- Story point estimates are owned by the team, not individuals
- Velocity = story points completed per sprint. Use measured velocity (not hoped-for) for planning.

## Stakeholder Alignment

### For Executives / Diretoria
- Present **themes and epics** with business value justification
- Use **MoSCoW prioritization** (Must/Should/Could/Won't)
- Show a **release roadmap** mapping themes to time horizons
- Communicate in business language — zero technical jargon
- Focus on: "What value does this deliver?" and "When can we expect it?"

### For Product Managers / Gerência
- Present **epics broken into stories** with acceptance criteria
- Show **sprint planning** with stories allocated by velocity
- Discuss **trade-offs**: if this story goes in, what comes out?
- Use story maps to visualize the user journey and identify gaps

### For Developers
- Present **stories with task breakdowns** and technical notes
- Include **acceptance tests** as concrete scenarios
- Clarify **dependencies** and **constraints** (constraint cards)
- Discuss **spikes** for unknown technical areas
- Agree on **definition of done** for each story

## Story Smells — Warning Signs

Watch for these anti-patterns:
- **Stories too small**: Causes estimation churn. Combine interdependent stories.
- **Interdependent stories**: Hard to plan. Combine or re-split.
- **Goldplating**: Developers adding unrequested features. Increase visibility via daily standups.
- **Too many details on the card**: You're documenting, not conversing. Use a smaller card.
- **UI details too early**: Keep interface out until you're close to implementation.
- **Thinking too far ahead**: Distant stories should be epics, not detailed specs.
- **Splitting too much**: If you're always splitting during planning, do a pre-refinement pass.
- **Customer can't prioritize**: Stories may be too big or not expressing business value clearly.

## Constraint Cards

Some requirements don't get estimated/scheduled — they must be *obeyed*:
- "The system must support 50 concurrent users" → Constraint
- "The software must run on all versions of Windows" → Constraint
- "Response time must be under 2 seconds" → Constraint

Write automated tests to verify constraints every sprint. Tape constraint cards on the wall as permanent reminders.

## Practical Workflow — Using This Skill

When the user asks for help, follow this flow:

1. **Understand context**: Ask about users, scope, planning horizon, audience
2. **Identify user roles**: Run through role modeling if not done yet
3. **Draft stories at the right level**: Epics for distant work, detailed stories for near-term
4. **Apply INVEST**: Check each story against all six criteria
5. **Write acceptance criteria**: Concrete, testable scenarios
6. **Break into tasks** (if requested): Technical tasks for developers
7. **Estimate** (if requested): Help structure a Planning Poker session
8. **Align for audience**: Translate the same work into executive, manager, or developer language

## References

For deeper guidance on specific topics:
- `references/user-roles.md` — Complete user role modeling process (brainstorm, organize, consolidate, refine, personas, extreme characters)
- `references/gathering-stories.md` — Story-writing workshops, interviews, questionnaires, observation techniques
- `references/planning-estimation.md` — Release planning, iteration planning, velocity tracking, MoSCoW prioritization

## Complementary Reading

These books pair well with Mike Cohn's *User Stories Applied*:
- **Jeff Patton — *User Story Mapping*** (2014): Adds the "story map" visualization technique for understanding the big picture of a product's user experience
- **Mike Cohn — *Agile Estimating and Planning*** (2005): Deep dive into estimation, velocity, and release planning
- **Roman Pichler — *Agile Product Management with Scrum*** (2010): Connects stories to product ownership and strategic vision
- **Marty Cagan — *Inspired*** (2017): Modern product management principles for deciding WHAT to build before writing stories
- **Kenneth Rubin — *Essential Scrum*** (2012): Comprehensive Scrum guide with strong story/backlog management chapters
