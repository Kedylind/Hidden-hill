# Definition of Done (DoD)

## Overview

A user story is **"Done"** when it meets ALL criteria below. Stories that don't meet DoD should **not** be marked as complete and should **not** be considered for release.

This ensures consistent quality, maintainability, and reliability of shipped features.

## Criteria for Done Stories

Before a story can be marked "Done" and moved to the Done column:

### 1. Code Written & Committed to Feature Branch
- [ ] Feature branch created: `feature/issue-#-short-name` or `fix/issue-#-short-name`
- [ ] Code implements all acceptance criteria
- [ ] Code is committed with descriptive commit messages
- [ ] No TODO comments or incomplete code
- [ ] Code is clean and follows project conventions

**Branch Naming Example:**
- `feature/34-user-authentication`
- `fix/42-login-timeout`

---

### 2. At Least 1 Peer Code Review Completed & Approved
- [ ] Pull Request opened with clear description
- [ ] At least **1 team member** has reviewed the code
- [ ] Reviewer has approved the PR (or "Approved" comment)
- [ ] All feedback has been addressed
- [ ] No "Changes Requested" status remaining
- [ ] Reviewer checked:
  - [ ] Code follows project style guide
  - [ ] Code is readable and maintainable
  - [ ] No security issues or hardcoded secrets
  - [ ] Performance is reasonable

**Code Review SLA:** 24 hours target

**Reviewer Checklist:**
```
- Is the code clear and well-structured?
- Does it follow our naming conventions?
- Are there any security concerns?
- Could this be optimized?
- Are there any edge cases missing?
- Is the solution reasonable for the problem?
```

---

### 3. All Tests Written & Passing
- [ ] Unit tests written for new functions/methods
- [ ] Integration tests written for feature interactions
- [ ] All tests pass locally and in CI/CD pipeline
- [ ] Test coverage target met ([80%/90%?])
- [ ] No test failures or skip directives

**Testing Standards:**
- **Unit Tests:** Test individual functions/methods in isolation
- **Integration Tests:** Test feature workflows end-to-end
- **Manual Testing:** Edge cases and UI interactions

**Example:**
```javascript
describe('User Login', () => {
  test('should log in with valid credentials', () => {
    // Test implementation
  });
  
  test('should reject invalid credentials', () => {
    // Test implementation
  });
});
```

---

### 4. Manual Testing Completed in Staging
- [ ] Feature deployed to staging environment
- [ ] Tester has verified all acceptance criteria manually
- [ ] Happy path works (normal use case)
- [ ] Edge cases tested (empty input, invalid data, etc.)
- [ ] Error handling verified (appropriate error messages)
- [ ] UI/UX verified (responsive, accessible, polished)
- [ ] No regression issues found

**Staging Environment:** [Link to staging URL]

---

### 5. No Hardcoded Values or Sensitive Data Exposed
- [ ] No passwords, API keys, or secrets in code
- [ ] Configuration values moved to environment variables
- [ ] Database credentials not stored in repository
- [ ] Sensitive data logging removed
- [ ] Code is safe to commit to public repository

**Security Checklist:**
```
✓ No .env files committed
✓ No console.log() with sensitive data
✓ No hardcoded URLs (use config)
✓ No comments with passwords or tokens
✓ Secrets stored in GitHub Secrets / CI variables
```

---

### 6. README/Documentation Updated (if applicable)
- [ ] README updated with new features/changes
- [ ] API documentation updated (if applicable)
- [ ] Code comments added for complex logic
- [ ] Setup/installation instructions updated (if needed)
- [ ] Changelog entry added

**Documentation Examples:**
- API changes → Update API docs
- New environment variable → Update README
- Database schema change → Update schema docs
- Complex algorithm → Add inline comments

---

### 7. Pull Request Merged to `main` Branch
- [ ] All review approvals obtained
- [ ] All status checks passing (CI, linting, tests)
- [ ] No merge conflicts
- [ ] Merge commit message is descriptive
- [ ] Feature branch deleted after merge

**Merge Checklist:**
```
✓ All approvals received
✓ CI pipeline passed
✓ No conflicts with main
✓ Commit message is clear
✓ Branch deleted
```

---

### 8. Deployed to Staging (or Ready for Release)
- [ ] Code deployed to staging environment
- [ ] Deployment was successful (no errors)
- [ ] Feature is accessible and functional on staging
- [ ] Monitoring/logging set up (if applicable)
- [ ] Performance metrics acceptable

**Deployment Verification:**
- [ ] Application starts without errors
- [ ] Feature is live and accessible
- [ ] Database migrations ran successfully
- [ ] No 500 errors in logs
- [ ] Response times are acceptable

---

### 9. Product Owner Verified Acceptance Criteria Met
- [ ] PO has reviewed the feature on staging
- [ ] All acceptance criteria confirmed as met
- [ ] Feature delivers the intended user value
- [ ] PO has signed off (or approved in GitHub)
- [ ] Ready for production release

**PO Sign-Off:**
- PO reviews feature on staging
- PO verifies each acceptance criterion
- PO approves and marks as "Done"

---

## DoD Checklist

Use this before marking a story as "Done":

```
Story: [Title]

Code Quality:
☐ Code written and committed
☐ At least 1 code review approved
☐ All tests written and passing
☐ Manual testing completed in staging

Security & Maintenance:
☐ No hardcoded secrets
☐ README/docs updated
☐ Code follows conventions

Deployment & Verification:
☐ PR merged to main
☐ Deployed to staging successfully
☐ Product Owner verified acceptance criteria

Status: ✓ DONE
```

---

## Who's Responsible?

- **Developer:** Write code, tests, commit to feature branch
- **Reviewers:** Code review, approve/request changes
- **Tester (Team):** Manual testing on staging
- **Product Owner (Nour):** Final verification and sign-off
- **DevOps/Scrum Master:** Deployment and environment setup

---

## Done vs. Shipped

| Status | Definition | Next Step |
|--------|-----------|-----------|
| **Done** | Meets all DoD criteria, deployed to staging, PO approved | Ready for release to production |
| **Shipped** | Released to production, working for real users | Monitor for issues, gather feedback |

---

## Common DoD Failures

❌ **"Code written but no tests"**
- ✅ Better: Write unit & integration tests before review

❌ **"Tests pass locally but not in CI"**
- ✅ Better: Debug CI environment, ensure all dependencies installed

❌ **"Manual testing skipped - tests should be enough"**
- ✅ Better: Always do manual testing on staging (catch UI issues, edge cases)

❌ **"PO didn't review but looks good to me"**
- ✅ Better: Wait for explicit PO approval before marking Done

❌ **"Hardcoded API URL for staging"**
- ✅ Better: Use environment variables for all configuration

---

## Sprint Metrics Using DoD

Tracking DoD compliance helps measure:
- **Quality:** % of stories meeting DoD on first attempt
- **Velocity:** % of committed stories actually marked Done
- **Technical Debt:** # of stories deferred due to failed DoD checks

---

## When to Reject a Story (Not Done)

Story should stay in "In Progress" or go back to "In Review" if:
- Tests are failing
- Code review not approved
- Manual testing found bugs
- Secrets/sensitive data in code
- Documentation not updated
- PO acceptance criteria not met
- Deployment failed

---

## Evolution of DoD

DoD is **not static**. We can update it if:
- A quality issue crops up repeatedly (add DoD criterion)
- A step becomes obsolete (remove from DoD)
- Tooling changes (update DoD process)
- Team feedback suggests improvement

**Discuss DoD updates in sprint retrospectives.**

---

## Deployment & Release

Stories marked as **Done** are candidates for production release. Actual production deployment happens on a **separate release schedule** (e.g., end of sprint, weekly, etc.).

**Release Process:** [To be defined]

---

**Document Owner:** Kenza Moussaoui Rahali (Scrum Master)  
**Last Updated:** November 5, 2025  
**Review Frequency:** End of each sprint (update if process changes)
