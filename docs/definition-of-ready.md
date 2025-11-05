# Definition of Ready (DoR)

## Overview

A user story is **"Ready"** when it meets all criteria below. Stories that don't meet DoR should **not** enter Sprint Planning. This ensures the team has clear, actionable work to estimate and implement.

## Criteria for Ready Stories

Before a story is added to a sprint backlog, it must satisfy ALL of the following:

### 1. User Story Statement Defined
- [ ] Story follows format: **"As a [role], I want [feature], so that [benefit]"**
- [ ] Role is clear and specific
- [ ] Feature is a user-facing action or capability
- [ ] Benefit explains the value/outcome

**Example:**
> "As a user, I want to save my search filters, so that I can quickly re-run previous searches without re-entering all parameters"

---

### 2. Acceptance Criteria Defined (Minimum 3)
- [ ] Written in **Given-When-Then** format
- [ ] At least 3 criteria defined
- [ ] All criteria are testable and verifiable
- [ ] Criteria cover happy path + edge cases
- [ ] No ambiguous language ("should", "might", "could")

**Example:**
```
Given: User is logged in and on the Search page
When: User clicks "Save Current Search"
Then: A modal appears with a "Search Name" input field

Given: User has entered a search name
When: User clicks "Save"
Then: The search is saved and appears in "My Saved Searches" list

Given: User attempts to save without entering a name
When: User clicks "Save"
Then: An error message appears: "Please enter a search name"
```

---

### 3. Story Points Estimated
- [ ] Team has estimated the story using Planning Poker or similar
- [ ] Estimate is on Fibonacci scale: 1, 2, 3, 5, 8, 13, ?
- [ ] Team agrees the estimate is reasonable
- [ ] If estimate is "?", story needs more clarification first

---

### 4. Priority Assigned
- [ ] Priority is set: **High**, **Medium**, or **Low**
- [ ] Priority reflects user value and urgency
- [ ] Rationale for priority is documented (if not obvious)

**Priority Guidelines:**
- **High:** Core user journey, blocks other stories, high user value
- **Medium:** Nice-to-have, secondary features, can be deferred
- **Low:** Polish, edge cases, can ship without it

---

### 5. Dependencies Identified
- [ ] Story is marked as **Independent** OR all dependencies are listed
- [ ] If dependent, blocking stories are identified
- [ ] Blocking stories are prioritized higher
- [ ] Team understands the dependency chain

**Example:**
> "Depends on: #12 (User authentication) - Must be completed first"

---

### 6. Technical Questions Answered
- [ ] Product Owner has answered all team questions
- [ ] Story is not blocked on external information
- [ ] Design/mockups provided (if applicable)
- [ ] Any ambiguity is resolved

**Questions to ask:**
- How does this integrate with existing features?
- Are there performance constraints?
- What should happen in error cases?
- Do we need external APIs or third-party services?

---

### 7. Mockups/Wireframes Attached (if applicable)
- [ ] For UI/UX stories: wireframe or mockup is linked
- [ ] Mockup shows key interactions
- [ ] Design annotations explain flow/intent
- [ ] Team has reviewed and understands the design

---

### 8. Story is Testable & Unambiguous
- [ ] QA can write test cases based on acceptance criteria
- [ ] There's a clear way to verify the story is done
- [ ] No vague requirements ("make it better", "optimize", "improve UX")
- [ ] Definition of Done checklist is achievable

---

## DoR Checklist

Use this before bringing a story to Sprint Planning:

```
Story: [Title]

☐ User story statement written (As a... I want... so that...)
☐ Acceptance criteria defined (Given-When-Then, minimum 3)
☐ Story points estimated (1, 2, 3, 5, 8, 13, ?)
☐ Priority assigned (High/Medium/Low)
☐ Dependencies identified
☐ Technical questions answered by PO
☐ Mockups/wireframes attached (if applicable)
☐ Testable and unambiguous
☐ Ready for Sprint Planning ✓
```

---

## Who's Responsible?

- **Product Owner (Nour):** Writes story, defines acceptance criteria, answers questions
- **Team:** Reviews for clarity, asks clarifying questions, estimates
- **Scrum Master (Kenza):** Ensures DoR is met before planning

---

## Common DoR Failures

❌ **"User should be able to login"**
- ✅ Better: "As a user, I want to log in with email/password, so that I can access my account securely"

❌ **Acceptance Criteria:** "Login should work" / "User should see feedback"
- ✅ Better: "Given valid credentials, When user clicks login, Then user is redirected to dashboard" / "Given invalid credentials, When user clicks login, Then error message appears"

❌ **"Estimate later"**
- ✅ Better: Team estimates as part of refinement

❌ **No mockup for complex UI story**
- ✅ Better: Wireframe or screenshot attached showing the interface

---

## When to Reject a Story (Not Ready)

Story should be **sent back to PO** if:
- No clear acceptance criteria
- Criteria are vague or untestable
- Story is too large (can't estimate or complete in 1-2 weeks)
- Dependencies are unclear or blocking
- Team has unanswered questions
- No user story statement

---

## Sprint Planning Impact

Stories that meet DoR → **Faster sprint planning** (team spends less time clarifying)

Stories that don't meet DoR → **Slower planning or deferral** (team can't estimate, or story gets deferred to next sprint for refinement)

---

**Document Owner:** Nour Al-Roub (Product Owner)  
**Last Updated:** November 5, 2025  
**Review Frequency:** End of each sprint (update if process changes)
