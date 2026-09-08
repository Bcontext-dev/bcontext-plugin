---
name: bcontext
description: How to work a Bcontext workspace as an agent — the 19-tool surface (nodes/nodes_write, subtasks/subtasks_write, tags/tags_write, views/views_write, links_write, ingest/ingest_write, skills/skills_run, admin/admin_write, ask_rag, list_changes, list_workspaces, ping), H2 blocks, subtasks inside a task, the if_updated_at rule, typed dependencies, cited retrieval, working alongside other agents, and what to write down. Use whenever reading from or writing to a Bcontext workspace (bcontext.dev or self-hosted) via the mcp__bcontext__* tools.
---

# Working with Bcontext

Bcontext is the shared memory of a team: people and other agents read and
write the same workspace while you work, and nothing you do not write down
exists for them. Everything below was learned by agents doing a week of real
work through this surface, and it is ordered by how much it saved them.

## Start here, in this order

1. **Read the server's `instructions`** from `initialize`. They are short,
   accurate for the build you are talking to, and they are the only manual.
2. **`list_workspaces`** — which workspaces this token reaches. A workspace
   token is bound to one; a user token picks one per call with
   `X-Bcontext-Workspace` (or `?workspace=`).
3. **`list_changes({ since })`** — what happened since you last looked. Keep
   the newest `ts` as your next `since`. This is how you catch up; do not
   re-read the tree.
4. **`nodes({ op: "list", kind: "task", unblocked: true })`** — what can be
   started now. Tasks that look independent usually are not; the unblocked set
   is the truth, and finishing one moves the frontier for everyone.
5. **`tags({})`** before you assign any `tag_ids`. There is no get-or-create.
6. **`ask_rag({ query })`** for "what do we already know about X" — read
   `scope.keyword_fallback` in the answer: `true` means no embeddings were
   used and ranking is coarse, so cross-check with `nodes(op=list)` before
   concluding something is not there.

## The surface

Reads never mutate; `*_write` tools do. Pick the operation with `op`:

| Read | Write | What it is |
|---|---|---|
| `nodes` (get · list · search) | `nodes_write` (create · update · delete · set_node_tags · toggle_checklist_item · attach_file) | the knowledge itself |
| `subtasks` (list · get) | `subtasks_write` (add · update · delete · toggle · reorder) | the steps inside one task, edited by id |
| `tags` (list) | `tags_write` (create · rename · merge · delete · link · unlink) | the taxonomy — needs the `tags:*` verbs, which an admin grants |
| `views` (list · get · resolve · query) | `views_write` (create · duplicate · update · delete) | saved query lenses |
| — | `links_write` (link · unlink) | typed edges between nodes |
| `ingest` | `ingest_write` | connector candidates awaiting review |
| `skills` | `skills_run` | prompt templates the server can execute |
| `admin` (access · preview · held_invitations) | `admin_write` (grant · revoke · invite · release_invite) | who can reach what — needs the `admin` verb |
| `ask_rag`, `list_changes`, `list_workspaces`, `ping` | | |

`tools/list` is authoritative and per-principal: a tool you cannot use is not
shown. Tools named `<provider>__<capability>__<tool>` run in an external
provider through Bcontext's gateway, re-authorised on every call; treat them
as open-world actions and never retry one unless the error says to.

**Learn the schemas from the errors.** Every write tool is a discriminated
union on `op`; calling `nodes_write({ op: "create" })` with nothing else
returns the required fields and the valid enum values (`kind`, `priority`,
`status`). Four cheap failed calls up front beat guessing. Two shapes to
remember: `create` takes fields flat; `update` nests them under `patch`.

## Write so that others can read

- **Titles are names, not codes.** `Worker propio con pg-boss`, not
  `F01 — Worker propio con pg-boss`; `Realtime propio: fuera Supabase
  Realtime`, not `F05 — …`. No numbering or prefix (`NEXO-03`, `P4-05`,
  `1.`), no ` — ` separators. A code means something to the session that
  wrote it and nothing to the next reader, and it goes stale the first time
  the plan is reordered; the id is the reference. Same for subtask titles.
  A write with a coded title still lands but answers with `title_hint`.
- **Structure every body with `##` headings.** Each H2 is an addressable block
  (`/n/<id>#<slug>`) other agents can cite and extend. A body without H2s is a
  wall nobody can point into.
- **Write the reasoning into the node**, not just the outcome. The most useful
  nodes a teammate will read are the ones with `## Why` or `## Status`
  explaining a judgment call, who made it and whether it was escalated. A
  decision you took alone is a `decision` node with `## Question / Decision /
  Consequences / Status`; a reviewer will extend that node rather than open a
  disconnected one.
- **Prose mentions are not links.** "See the decision below" is invisible to
  the graph. `links_write({ op: "link", from_id, to_id, relation:
  "references" })` right after creating related nodes — one call per edge —
  is what makes `nodes(op=get)` show the relationship and what makes
  `unblocked` correct.
- **Dependencies are edges, never text.** `blocked_by` from a task to what it
  waits on. The target must be something that can reach `done` — a task or
  bug, never a decision or doc (the server refuses, because it could never
  unblock). A task blocked on a decision `references` it and says so in
  `## Status`.
- **An unplanned blocker** is a new task describing the missing prerequisite,
  `blocked_task --blocked_by--> new_task`, and one line in the blocked task's
  body pointing at it. That keeps everyone's `unblocked` list honest.
- **A question for a specific teammate**: there is no mention or assignment
  primitive yet. The convention that worked: an open `decision` node with a
  `## Question` heading, the options named, `## Status: Open`, tagged like
  everything else. Say who you need an answer from. Nobody is notified; they
  will see it on their next `list_changes`.

## Write safely — the rule that bit a real agent

Patching `content_md` **requires** `if_updated_at`: read the node, take its
`updated_at`, pass it back. The server refuses a body rewrite without it and
tells you the current value. A **409** means someone wrote in between —
re-fetch, **merge** your change onto the current body (the other write may
have added a whole section you do not have), retry with the new timestamp.

Why it is mandatory: a builder agent forgot it once and silently replaced a
reviewer's notes and a planner's sign-off that had landed on the same node
minutes earlier. The only trace was an unfamiliar `previous_block_ids` in the
response; the text came back from `list_changes({ node_id })`'s
before-snapshots. Metadata patches (`title`, `status`, `tag_ids`) touch no
prose and need no timestamp. To flip one checkbox use
`nodes_write({ op: "toggle_checklist_item" })` — one call, no
read-modify-write.

## Archive what is finished or abandoned

`status: "archived"` is cleanup, not deletion: the task leaves every view,
board, graph and `nodes({ op: "list" })` — but stays readable by id
(`nodes({ op: "get" })`), linked, cited and searchable, and any other status
brings it back. Tasks are created for a piece of work and then just exist;
archive them once the work is done or dropped so live views stay about live
work. `nodes({ op: "list", status: "archived" })` (or `include_archived:
true`) shows the shelf; an archived blocker no longer blocks.

## A task is one node; its steps are subtasks

Do not split one piece of work into `phase 1 / phase 2 / phase 3` task nodes.
The graph is for meaningful units — a task someone can pick up, block on, or
finish — and five half-tasks give every reader five reads and every planner
five statuses to reconcile. The steps to finish a task are its **subtasks**:
they live on the task (`data.subtasks`), never in the graph, and one
`nodes({ op: "get" })` returns the task with all of them, in full.

- **Create them with the task**, in the same call:
  `nodes_write({ op: "create", kind: "task", title, content_md, subtasks: [
  { title, summary, content_md }, … ] })`. Each subtask has a `title`, a
  one-line `summary` (what the card shows — you write it, with the context
  you have now; the server never generates one) and a full `content_md`
  (what opening the card shows). Order is the array order.
- **Edit one step, not the task.** `subtasks_write({ op, node_id, … })`:
  `add` (title, summary, content_md, optional `position`), `update` (`id`,
  `patch`), `delete` (`id`), `toggle` (`id`, optional `done` — omit to flip),
  `reorder` (`ids` = the full order, or `id` + `position`). Every op costs
  one small call and never touches the task's body or `updated_at` guard
  logic on your side: omit `if_updated_at` and a lost race is retried for
  you; pass it and a mismatch is a 409.
- **Read cheaply.** `subtasks({ op: "list", node_id })` returns cards
  (id, title, summary, done) and progress; `subtasks({ op: "get", node_id,
  id })` returns one with its `content_md`. Ids are short (`s3kd9a2`) so
  that addressing a step costs fewer tokens than describing it.
- **Mark progress as you go.** Toggling a subtask is how humans on the board
  and other agents see where the task is; the task's own `status` moves to
  `done` when the last step does — that part is still yours to set.
- Subtasks are for `task` and `bug` nodes. A doc's checklist stays a
  `- [ ]` list in its body (`toggle_checklist_item`).

## Views are for the team

`views_write({ op: "create" })` defaults to `visibility: "workspace"`. The view
`query` is a versioned object (`schema_version: 1`, `kinds`, `statuses`,
`tags_any`/`tags_all`, `dates`, `source`, `actor`…) — the same contract
`ask_rag`'s `scope` and `views(op=query)` use, and **not** the same field
names as `nodes(op=list)` (`kinds` vs `kind`). Today the view query cannot
express `unblocked`; a view of "what can start now" is an approximation
(`statuses: ["todo"]`) — say so in the view's `description`, and point at
`nodes(op=list, unblocked: true)` for the real answer.

## Retrieval scope, explicitly

`nodes(op=search)` and `ask_rag` default to the whole workspace. When the
question is local, pass `scope` as an object:

```json
{ "schema_version": 1, "type": "view", "view_id": "<id>" }
{ "schema_version": 1, "type": "tags", "tags_any": ["billing", "vat"] }
{ "schema_version": 1, "type": "selection", "node_ids": ["<id>", "<id>"] }
{ "schema_version": 1, "type": "neighborhood", "node_id": "<id>", "depth": 1 }
```

Scopes filter before retrieval; `broad: true` lets graph expansion reach
outside the set and marks those hits `outside_scope`. `auto` always keeps a
visible workspace lane — never describe it as a permission boundary. Every
hit carries a paste-ready `ref` (`/n/<id>#<block>`); follow it with
`nodes(op=get)` before acting on a claim. New content is searchable after
~15 s (async embedding); direct reads see it at once.

## Who did what

`list_changes` gives millisecond timestamps, before/after snapshots, and an
`actor_label` — today a hash of the principal, not a name, so two agents
starting together are told apart by the shape of what they wrote until a
second id appears. Write your role and intent into `## Status` sections; it
is what the next reader will use.

## What to leave out

Raw JSON you paged through to reconstruct a timeline, and observations about
the tool itself ("I made a concurrency mistake"), do not belong in the
workspace: they are process, not the team's knowledge. Put them in your
report, not in a node.

## Misc contracts

- Deletion is irreversible and cascades to descendants; inspect
  `nodes(op=get)` first.
- `nodes_write({ op: "attach_file" })` for screenshots (png/jpeg/webp/gif
  ≤ 4 MB); embed the returned URL as markdown.
- Errors are written to be read: they name the offending key, list the valid
  options, or carry the recovery recipe. Read them before retrying.
