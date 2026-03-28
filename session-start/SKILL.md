---
name: session-start
description: Use at the beginning of any session to orient — checks active branches across repos, reads all recent handoffs, queries Jira across all projects, checks GitHub PRs, and prevents stale-context errors
---

# Session Start

## Overview

Full orientation at session start. Prevents "loaded wrong plan," "wrong branch," "stale ticket," and "missed overnight activity" errors by checking actual state across all surfaces before doing anything.

## When to Use

- Start of any new session
- Resuming after a break
- User asks "where are we?" or "what's the status?"

## Process

### 1. Git state

```bash
# Fetch and prune stale remote-tracking branches
git fetch --prune

# Current branch and recent commits (since last handoff, not arbitrary 5)
git branch --show-current
git log --oneline --since="yesterday"

# Any uncommitted work?
git status

# Stale local branches tracking deleted remotes (merged PRs that weren't cleaned up)
git branch -vv | grep ': gone]'
```

If the most recent handoff mentions work in other repos, check those too:

```bash
git -C /path/to/other/repo fetch --prune
git -C /path/to/other/repo log --oneline --since="yesterday"
```

### 2. Read recent handoffs

```bash
# ALL handoffs from today and yesterday — not just the most recent
ls -t docs/HANDOFF-$(date +%Y-%m-%d)-session-*.md docs/HANDOFF-$(date -v-1d +%Y-%m-%d)-session-*.md 2>/dev/null
```

Read ALL of them, not just the last one. Some users do up to 8 sessions per day. Each handoff has:
- Completed items — verify they're actually done, don't re-list
- In-progress items — verify branch/PR still exists
- TODOs with `@done` tags — respect them, never re-list as pending
- Harvest sections — scan for unharvested bonbons, blog material, reusable patterns
- Reminders for next session — these are instructions, follow them

### 3. Check active work items (all projects)

If you work across multiple Jira projects, query all of them:

```bash
# Primary project
JIRA_PROJECT=PROJ jira ls -q "assignee = currentUser() AND status = 'In Progress' ORDER BY updated DESC"

# Secondary projects (add as many as needed)
JIRA_PROJECT=PROJ2 jira ls -q "project = PROJ2 AND assignee = currentUser() AND status = 'In Progress' ORDER BY updated DESC"

JIRA_PROJECT=PROJ3 jira ls -q "project = PROJ3 AND assignee = currentUser() AND status = 'In Progress' ORDER BY updated DESC"

# Recently updated tickets across ALL projects (last 24h) — catches status changes, new comments, new tickets
JIRA_PROJECT=PROJ jira ls -q "updated >= -1d ORDER BY updated DESC"
```

### 4. Check PRs across platforms

GitHub:
```bash
# Open PRs across your org's repos
gh pr list --repo your-org/your-repo --state open
gh pr list --repo your-org/your-api --state open
gh pr list --repo your-org/your-frontend --state open

# Recent workflow failures
gh run list --repo your-org/your-repo --limit 5 --json conclusion,workflowName --jq '.[] | select(.conclusion == "failure")'
```

If you use multiple Git hosting platforms (Bitbucket, GitLab, etc.), check those too:
```bash
# Use the appropriate CLI or API for each platform
# Check all repos with active development for open PRs
```

### 5. Check GHA workflow hygiene

If you own the CI/CD surface and other team members create their own workflows, detect changes and quality issues:

```bash
# New or modified workflow files in the last 7 days
git log --since="7 days ago" --name-only --diff-filter=ACMR -- '.github/workflows/*.yml' | sort -u

# For repos not cloned locally, compare HEAD against a week-old commit
# gh api "repos/OWNER/REPO/compare/OLDSHA...HEAD" --jq '.files[] | select(.filename | startswith(".github/workflows/"))'

# Recent workflow runs — look for sloppy patterns:
# - Same workflow run 5+ times in a row (trial-and-error without fixing the root cause)
# - Runs without meaningful titles (no context for future debugging)
# - Failed runs that were never investigated
gh run list --repo your-org/your-repo --limit 30 --json workflowName,conclusion,displayTitle,createdAt
```

Flag for review:
- New workflow files you haven't reviewed
- Workflow runs with empty or default titles (no `displayTitle` or just the commit message)
- The same workflow failing 5+ times in a row (someone is spamming reruns instead of debugging)
- Any workflow that bypasses plan-before-apply or skips tests

Also watch for infra drift — resources created via kubectl or console that aren't in terraform, and untagged infrastructure. This includes live patches applied during firefighting that never made it into the IaC module. If the last handoff mentions live patches, check whether they've been baked into terraform yet.

This is protagonist support for the shared CI/CD space — making sure the infrastructure stays legible and accountable.

### 6. Check pipeline state

For each in-progress work item, trace through the pipeline:

| Stage | How to check |
|-------|-------------|
| Ticket created | Jira query (step 3) |
| Branch exists | `git branch -a \| grep TICKET` |
| PR open | `gh pr list --search TICKET` or platform API |
| PR merged | `gh pr list --state merged --search TICKET` |
| Needs approval | Check last Jira comment / Slack thread |
| Deployed | Check workflow runs |

### 7. Summarize

Present a brief orientation:

```
Branch: feat/PROJ-481-copilotkit-sidecar
Stale branches: fix/PROJ-482-apiserver-stability (remote deleted)

Handoffs read: 2 from today, 1 from yesterday
  - Session 01: ETL seed job completed, service decoupled
  - Session 02: design investigation, synthesis posted

Active tickets (across projects):
  - PROJ-483 (EKS node auto-repair) — To Do
  - PROJ-488 (demo response epic) — To Do
  - PROJ2-434 (FinOps spike) — In Progress

Open PRs: none
Failed workflows: none

Unharvested from handoffs:
  - bonbon: "the format is not the signal"
  - blog idea: template skills vs behavior skills
```

Keep it dense. This is orientation, not a report.

### 8. Load project-specific skills and briefing

If the session involves project-specific work:

1. Load any Jira-related skills — if your org uses Jira heavily, every session will touch tickets.
2. Load any briefing/morning-paper skill — pulls Slack channels, cloud support cases, meeting transcripts, and semantic search into a full situational briefing.

The morning paper builds on the orientation from steps 1-7. Session-start provides the facts. Morning paper provides the narrative.

## Rules

- Do NOT start implementing anything until orientation is complete
- Read ALL recent handoffs, not just the most recent — some users do 8 sessions/day
- Respect `@done` tags — if a TODO has `@done`, it's done. Period.
- Check multiple repos if the handoff mentions cross-repo work
- Check all Jira projects, not just the main one
- `git fetch --prune` before reporting branch state
- Report stale local branches that track deleted remotes
- Check all Git hosting platforms for PR state

## Red Flags

- Starting implementation without checking current branch
- Ignoring older handoffs from the same day
- Assuming ticket states without checking — statuses change between sessions
- Working on a ticket that's already Done
- Reporting on a branch that was merged and deleted upstream
- Only checking one Jira project
- Missing PR activity on secondary platforms
- Re-listing items marked `@done` in handoff TODOs
