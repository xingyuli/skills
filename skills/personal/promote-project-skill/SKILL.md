---
name: promote-project-skill
description: >-
  Promote a project-local skill (one validated in daily work, not yet
  managed by this repo) into this shared skills repo, or port one in from
  an external repo. Use when the user says "add <skill> from <repo/project>
  into this repo", or decides a project-local skill under .agents/skills or
  .claude/skills is ready to share.
disable-model-invocation: true
---

# Promote Project Skill

Take a skill that already works somewhere else, a project's `.agents/skills/` or `.claude/skills/`, or an external repo, and move it into this shared repo so it's versioned and reusable from here.

## 1. Discover

Read the source `SKILL.md` in full, plus every file it references: companion reference docs, `references/`, scripts. Note every CLI, MCP namespace, or other skill it leans on; those become its own `metadata.requires` in step 5.

## 2. Choose the bucket

| Situation | Bucket |
|---|---|
| Generic, works for anyone who installs the plugin, daily code work | `engineering/` (promoted) |
| Generic, works for anyone who installs the plugin, daily non-code workflow | `productivity/` (promoted) |
| Depends on internal-only or company-specific tooling (a private CLI, an MCP server not everyone has, another skill that isn't public), and you want it reused across your own projects | `personal/` (not promoted, no docs page, still linked by `scripts/link-skills.sh`) |
| Generic and works standalone, but you rarely reach for it | `misc/` (not promoted, no docs page, excluded from `scripts/link-skills.sh`) |
| Working feature you want public feedback on before it graduates | `in-progress/` (public, excluded from the plugin) |

A promoted bucket (`engineering/`, `productivity/`) commits the skill to the public plugin: every installer gets it, so it needs to work without any of *your* private setup. `misc/` and `personal/` both stay off the plugin, but split on a different axis: `misc/` is about how often you reach for a skill that works fine for anyone, `personal/` is about a skill that structurally can't work for anyone without your internal tooling, however often you use it. When in doubt, ask the user rather than guessing; the choice decides how much wiring follows in step 6.

## 3. Choose the name

Prefer an action name (verb-first) over a noun. If the source name already reads as an action, keep it. Otherwise propose two or three renames and let the user pick; don't invent a backward-compat alias unless something already published from this repo depends on the old name.

## 4. Copy and adapt

Copy the source files verbatim into `skills/<bucket>/<name>/`, then adapt:

- Frontmatter `name:` matches the new directory name.
- Add `agents/openai.yaml` (`interface.display_name`, `interface.short_description`), mirroring a sibling skill in the same bucket.
- Decide user-invoked vs model-invoked per [invocation.md](../../../.agents/invocation.md); keep the source's trigger-phrasing style if it already fits the target mode.
- Rewrite every cross-skill reference per invocation.md's convention: `Call the Skill tool with "<name>"` for a model-invoked dependency, or an instruction to the human ("tell the user to run `<name>`") when the dependency is user-invoked. This holds whether the dependency lives in this repo or is expected to be installed externally: naming the skill is what fires it (or, for a user-invoked one, what the human needs to type), regardless of which repo shipped it.

## 5. Make prerequisites self-contained

Record every CLI, MCP namespace, or other-skill dependency **on the skill itself**, never in a separate repo doc: a doc reference breaks the moment this skill is copied standalone into another project (exactly how `lark-doc`/`lark-shared` get consumed there today, one skill folder, not the whole source repo).

- Add `metadata.requires` to the frontmatter, following `lark-doc`'s own shape: `bins` (CLI binaries), `skills` (other skills it calls), and, where relevant, `mcp` (MCP namespaces).
- Add a short `## Prerequisites` section in the body: for each requirement, where it comes from (install source, repo URL) and what auth it needs.

Then call the Skill tool with "verify-skill-setup" to check, install, and live-verify every one of those prerequisites before calling the port done. Don't hand-roll that check here; it's the one skill's whole job, and duplicating it drifts.

**Done when:** the skill's own frontmatter and `## Prerequisites` section are the only place its dependencies are recorded, and `verify-skill-setup` has run clean against them.

## 6. Wire pointers

- **Promoted bucket** (`engineering/`, `productivity/`): add a linked entry to the top-level `README.md`, add the skill's path to `.claude-plugin/plugin.json`'s `skills` array, and create `docs/<bucket>/<name>.md` per [writing-docs.md](../../../.agents/writing-docs.md).
- **Non-promoted bucket** (`misc/`, `personal/`, `in-progress/`): add a one-line entry to `skills/<bucket>/README.md` only; no top-level README, no `plugin.json`, no docs page. In practice, existing `misc/` and `personal/` skills aren't mentioned in `ask-matt` either, so a skill landing there doesn't need that update.
- **Promoted bucket**: re-read [`ask-matt`'s `SKILL.md`](../../engineering/ask-matt/SKILL.md) and update its map so it doesn't route around (or into) a skill it doesn't mention.

## 7. Read-through check

Read the finished `SKILL.md` end to end as if it were new. Confirm: every "Done when" is checkable, no dependency instruction calls the Skill tool on something user-invoked, and no relative link points at a doc that won't travel if this skill is ever copied out standalone.

## 8. Re-link

Run `scripts/link-skills.sh` if the chosen bucket is one it covers (everything except `deprecated/` and `misc/`), so local harness skill directories pick up the new skill.

## Done when

- The skill's directory, its bucket `README.md` entry, and, if promoted, the top-level `README.md` / `plugin.json` / docs page all agree.
- The skill's own frontmatter and `## Prerequisites` section fully describe its dependencies, no separate doc required.
- `verify-skill-setup` has checked, installed, and live-verified those dependencies at least once.
- `ask-matt` mentions the skill if a human can reach it directly.
