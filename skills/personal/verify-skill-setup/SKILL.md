---
name: verify-skill-setup
description: >-
  Check, install, and live-verify a skill's prerequisites: a CLI, an MCP
  namespace, another skill, or auth it declares in its own frontmatter. Use
  right after copying a skill (for example brief-feishu-ticket or
  requirement-intake) into a new project, or whenever the user asks to set
  up, install, or verify a skill's dependencies.
---

# Verify Skill Setup

Given a skill, named, or the one just copied into this project, make its declared prerequisites real, then prove it works against real data. Read only the skill's own `metadata.requires` frontmatter and `## Prerequisites` section: never a separate doc in some other repo. That's what makes this work even when the skill was copied in standalone, with nothing else from its source repo alongside it.

## 1. Bring the skill in, if it isn't already

Prefer `npx skills@latest add xingyuli/skills --skill <name>` over a manual copy: it fetches straight from GitHub, links it into the right agent directories, and leaves `npx skills update` as the way to pick up later changes. Project-scoped by default; add `-g` only for a skill you want available everywhere (that's how this repo's own `promote-project-skill` and `verify-skill-setup` are meant to be installed, since they're generic, not tied to one project). Fall back to a manual project-local copy, matching this repo's own `lark-doc` precedent, only when that installer isn't reachable or the skill isn't published yet.

**Done when:** the skill's `SKILL.md` (and any companion files it ships, like `lanhu-sources.md`) are in this project's own skill directory.

## 2. Read what it needs

Open the target skill's `SKILL.md`. Collect:

- `metadata.requires.bins`: CLI binaries.
- `metadata.requires.skills`: other skills it calls.
- `metadata.requires.mcp`: MCP namespaces, where declared.
- The `## Prerequisites` section's prose: install source for each bin, which skill's repo to pull from, what kind of auth.

If a skill declares none of this, there's nothing to verify; say so and stop.

## 3. Check each one

| Prerequisite | Check | If missing |
|---|---|---|
| CLI | `which <bin>`, `<bin> --version` | Install it yourself (`npm i -g <package>`, etc.), if the `## Prerequisites` section gives a plain install command |
| Skill | Does it show up as an available skill this session, or under `~/.claude/skills` / `~/.agents/skills` / `<project>/.agents/skills`? | If it's a plain copy or symlink you can make yourself from a known source, do it |
| MCP namespace | Is it in the harness's dynamic tool catalog? | Needs registering in the harness's MCP config, usually with a token or URL from a dashboard |
| Auth | Run the CLI's own status check (`meegle auth status --format json`, etc.) | Expired or missing: re-authenticate |

## 4. Close the gaps

- **Agent-doable** (installing a package, writing a config file, symlinking a directory you already have the source for): do it yourself.
- **Human-only, multi-stage, with a value or two to persist** (registering a new MCP server: get a token from the provider's site, write it into the harness's MCP config, confirm it connects): call the Skill tool with "wizard". That's its exact remit: "provisioning infrastructure, setting up credentials... walking an unfamiliar third-party dashboard."
- **Human-only, single value, nothing to persist** (a device-code login the CLI already stores in its own config): walk the human through that one prompt in conversation. Generating a wizard script for one URL and one code is more ceremony than the step needs.

**Done when:** every prerequisite from step 2 is either verified working on this machine, or has a concrete next action (a wizard script, or a named conversational step) that gets it there.

## 5. Verify live

A prerequisite check isn't enough for a skill whose whole job is fetching real external data: it catches a missing binary, not a dead integration. Ask the user for one concrete piece of real input per capability the skill exercises, the exact kind of thing its own `description` says it triggers on (a Feishu Project issue URL for `brief-feishu-ticket`; a Lanhu page URL and a Feishu wiki URL for `requirement-intake`), and actually run the skill's first fetch or decode step against it. Report exactly what came back.

**Done when:** the skill has been run, at least once, against real data the user supplied, and it worked, or the failure is now a named, fixable gap rather than an untested assumption.
