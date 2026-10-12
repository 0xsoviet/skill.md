---
name: git-control
title: "Git control"
description: "Skill for AI agents to perform Git workflows (branching, commits, PRs, hotfixes, releases) across assigned repositories. Repository-specific rules override this skill."
version: "1.0"
scope: "global"
license: "MIT"
tags: ["git","workflow","conventional-commits","ai-agent","skill"]
capabilities:
  - branch-management
  - commit-convention
  - pull-request
  - release
  - hotfix
  - safety
requirements:
  - git
  - repo-access: write-via-pr
---

<!--
  This file is importer-friendly for AI agents (e.g., OpenMausBot, Claude Code).
  Requirements from importer:
  - MUST have YAML frontmatter (--- block) at the VERY TOP of the file.
  - Provide `name`, `description`, and `license` (other fields optional but helpful).

  Behavior:
  - This skill should be treated as a global default for any repository the AI agent is assigned to.
  - If the repository contains its own SKILL.md, CONTRIBUTING, or other repo-level conventions, those take precedence.
-->

# Kỹ năng: Git control cho AI agents

Mục tiêu: cung cấp một bản quy ước máy-đọc được và dễ áp dụng để AI agent (ví dụ Claude Code) thực hiện các thao tác Git an toàn và nhất quán trên bất kỳ repository nào được giao.

Lưu ý nhanh:
- Luôn đọc README, CONTRIBUTING, và bất kỳ file SKILL.md cấp repo nào trước khi hành động — repo-level rules override.
- Không push trực tiếp lên branch được bảo vệ; mọi thay đổi vào branch bảo vệ phải qua Pull Request.
- Tuyệt đối không in hoặc ghi log secrets, tokens, hay environment variables.

## Machine checklist (for agents)
1. Pre-checks
   - Read repository files: README, CONTRIBUTING, SKILL.md, CODE_OF_CONDUCT, and any workflow protections.
   - Detect default branch name (main/master) and whether dev branch exists.
   - Detect branch protection rules (if accessible) and required reviewers/CI policies.
2. Workspace rules
   - Create a dedicated feature/fix branch for each task.
   - Use conventional commits (recommendation) for commit messages.
   - Run repository tests and linters locally or in the configured CI environment before pushing.
3. Push & PR
   - Push branch to remote and open a PR into the repository's chosen base branch (dev or default).
   - PR must include: summary, testing steps, test results, related issues (Closes #N).
   - Do not merge until required reviews and CI checks pass (unless repository rules allow otherwise).
4. Release & hotfix
   - Follow repository's release process; tag releases with semantic versioning.
   - For hotfixes: branch from main, PR to main, then backport to dev/default as required.
5. Safety
   - Never create commits that add secrets (.env, API keys) — if found, abort and notify owner.
   - Do not attempt to bypass branch protections or set permission modes programmatically.

## Recommended Git conventions (importer-friendly)
- Branch naming: lower-case, hyphen-separated.
  - feature/<task>, fix/<task>, refactor/<task>, docs/<task>, test/<task>, chore/<task>, hotfix/<issue>
- Commit messages: follow Conventional Commits
  - Example: `feat(auth): add login API`
- PR workflow: create PR into dev or default branch; use Squash for feature PRs (unless repo specifies otherwise).

## Minimal automated checks (for agents to perform)
- Verify there is a SKILL.md or CONTRIBUTING; if none, proceed with this skill as default.
- Run `git fetch` and merge base branch before pushing.
- Run test suite (e.g. `npm test`, `pytest`, or repo-specific command detected in package.json or pyproject)` — if tests fail, do not open PR and report failure.

## OpenMausBot / Claude Code compatibility notes
- The importer requires YAML frontmatter at the very top of the SKILL.md file — this file includes that block.
- If your engine still complains, ensure the importer reads the repository path you're targeting and that the file name is exactly `SKILL.md` (case-sensitive).
- If the agent expects specific frontmatter keys (e.g., `name`, `description`, `license`), those are present.

## Example quickstart for human readers
1. Ensure the repository grants you PR access (or invite a maintainer).
2. Create branch: `git switch -c feature/short-description`
3. Make focused commits following Conventional Commits.
4. Push and open PR: include description and test steps.
5. Wait for CI and reviewers; merge per repo policy.
