# User Role Modeling — Complete Guide

Based on Mike Cohn's *User Stories Applied*, Chapter 3.

## Why Role Modeling Matters

Most teams write all stories from the perspective of a single generic "user." This leads to software that ignores the needs of at least some user types. By identifying distinct user roles, you write stories that reflect the real diversity of your user base.

A **user role** is a collection of defining attributes that characterize a population of users and their intended interactions with the system.

## The 4-Step Process

### Step 1: Brainstorm Initial Roles

Gather the customer team and developers. Give everyone note cards. Each person writes role names on cards and places them on the table, announcing only the name. No discussion, no evaluation — pure brainstorming. Continue until ideas slow down (usually under 15 minutes).

**Rules:**
- Each role represents a SINGLE user (not "a company" — who at the company?)
- Prefer human roles over system roles
- Cast a wide net — you'll consolidate later

### Step 2: Organize the Set

Move cards on the table to show relationships:
- Overlapping roles → overlap their cards proportionally
- Completely overlapping → stack the cards
- Unrelated roles → keep cards apart

This visual layout reveals the structure of your user base.

### Step 3: Consolidate Roles

For cards that overlap significantly:
- Authors describe what they meant by each role
- Group decides if roles are equivalent → merge into one
- If roles serve distinctly different goals → keep them separate

Also: **remove roles that aren't important** to the success of the system. Don't try to satisfy everyone — focus on the roles that make or break the product.

### Step 4: Refine Roles with Attributes

For each surviving role, capture defining attributes:
- Frequency of use (daily? monthly? once?)
- Domain expertise level
- General computer proficiency
- Proficiency with THIS software
- Primary goal (convenience? rich experience? speed? comprehensiveness?)
- Any domain-specific attributes

Write these on the role card and hang role cards in the team's common area.

## Personas

For the most important roles, create a **persona** — an imaginary but vivid representation:
- Give them a name, a job, a backstory
- Include a photograph (stock photo, magazine cut-out)
- Describe enough that every team member feels they "know" this person
- Hang persona definitions in the team workspace

**Example:** "Mario works as a recruiter at SpeedyNetworks. He's been there 6 years, works from home on Fridays, considers himself a power user. SpeedyNetworks is always growing, so Mario is constantly looking for good engineers."

Stories become more expressive when written for a persona:
- Generic: "A user can search for jobs by location"
- With persona: "Allan (Geographic Searcher) can restrict his search to a specific region"

## Extreme Characters

A technique from Djajadiningrat et al.: imagine extreme users for your system (e.g., a drug dealer, the Pope, a person juggling multiple boyfriends) to surface stories you'd otherwise miss. This is optional and brief — a few minutes of creative thinking that might yield unexpected insights.

## Using Roles in Stories

After modeling, always use role names in your stories:
- ❌ "A user can post a resume"
- ✅ "A Job Seeker can post her resume"
- ✅ "As an Internal Recruiter, I want to filter candidates by experience level so that I can quickly shortlist qualified applicants"

This keeps real users in the forefront of everyone's mind during development.
