---
name: hoja
description: Use the Hoja plugin's TaskManage workflow — register every user request as a task (deliverables as sub-issues), plan work as tasks with acceptance questions, focus one before acting, and complete it only with real evidence. Also covers the hoja:tasks, hoja:hoja, hoja:trace and hoja:setup prompt commands and the session trace attached to Hoja issues.
---

# Hoja task workflow

The Hoja plugin enforces Trebol's task gate: tools that change things (`write_file`, edits, `run_command` with a
non-read-only command, write-capable MCP tools) are denied until a TaskManage task is focused. Reading is always
allowed, and everything the gate allows still goes through Antigravity's own permission settings.

1. **Create** a task with the `TaskManage` tool (server `hoja`, `op: "create"`), with 1–12 acceptance `questions`
   that define "done".
2. **Focus** it: `op: "update"`, `status: "in_progress"`, `active: true`. The gate opens.
3. **Do the work.**
4. **Complete** it: `op: "update"`, `status: "completed"`, with one `answers[]` entry per question. Every answer
   needs `evidence` that resolves:
   - `path/to/file#L10-L20` or `README.md#Heading`: a real workspace file;
   - `tool:Bash`, `tool:Bash#2`, `tool:Write#1`, `tool:TaskManage#1`: a real tool result from this conversation.
     `run_command` counts as `Bash`, `write_file` as `Write`, `read_file` as `Read`, and an MCP tool by its own name.
   A rejected completion lists exactly what is missing. Fix that input; never repeat it unchanged.

When a tool call is denied with `[TASK ENFORCEMENT - BLOCKED]`, create and focus a task, then retry the call.

## Register work without being asked

The user should never have to remind you to track work or update a task.

- **Every request that changes files or state gets a task before you act**, not only long work. Short follow-ups
  ("commit everything", "update both READMEs", "merge and deploy") are requests too.
- **One request, several deliverables:** one root task plus one child per deliverable (`parentTaskId`, in the same
  batch). With Hoja mode on, the root becomes one issue and each child becomes its sub-issue.
- **A new message outside the active task:** create a new root task for it (or a child, if it serves the same
  goal). Never fold a new request into an unrelated task.
- **`metadata: {"hoja":"local"}`** is for mechanical steps under a parent (run the tests, push). Never put it on a
  user request: that request then never reaches Hoja.
- **Before you end a reply:** complete each finished task with evidence, or `addNote` what is left. If you forget,
  the plugin continues your turn once with a reminder.

With Hoja mode on, each top-level task becomes a Hoja issue (and each child of it a sub-issue), kept current on every update and moved to Done with an
evidence comment when it is completed. That completion also attaches this conversation's trace (turns, model calls
with token counts, every tool call and gate block) to the issue, and the TaskManage result says so.

## Prompt commands

Type these as the whole prompt. The plugin answers them itself, before the model is called:

| Prompt | Does |
| --- | --- |
| `hoja:tasks` | Show the read-only task board |
| `hoja:hoja on` · `hoja:hoja off` · `hoja:hoja status` | Turn Hoja mode on or off for this conversation, or show it |
| `hoja:trace` · `hoja:trace upload HOJA-123` · `hoja:trace path` | This conversation's trace: summary, attach it to an issue now, or the JSON file |
| `hoja:setup` | Check credentials, both Hoja endpoints, the project and the enforcement mode |

If a prompt command comes back to you instead of being answered, the plugin's hooks could not reach its server. Call
the read-only `hoja_setup` tool and show its output.
