---
name: brief-feishu-ticket
description: >-
  Brief one Feishu Project issue via meegle. Use when the user pastes a
  project.feishu.cn issue/detail URL, or asks to read a 飞书项目 缺陷 or 需求.
metadata:
  requires:
    bins: ["meegle"]
    skills: ["meegle"]
---

# Brief Feishu Ticket

Produce a **brief** of one Feishu Project work item so the rest of the session can work from it. GitLab Issues stay on `glab`.

## Prerequisites

- **`meegle` CLI and `meegle` skill**: both ship from [larksuite/meegle-cli](https://github.com/larksuite/meegle-cli). Install the CLI with `npm i -g @lark-project/meegle`; install the skill by placing (or symlinking) that repo's skill directory into your harness's skill directory (project-local, e.g. `.agents/skills/meegle`, or global).
- **Auth**: device-code login, per Feishu Project host: `meegle auth login --device-code`, then poll per the skill's own auth-guard.

New to this project, or unsure any of this is in place? Call the Skill tool with "verify-skill-setup" first. Otherwise, call the Skill tool with "meegle" whenever a step below needs command shape, the auth-guard, URL kinds, comments, or attachments; this file is only the brief recipe.

## 1. Decode

```bash
meegle url decode --url '<URL>' --format json
```

Save `host`, `simple_name`, `work_item_type`, `work_item_id`. Branch on `url_kind` against meegle's `url-kinds.md` reference (call the Skill tool with "meegle" to reach it): only `workitem_detail` continues here.

**Done when:** `url_kind` is `workitem_detail` and those four fields are saved.

## 2. Auth

Follow meegle's **auth-guard** against the decoded `host`. In this agent shell, start login with `--device-code` (`meegle auth login --help` for `--phase init` / poll).

**Done when:** `meegle auth status --format json` has `authenticated: true` and `host` matching the URL host.

## 3. Harvest

```bash
meegle project search --project-key "$simple_name" --format json
```

Save the returned `project_key`. Then `workitem get` and `comment list` for that key and `work_item_id`, inspecting both commands first. Pull every field page (`fields=["_all"]`; meegle's workitem get reference covers `page_size` via `--params`). People are on `work_item_attribute.role_members` and `current_status_operator`.

Description and comment image URLs are opaque `file_url`s: `attachment +download` each one, then Read the local file. Image-only descriptions are the ticket: the image's claims are body text.

Capture:

- title, status, priority, assignee, reporter
- modules / platforms
- description text
- every evidence image's claim (expected vs actual, which field, which screen)
- every comment, in order

Finding facts is your job. Do not ask the user to paste the ticket.

**Done when:** the brief can stand alone: a reader who never opened Feishu can name the bug, the evidence, and any comment that changes scope. Missing one evidence image is not done.

## 4. Brief

Lead with the ticket URL and title. Then: symptom, expected, evidence, scope (platforms, official vs custom, which screen). Quote UI field names as they appear. Keep product language; file paths wait until a later skill asks for a plan.

This skill **reads**. Implement, grill, or comment back on the ticket only when the user asked for that in the same turn.

**Done when:** you have sent the brief. Whatever comes next (implementation, a design grill, TDD) starts from it, not from a second fetch.
