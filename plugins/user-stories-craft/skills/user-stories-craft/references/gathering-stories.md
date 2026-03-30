# Gathering Stories — Techniques Guide

Based on Mike Cohn's *User Stories Applied*, Chapter 4.

## Mindset: Trawling, Not Capturing

Requirements aren't "out there waiting to be captured." They emerge through conversation, evolve over time, and some die as the product changes direction. Think of gathering stories as trawling with a net:
- **First pass**: Large mesh net → get the big, obvious stories
- **Later passes**: Finer mesh → refine and discover smaller stories
- **Ongoing**: Stories mature, new ones emerge, old ones become irrelevant

## Technique 1: User Interviews

The default approach for most teams.

**Key principles:**
- Interview users from DIFFERENT user roles
- Don't just ask "what do you need?" — users often can't express their true needs
- Users have the problem, but they're not always qualified to propose the solution

**Question types:**

**Open-ended questions** (preferred):
- "What would you be willing to give up in order to have X?"
- "Walk me through how you do this today"
- "What's the most frustrating part of your current process?"

**Context-free questions** (no implied answer):
- ✅ "What kind of performance is required?"
- ❌ "How fast do searches need to be?" (implies there IS a search performance problem)
- ❌ "You wouldn't want to give up X for Y, would you?" (leading)

Start with context-free questions, then narrow down to specifics. This surfaces stories that would remain hidden if you jumped straight to detailed questions.

## Technique 2: Questionnaires

Good for:
- Gathering prioritization data from a large user population
- Getting answers to specific questions at scale

Bad for:
- Discovering NEW stories (no follow-up questions possible)
- Understanding nuance (can't follow interesting tangents)

Use questionnaires to complement interviews, not replace them.

## Technique 3: Observation

Watch users work with the current system (or a competitor's):
- Users often forget to mention things they do habitually
- Observation reveals the "invisible" workflows
- Watch for workarounds — they signal unmet needs

## Technique 4: Story-Writing Workshops

The most effective technique for kickstarting a project's story set.

**Setup:**
- Include the full customer team + as many developers as practical
- Provide note cards, markers, and a large table or wall
- Timebox to 2-4 hours (can extend if productive)

**Process:**
1. Review user roles and personas (do role modeling first if not done)
2. For each role, brainstorm stories the user would need
3. Think about what each role's GOALS are, then write stories supporting those goals
4. Don't evaluate or estimate during the workshop — just write
5. After brainstorming, do a quick pass to identify obvious epics and duplicates

**Tips:**
- Use the Connextra template: "As a [role], I want [function] so that [business value]"
- Write at varying levels of detail — epics for distant horizons, detailed stories for near-term
- Don't try to get every story — you'll discover more through iterations
- If the workshop stalls, switch to a different user role to spark new ideas

## How Much Upfront?

Look one release ahead (3-6 months). Write stories that decrease in detail as the time horizon increases:
- Next sprint: detailed stories with acceptance criteria
- Next 2-3 sprints: smaller stories, some notes
- Next release: epics with brief descriptions
- Beyond next release: themes / strategic areas only

Don't spend 3 months writing stories. Spend enough time to get a feel for scope and cost, then start building and refining.

## Working with User Proxies

When real end users aren't available, work with proxies. Each type has strengths and risks:

| Proxy Type | Strength | Risk |
|---|---|---|
| Users' manager | Knows the domain | May not use the software day-to-day |
| Development manager | Understands technical feasibility | Biased toward technical concerns |
| Sales team | Knows customer pain points | Biased toward features that sell |
| Domain experts | Deep subject knowledge | May overcomplicate |
| Marketing | Understands market positioning | May not know real usage patterns |
| Former users | Real experience | Knowledge may be outdated |
| Trainers/Support | Know common problems | See only the problem areas |
| Business analysts | Skilled at requirements | May over-document |

**When working with proxies:**
- Be explicit that they're proxies, not the actual end user
- Seek multiple proxy perspectives to reduce individual bias
- Validate proxy-written stories with real users whenever possible
