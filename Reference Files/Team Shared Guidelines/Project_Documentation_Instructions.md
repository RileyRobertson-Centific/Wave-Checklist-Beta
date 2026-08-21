# Project Documentation Standard – Data Collection Quality and Technical Operations

These are custom Claude instructions for how to create and maintain project documentation. Provide the following to Claude as a whole file or copy and paste into a Claude project’s instructions.


## When this standard applies
Use this four-document standard for projects with real staying power: an app, tool, or recurring deliverable that will be worked on across multiple sessions, that someone other than its original owner might eventually need to pick up, or that has real downstream dependencies (other systems, other teams, other files that consume its output).

Don't apply it to quick one-off work — a single report, a one-time script, a single analysis someone asked for and will use once. Producing four documents for a task that took ten minutes is worse than producing none. If you're not sure which category something falls into, err toward not creating the full set yet, and revisit once the project shows signs of an actual second session (someone asks for a change, or opens a new chat about it).

If a project you're working on doesn't have these files yet but seems to qualify, say so and offer to set them up rather than assuming — this is a real structural decision (see the project's own instructions on when to check in before acting), not a cosmetic one.


## The Four Documents

### Overview and Purpose
Each document exists because it has a different reader, and conflating readers is what makes documentation rot. Before writing into any of these, know your audience:

 | Document Name | Who reads it | What it's for |
|---|---|---|
| `README.md` | Anyone (before other documents) | Quick orientation — what this is, what's here, where to go next |
| `Changelog.md` | Anyone curious about project history | What changed, version by version, and why |
| `Dev_Notes.md` | A Claude chat picking this project back up | Full working context so nothing has to be re-explained |
| `Team_Handoff.md` | A human teammate maintaining or inheriting this project | How to make changes and/or keep it running without the original owner |

A reader should never need to read one of these to make sense of another — link between them instead of repeating. In particular, high-level "what is this project" context belongs in `README.md` only; `Dev_Notes.md` and `Team_Handoff.md` should point back to it rather than restating it.


### README.md
The front door. Short — a few minutes to read, not a reference manual. Contains:

- What this project is, in a paragraph or two.
- What's in the project folder — a table of every file/folder and what it's for.
- How the thing actually gets used, in plain terms (if it's a tool: what does a user do with it; if it's a report or dataset: how is it consumed).
- How it fits into anything bigger it connects to — other systems, other teams' workflows, anything downstream that depends on this project's output.
- A "where to go from here" table: given a reader's actual goal ("I need to fix a bug," "I need to hand this off," "I need the full history"), which document should they open next.

Update rarely — only when the big picture itself changes (the project's purpose shifts, a major new system integration appears, the folder structure or key files are reorganized). If you find yourself editing this file often, something has probably drifted into it that belongs in `Dev_Notes.md` or `Team_Handoff.md` instead.


### Changelog.md
A chronological, version-by-version record of what changed and why. Within a version, lead with the changes that matter most. The small stuff can be grouped under a lightweight "Minor changes" heading rather than each getting equal billing.

If, within a single version, a change is made and then reverted to how it was before, remove any mention of the change within `Changelog.md`, but add a note in `Dev_Notes.md` to record what happened and any reasoning that might be relevant for decisions later on.

Similarly: if, within a single version, multiple changes are made to a single item (e.g. text color changed from white, to grey, and then finally to blue) in which the intermediate changes are removed before landing on a final version, `Changelog.md` does not need record of every step. Instead, add a note in `Dev_Notes.md` to record that the testing happened and any reasoning that might be relevant for decisions later on.  

If the number of updates for a single version grows past 10 major items, consider categorizing the major changes according to 2 or 3 high-level project-relevant attributes (e.g. fuctional/visual or back-end/interface) and prompt for confirmation before executing on this.

Adopt (or ask the project's owner to confirm) a version-numbering convention early, and hold to it — something like MAJOR.MINOR.date works well because the date segment makes it easy to tell how fresh a given build is at a glance, but the exact scheme matters less than picking one and staying consistent. Don't bump the version number for every small edit — reserve it for changes significant enough that someone using the output would benefit from knowing which version they're on (a changed file format, a changed behavior, a fix worth flagging). 

Important note: unless directly instructed otherwise, **always** ask before bumping the version number rather than deciding alone — it's a real judgment call, not a mechanical one.


### Dev_Notes.md
Written entirely for continuity between Claude sessions in case context grows too large or chat limits are reached — assume the reader (a future Claude chat) knows nothing about this conversation and has only this file, the code/files themselves, and whatever else is in the project folder. This is the one document that should be kept current at all times, not just at milestones; update it alongside the change itself, not after the fact.

Include:
- A pointer to `README.md` for the big picture, so this file doesn't have to repeat it.
- **A living file/folder inventory** — what everything is, written for someone about to work in the code, not someone just orienting.
- **The current state** — current version, what's shipped vs. still in progress.
- **To-do list** — Anything that must happen before this ships or goes live for real, if that's not yet done.
- **A roadmap of deferred work** — ideas raised but not prioritized, with enough context that revisiting them later doesn't require reconstructing the original reasoning.
- **Key decisions, each with its reasoning, not just its outcome.** This is the highest-value section in the whole document. A future session re-deciding something already settled — and getting a different answer — is the single most expensive failure mode this file exists to prevent. When you record a decision, record *why*, not just *what*, so a later session can tell the difference between "this was a deliberate tradeoff" and "this just hasn't been reconsidered yet."
- **Working conventions** — anything about *how* to collaborate on this specific project that isn't obvious from the files themselves: how the owner likes feedback, whether to preview changes before building them, anything you got corrected on once and shouldn't repeat.


### Team_Handoff.md
Written for a person, explicitly not for a Claude chat — open with a line saying so, and pointing to `Dev_Notes.md` for anyone continuing this project with Claude's help. The goal: if this project were handed to a new teammate with zero other context, this one file should tell them how to keep it running.

Organize by category rather than by strict chronology or file-by-file walkthrough — readers come to a handoff doc with a specific need, not to read cover to cover. A structure that's worked well:

- **Maintaining the application** — the actual how-to: how to make a change (including how to work with Claude directly if the reader isn't technical — be explicit and step-by-step here, don't assume familiarity), what to do if editing something manually instead of through Claude (and to mention the manual edit next time, since Claude has no way to know about it otherwise), and a clear list of the specific recurring things that need upkeep (a reference list, a config file, a set of credentials — whatever they are for this project).
- **Things that must not change without care** — anything downstream depends on staying stable (an exact file format another system consumes, a naming convention an automation expects, an API contract). Be specific about *what breaks* if it changes, not just that it shouldn't — "changing this silently breaks X" is far more useful than "please don't change this."
- **Other things to know** — reference details that don't fit the above (naming/format conventions, versioning, anything situational).
- **Action items** — anything explicitly not done yet that whoever inherits this should know about, even if the current owner expects to finish it themselves first. Include it anyway, as a safety net.


## General Guidelines

### Naming and formatting conventions
- File names: `README.md` in the conventional all-caps form; the other three in title case with underscores between words: `Changelog.md`, `Dev_Notes.md`, `Team_Handoff.md`. pick one convention per project/team and apply it consistently rather than mixing styles across files.
- Use tables for anything that's fundamentally a list of pairs (item, description) — file inventories, column references, field references. Prose reads worse than a table for that shape of information.
- Cross-reference by file name in backticks (`Dev_Notes.md`) so links are consistent and easy to spot.


### Setting this up on a new project
1. Confirm with the project owner that the project qualifies (see "When this standard applies" above) before creating all four files unprompted.
2. Draft `README.md` first — everything else can reference it, and writing it forces clarity on what the project actually is.
3. Start `Dev_Notes.md` and `Changelog.md` together once there's a first real working version to describe.
4. Add `Team_Handoff.md` once the project reaches the point where someone other than you maintaining it becomes a real possibility — don't wait until an actual handoff is imminent, since that's the worst time to be reconstructing context from scratch.


### Picking this up on an existing project
If a project already has some (but not all) of these files, or has similar documents under different names, don't force a rename or restructure unprompted — ask the owner whether they want to adopt this standard, and if so, whether to migrate existing content or start fresh. Read whatever documentation already exists before proposing anything; it likely already encodes decisions worth preserving.


### Diverting from the standard
If instructions are given that would result in changes to these documents that contradict the standards laid out above, provide a clear warning about the implications and request explicit confirmation before proceeding.
