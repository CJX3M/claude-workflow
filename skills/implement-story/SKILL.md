---
name: implement-story
description: End-to-end workflow to implement an Azure DevOps story across BackEnd, FrontEnd and Azure Functions. Plans from the story, waits for human approval, then branches, codes, tests, reviews, pushes and opens PRs using dedicated agents. Usage - /implement-story <storyNumber>
---

Run the full story workflow for story **$ARGUMENTS** by delegating to the agents below. You (the main conversation) orchestrate; subagents cannot call each other. Keep the user informed in Spanish with short status lines.

Agents: `story-planner`, `git-manager`, `backend-developer`, `frontend-developer`, `test-engineer`, `code-reviewer`.

If `$ARGUMENTS` is empty, ask for the story number.

## Phase 1 — Plan (stop for approval)
1. Launch `story-planner` with the story number. It writes `~/.claude/story-plans/<id>-plan.md`.
2. Show the user the plan's summary, open questions and the path. Ask them to review it and approve or request changes. Iterate with `story-planner` (SendMessage) until approved. **Do not proceed without an explicit approval.** Open questions marked as blocking must be answered first.

## Phase 2 — Branches
1. Launch `git-manager` to sync the base branch and create, per affected repo, the code branch (and the `-db` branch if the plan has `[DB]` steps), following the naming standard. If it needs the standard confirmed, relay the question to the user.

## Phase 3 — Implementation, step by step
Follow the plan's step order. Pick the developer by repo: BackEnd and Azure Functions → `backend-developer`; FrontEnd → `frontend-developer`. Give each the plan path, the step number(s), the repo path and the branch.
1. **DB first, but never blocking.** Run `[DB]` steps on the `-db` branch. The user cannot run scripts by hand: the deployment pipeline runs them only after the `-db` PR is approved and merged into the base branch, which can take time. So once the DB steps are committed, have `git-manager` push the `-db` branch and open its PR right away (pre-authorized by the user for `-db` branches), tell the user the PR link so it gets approved, and CONTINUE with the code steps without waiting. Tell the user which behavior can only be verified after the script is deployed.
2. For each code step: developer implements and commits → `test-engineer` adds/adjusts tests and commits, verifying compile and full run → if tests reveal a production issue, send it back to the developer. Never move on with a failing build or tests.
3. Steps in different repos with no dependency may run in parallel; steps touching the same repo run sequentially.

## Phase 4 — Review
After all steps of a repo, launch `code-reviewer` with the plan path and repo/branch. If it returns CHANGES REQUESTED, route each finding to the right agent (developer or test-engineer), have them fix with new commits, and re-review. Repeat until APPROVE. Escalate to the user if the same blocker survives two rounds, or if a fix would go beyond the plan.

## Phase 5 — Delivery (ask first)
Show the user a summary per repo (branches, commits, test results, review verdict). On their go-ahead, launch `git-manager` to push the branches and open the PRs (the `-db` PR should already be open from Phase 3; flag it as to-merge-first since the pipeline runs the script on merge). Report the PR links.

## Rules
- Never skip the approval gate in Phase 1, never push or open PRs without the user's go-ahead (except `-db` branches, see Phase 3), never let a DB change land in a code branch.
- Anything outside the plan goes back to the user, not to an agent to "just do".
- If a tool is missing (Azure CLI, PAT, node, etc.), tell the user exactly what to install or provide, then continue.
