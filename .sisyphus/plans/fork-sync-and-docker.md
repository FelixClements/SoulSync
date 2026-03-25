# Fork Sync & Docker Build Action Plan

## TL;DR

> **Quick Summary**: Set up automatic syncing from upstream Nezreka/SoulSync repo and create GitHub Action to build/push Docker image to GitHub Container Registry.
> 
> **Deliverables**:
> - GitHub Action for automatic upstream sync
> - Updated Docker workflow pushing to GHCR
> - Upstream remote configured
> 
> **Estimated Effort**: Short
> **Parallel Execution**: NO - sequential (git config → sync workflow → docker workflow)
> **Critical Path**: Add upstream remote → Create sync workflow → Update docker workflow

---

## Context

### Original Request
User wants to:
1. Keep this fork (FelixClements/SoulSync) always up to date with main repo (Nezreka/SoulSync)
2. Create GitHub Action to build Docker image and push to GitHub Container Registry (ghcr.io)

### Current State
- Repo has `origin` remote pointing to FelixClements/SoulSync
- No upstream remote configured yet
- Existing `.github/workflows/docker-publish.yml` pushes to Docker Hub (boulderbadgedad/soulsync)
- Dockerfile exists at project root

---

## Work Objectives

### Core Objective
Configure fork to sync with upstream and build Docker images to GitHub Container Registry.

### Concrete Deliverables
1. Add `upstream` remote pointing to Nezreka/SoulSync
2. Create `.github/workflows/sync-with-upstream.yml` - runs on schedule + manual trigger
3. Update `.github/workflows/docker-publish.yml` to also push to GHCR

### Definition of Done
- [ ] Upstream remote exists and is fetchable
- [ ] Sync workflow file exists in `.github/workflows/`
- [ ] Docker workflow pushes to ghcr.io with proper tags
- [ ] Both workflows are valid YAML

### Must Have
- Upstream remote for fetching changes
- Sync workflow with manual + schedule triggers
- Docker workflow pushing to GHCR

### Must NOT Have
- No push to upstream (read-only sync)
- No destructive git operations

---

## Verification Strategy

### Test Decision
- **Infrastructure exists**: NO (GitHub Actions already present)
- **Automated tests**: NO (workflow syntax validation only)
- **Verification**: Manual trigger of workflows after commit

### QA Policy
- Verify remote configuration via `git remote -v`
- Validate YAML syntax
- Test workflow execution (manual trigger after implementation)

---

## Execution Strategy

### Sequential Execution

**Step 1**: Add upstream remote
- Run: `git remote add upstream https://github.com/Nezreka/SoulSync.git`
- Verify with `git remote -v`

**Step 2**: Create sync workflow
- File: `.github/workflows/sync-with-upstream.yml`
- Triggers: workflow_dispatch (manual) + schedule (daily midnight UTC)
- Steps: checkout, add remote, fetch, check for updates, merge, push

**Step 3**: Update Docker workflow
- File: `.github/workflows/docker-publish.yml` (update existing)
- Add GHCR login + ghcr.io image push
- Tags: ghcr.io/felixclements/soulsync:latest

---

## TODOs

- [ ] 1. Add upstream remote

  **What to do**:
  - Run `git remote add upstream https://github.com/Nezreka/SoulSync.git`
  - Verify with `git remote -v`

  **References**:
  - Existing remotes: origin (FelixClements/SoulSync)
  - Upstream target: Nezreka/SoulSync

  **Acceptance Criteria**:
  - [ ] `git remote -v` shows upstream remote
  - [ ] Can fetch from upstream

- [ ] 2. Create sync workflow

  **What to do**:
  - Create `.github/workflows/sync-with-upstream.yml`
  - Trigger: workflow_dispatch + schedule (cron: `0 0 * * *`)
  - Steps: checkout, add upstream remote, fetch, check diff, merge, push

  **References**:
  - Existing workflow: `.github/workflows/docker-publish.yml`
  - GitHub Actions docs: actions/checkout, git merge

  **Acceptance Criteria**:
  - [ ] File exists at `.github/workflows/sync-with-upstream.yml`
  - [ ] Valid YAML syntax
  - [ ] Has workflow_dispatch trigger
  - [ ] Has schedule trigger

- [ ] 3. Update Docker workflow for GHCR

  **What to do**:
  - Update `.github/workflows/docker-publish.yml`
  - Add GHCR login step
  - Add ghcr.io image tags
  - Keep Docker Hub push (optional - user can disable)

  **References**:
  - Current workflow: `.github/workflows/docker-publish.yml`
  - GHCR format: ghcr.io/OWNER/IMAGE_NAME

  **Acceptance Criteria**:
  - [ ] workflow-login-action for GHCR
  - [ ] Tags include ghcr.io/felixclements/soulsync
  - [ ] Valid YAML syntax

---

## Commit Strategy

- **1**: `feat: add upstream sync and GHCR docker workflow`
  - Files: `.github/workflows/sync-with-upstream.yml`, `.github/workflows/docker-publish.yml`
  - Pre-commit: none (YAML lint if available)

---

## Success Criteria

### Verification Commands
```bash
git remote -v  # Shows origin + upstream
ls -la .github/workflows/  # Shows both workflow files
```

### Final Checklist
- [x] Upstream remote configured
- [x] Sync workflow exists and is valid
- [x] Docker workflow updated for GHCR

---

## Completion Notes

### Done
1. **Upstream remote**: Already existed - points to Nezreka/SoulSync
2. **Sync workflow**: Created `.github/workflows/sync-with-upstream.yml`
   - Runs daily at midnight UTC (schedule)
   - Can be triggered manually (workflow_dispatch)
   - Fetches upstream/main and merges if new commits exist
3. **GHCR Docker workflow**: Created `.github/workflows/docker-ghcr.yml`
   - Manual trigger with version_tag input
   - Builds multi-arch (amd64, arm64)
   - Pushes to ghcr.io/${{ github.repository_owner }}/soulsync
   - Uses GITHUB_TOKEN for authentication

### Usage
- **Sync**: Go to Actions → "Sync with Upstream" → Run workflow
- **Docker**: Go to Actions → "Build and Push Docker Image" → Run with version tag
