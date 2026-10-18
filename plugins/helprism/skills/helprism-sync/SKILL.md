---
name: helprism-sync
description: Beta. Use after you finish a code change, or before you open a pull request, in the session that wrote the change. Decides whether the Helprism help center is now wrong or incomplete for its readers and, if so, drafts the exact article rewrites and sends them to Helprism for human review through the Helprism MCP server (report_change). Many changes need nothing.
license: MIT
compatibility: Needs the Helprism MCP server, and git. Connect the hosted server at https://api.helprism.com/mcp and sign in with your browser when asked (no API key), or run the local @helprism/mcp package after npx -y @helprism/mcp@latest login. An API key is for CI only.
metadata:
  version: agent-door-v3 (2026-09-23)
  procedureVersion: v3-4544eddb
  status: beta
---

<!-- Source: logs/agent-door-v3-2026-09-23/procedure-v3.md, sha256 4544eddb1260ba1b9210ee7e7f0e090feac33ee5f6e8e5ae0cbf0d2fcada579a (copy: packages/skill/source/procedure-v3.md). Every judgement rule below is that frozen, measured text. Only the I/O is adapted: the help center is read through MCP instead of the eval corpus, and the result goes to report_change instead of a predictions file. Check with: node packages/skill/scripts/check-skill-vs-v3.mjs -->

# helprism-sync: keep the help center true when you ship

You are the coding agent that just made this change. You know what it does better than anyone. Your job now is narrow: tell the team whether the **public help center** needs to change because of it, and if it does, write the change.

Two failures cost the team. A false alarm lands in a human's review inbox, and an inbox full of "maybe check this" gets muted. A missed update leaves readers following instructions that no longer match the product. **Saying "no change" is the right answer for internal, visual and marketing work, and for articles that already match the change in every detail.** When the change does alter what an article tells readers, propose the update, in every article that says it.

## Inputs

- **The change:** the diff and the commit message(s) or PR description. You may read any file in the repository to understand what the change does for users (for example the component that renders a label, or the route that enforces a limit).
- **Read the code as of the change.** The change is the source of truth for product facts. Read repository files at the change's own revision (for a commit, `git show <sha>:<path>`), not from a newer or older checkout. If a later version of the code says something different, ignore it: you are judging this change.
- **The help center:** every article as it reads **today**, from the Helprism MCP server. `list_articles` with status `published` lists what readers see, with each article's id and slug; `get_article` returns an article's full Markdown. There is no search tool: to search article bodies, read the articles with `get_article`. Category ids come from `list_categories`.

Judge the articles as they read now against the product as the change leaves it.

## Step 1. Impact note (one line, written first)

Before you open any article, write exactly one line from what you know as the author:

- `Help center impact: NONE`, or
- `Help center impact: <everything that changed for a user of the product, in plain words>`

"A user" means a customer using the product: someone signing up, writing articles, configuring their help center, calling the public API or MCP server, or a visitor reading the public help center. Developers of this codebase are not users. Write what a user would notice, not what the code does.

If the change does several user-visible things, list all of them. Each one is judged in step 3.

## Step 2. Classify, and stop early only for the clear cases

Pick exactly one class and give a one or two sentence reason.

| Class | Use it when | Default outcome |
|---|---|---|
| `new-behaviour` | Users can now do or see something they could not before. | Continue to step 3. |
| `changed-behaviour` | Something users already could do now works differently: different rules, limits, defaults, steps, permissions, plan gating, visible text, API request or response shape, error codes. | Continue to step 3. |
| `ui-copy-or-layout` | A label, button text, menu name, tab, settings location, visible message or navigation path that users follow was renamed, moved, added or removed. The underlying behaviour is the same. | Continue to step 3. |
| `bugfix-restores-documented-behaviour` | Something was broken and now works the way users would expect, or the way an article already says it works. | **Stop. `changes: []`.** |
| `internal` | Refactors, tests, CI, build, dependencies, types, logging, performance with no visible behaviour change, backend plumbing, admin-only or staff-only tools, developer docs, eval or tooling, repo housekeeping. | **Stop. `changes: []`.** |
| `marketing-site` | The landing page, pricing page presentation, blog, SEO pages, legal pages, or other marketing surfaces. The product itself did not change. | **Stop. `changes: []`.** |
| `visual-only` | Styling, colours, spacing, animation, icons, responsive tweaks, loading states. No label, step, location, option or message that a reader follows changed. | **Stop. `changes: []`.** |

Rules for classifying:

- If a commit mixes several things, classify by the most user-significant part, but in step 3 judge **every** user-visible part you listed in the impact note, not only that one. A commit that is mostly internal but also renames one visible label still continues for that label.
- A bug fix that changes behaviour away from what an article says is `changed-behaviour`. A bug fix that makes the product match the article, or match what any user would assume, is `bugfix-restores-documented-behaviour`: the article was already right.
- A change to plan limits, prices or plan gating is `changed-behaviour` even if it lives in pricing code. A change only to how the marketing page presents plans is `marketing-site`.
- If your impact note is `NONE`, the class must be one of the four stopping classes, and `changes` is `[]`.

## Step 3. Find every affected article, then test each one

### 3a. Find the candidates

- Search titles and bodies for the feature, its settings location, its labels and its visible text.
- **Sweep for old values.** If the change renames, moves or removes a label, button, menu item, navigation step, URL, path, limit, price or plan tier, search **every** article for the old value. Every article that tells the reader to use the old value is a candidate, including getting-started guides and overviews. If the old value **keeps working** (a fallback, an alias, a redirect), an article that only mentions it in passing is not wrong: keep only the articles where the reader goes to find or use that value.
- **Where the reader acts.** If the change adds a rule to an action (who may do it, when it is unavailable, a limit, a confirmation), the article that walks the reader through that action is a candidate, as well as any overview that lists the rule.

There is no quota. Most changes touch one article; a rename that appears in four articles touches four. Each candidate passes or fails the test below on its own.

**Primary and secondary articles.** A *primary* article is about the changed feature, or walks through the action the change constrains. Every other candidate is *secondary*. A secondary article changes only under test 1 below (a sentence that is now wrong and that a reader would act on). Tests 2 and 3 apply to primary articles only. If you are unsure whether a secondary article needs the change, leave it out.

### 3b. The test: would a reader be misled, fail, or miss something?

For each candidate, read the **whole article** and ask:

> Would a reader following this article today, on the product as this change leaves it, be **misled**, **fail**, or **miss something the change added to the task this article teaches**?

It is **yes** when any of these holds. For each, you must be able to name the sentence or the section:

1. **A sentence is now wrong.** Quote it and say what it should say. A settings path that moved, a label or button name that differs from the UI, visible text that differs, a limit or plan tier that changed, a step that no longer exists or is now in a different order, an API field that was renamed, an old URL that is no longer the address users are given.
2. **A described behaviour is now incomplete in a way that changes what the reader does or expects.** The article describes what happens in a step, and the change alters or adds to what happens there: an automatic behaviour, a new default, a new time the step takes, a different input it accepts.
3. **A new constraint, state or message in the task the article teaches is missing.** Who can do it, when it is unavailable, a new required step or confirmation, a new status or error the reader will see and what to do about it, whether an old path or link still works.
4. **A new capability has no home.** Covered in the `create` rules below.

It is **no** when:

- **The article already covers the change, and every concrete detail matches.** Do a detail audit before you say this. List the concrete facts the article states about the changed feature: UI labels, button and section names, visible text, plan tiers, limits and numbers, URLs and paths, the order of steps, who can do it. Check each one against the code as of the change. If all of them match, the article needs nothing, even if the feature is brand new in this commit. If a **name** differs (a label, button or section name, visible text, a plan tier, a number, a path), that sentence is now wrong: it is a yes under 1, however many names differ, and the update fixes each one.
- **The article documents a different version of the feature.** Sometimes this change *introduces* a feature and an article about it already exists, written from a plan, a spec or another version of the product. Wrong names alone never mean this: fix them. It means the article describes **behaviour or controls the change does not have**: a different overall flow (a different way to sign in or authenticate, a different sequence of steps), places the feature appears that it does not, controls or screens that do not exist at all, or outcomes that do not happen. If you find two or more such behaviour mismatches, this change did not make the article wrong: the two describe different versions. Do not rewrite it. Propose `verify` on it (at most 2 per commit) and name the biggest mismatch in `reason`. This applies only when the change introduces the feature. When the change modifies a feature that users already had, every mismatch it causes is a yes.
- **Mentions of other features are not mismatches.** If the article mentions another feature, integration or plan that does not exist yet at this change, or that this change does not touch, leave that sentence alone and do not count it.
- The change adds an **optional extra** that the article's task never touches and that nobody needs to complete that task (a new optional attribute, a cosmetic option, an advanced setting the article does not discuss). "It would be nice to mention" is no.
- The article is only **related** to the feature. Being about the same area is not enough; one of 1 to 3 must hold.

Before proposing `create`, search the help center for an existing article that covers the topic. If one exists, propose an `update` to it instead. Propose `create` only when a real new capability has no home anywhere and a user would need instructions to use it.

## Step 4. Write complete rewrites of the affected sections only

Each change is one of:

- **`update`**: the full revised article. `draft` is the complete article body in Markdown, without the frontmatter: every unchanged section copied verbatim, only the affected sections rewritten. `changedSections` lists the headings of the sections you changed (use `"(intro)"` for text before the first heading).
- **`create`**: `articleSlug: null`, `newArticle: { title, categoryId }`, and `draft` is the full article body in Markdown. `categoryId` is an existing category id, or `new:<name>` if no category fits.
- **`retire`**: the change removes the feature an article is about. `draft: null`.
- **`verify`**: the article is squarely about the feature you changed, and a specific claim in it might now be affected, but you could not settle it from the code. `draft: null`. A `verify` never reaches the review inbox; it goes to a background freshness list. At most **2** per commit. Also use `verify` for an article that documents a different version of a feature this change introduces (step 3b). Never use `verify` as a hedge for an article that is merely related, never for a single claim you could check in the code, and never on a commit you classified in a stopping class.

Draft rules:

- **Scope: touch only the sections the change affects.** Sections the change does not affect are copied word for word. Do not restructure, re-title or "improve" them.
- **Completeness inside those sections.** A reviewer should not have to add a fact the change introduced. Within each section you touch, cover everything in the change that a reader of that section would notice:
  - the corrected labels, names, visible text, numbers, tiers and paths, using the exact wording in the UI;
  - new fields or information the screen now shows in the part the article walks through;
  - who can do it, and when it is unavailable (roles, statuses, plans);
  - defaults and automatic behaviour, and how long it takes if the reader waits for it;
  - new messages, statuses or errors the reader will see, and what to do about each one;
  - whether the old way, old path or old link still works;
  - edge behaviour the change defines (items that are skipped, partial failures, what is kept or lost).
- **Only what the change touches.** Do not add facts about other behaviour, even if they are true. Each fact must be confirmed in the change or in the code as of the change. Do not invent numbers, plan tiers or labels.
- Write for the help center's readers, in the article's existing voice, headings and formatting.
- Never put code, diffs, file paths from the repository, internal function names, environment variables or secrets in a draft. Drafts are help-article prose only.
- No draft means no `update` or `create`. If you cannot write the rewrite, you do not have an update.

## Output

Send your judgement to Helprism with **one `report_change` call** per change, even when `changes` is `[]`: a no-change report is logged and never reaches a human. Map your result onto the call like this:

| Your result | `report_change` field |
|---|---|
| the commit (`sha`, subject, link) or the PR (number, title, link) | `ref`: `{ "kind": "commit", "id": "<sha>", "label": "<subject>", "url": "<link>" }` or `{ "kind": "pr", ... }` |
| the commit subject or PR title | `title` |
| the paths the change touched | `filesTouched`: paths only, never file contents or diffs |
| `impactNote` | `impactNote` |
| `changeClass` | `changeClass` |
| (fixed) | `procedureVersion`: `"v3-4544eddb"` |
| `changes` | `suggestions`, one entry per change, below |

Each change becomes one suggestion:

- `update`: `{ "kind": "update", "articleSlug": "<slug>", "draftMarkdown": "<draft>", "changedSections": [...], "rationale": "<the reason for this article>" }` (use `articleId` instead of `articleSlug` if you have it).
- `create`: `{ "kind": "create", "title": "<newArticle.title>", "categoryId": "<newArticle.categoryId>", "draftMarkdown": "<draft>", "changedSections": [...], "rationale": "<why a reader needs it>" }`.
- `verify`: `{ "kind": "verify", "articleSlug": "<slug>", "rationale": "<the claim to check, or the biggest mismatch>" }`.
- `retire`: the beta does not accept it. Do not send it; tell the user which article should be retired and why.

If `report_change` returns a validation error, fix the shape of the call (a missing field, a slug that does not exist) and send it again. Do not change your judgement to get past it.

Then tell the user, in a few lines:

- the impact note and the class, with your one or two sentence reason;
- one line per suggestion (kind, article, what changed), and any `retire` you could not send;
- the `reviewUrl` from the response. Nothing is published until someone accepts it there. `list_suggestions` and `get_suggestion` show the same suggestions as diffs in the terminal.

For a stopping class, one line is enough: the class, the reason, and that no help-center change is needed.

`changeClass` is one of `new-behaviour`, `changed-behaviour`, `ui-copy-or-layout`, `bugfix-restores-documented-behaviour`, `internal`, `marketing-site`, `visual-only`. Each article appears at most once per commit.

## Checklist before you answer

1. Did I write the impact note, listing every user-visible part, before reading articles?
2. Did I read the code as of the change, not a later version?
3. If the class is a stopping class, is `changes` empty?
4. For a renamed or moved value, did I search every article for the old value?
5. For every article I left alone that covers the feature, did I check each concrete detail it states against the change? Did I fix every wrong name, and use `verify` only when the article describes two or more behaviours or controls this new feature does not have?
6. For every `update`, can I name the wrong sentence, the incomplete behaviour or the missing constraint, state or message? Is every secondary article I changed wrong under test 1, not just incomplete?
7. Does every draft cover all of the change inside the sections it touches, leave every other section word for word, and contain no code, paths or secrets?
8. At most 2 `verify`, none that are just "related" and none I could have settled in the code?
9. Did I send exactly one `report_change` (paths only, no code) and show the user the `reviewUrl`?
