---
name: requirement-intake
description: >-
  Intake a new requirement from Lanhu MCP prototypes and Feishu wiki.
  Use when the user pastes lanhuapp.com, names a 蓝湖 group, asks to use
  the lanhu MCP, or gives a Feishu wiki PRD before grilling or implementing.
metadata:
  requires:
    bins: ["lark-cli"]
    skills: ["lark-doc", "lark-shared"]
    mcp: ["lanhu"]
---

# Requirement intake

Shape understanding from **Lanhu** screens and **Feishu wiki** before design or code. Interactive collect first; fetch second; grill third.

## Prerequisites

- **`lanhu` MCP namespace**: served by the community project [dsphper/lanhu-mcp](https://github.com/dsphper/lanhu-mcp). Its own docs default to Docker; running it directly on the host works too, and uses less memory. Register it in your harness's MCP config either way.
- **`lark-doc` and `lark-shared` skills**: both ship, along with the rest of the `lark-*` family, from [larksuite/cli](https://github.com/larksuite/cli). Installing `lark-cli` drops the *entire* family into your harness's global skill directory; a project-local install of just these two keeps that global directory lean.

New to this project, or unsure any of this is in place? Call the Skill tool with "verify-skill-setup" first.

## 1. Collect

Ask in one round for every missing input. Stop after the questions. Finding the Lanhu project is not a fact hunt: the URL is the user's to give.

Need:

- Lanhu URL (`tid` + `pid`; `docId` if Axure)
- Feishu wiki / Docx URL(s)
- Group or page names to download
- Local save directory

**Done when:** each of those four is in this conversation, or the user marked one N/A.

## 2. Fetch Lanhu

Use the **lanhu MCP** (`namespace: lanhu`). Inspect tool schemas, then follow [`lanhu-sources.md`](lanhu-sources.md). List, then download only the named groups. Unique filenames (index + id), then rename after you Read each image.

**Done when:** every named group has uniquely named screens in the save directory.

## 3. Fetch wiki

Call the Skill tool with "lark-doc", then `docs +fetch`: outline first, in-scope sections next. `--as bot` if user OAuth is missing.

**Done when:** outline plus every in-scope section is in context.

## 4. Grill

`grill-with-docs` is user-invoked: tell the user to run it once artifacts are on disk (it chains grilling and domain-modeling). Artifacts already on disk are the brief; it should not re-fetch them. Do not implement until the frontier is empty and the user confirms shared understanding.

**Done when:** the user has started that grilling round, or explicitly declined it.
