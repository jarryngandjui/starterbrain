---
name: brain-sync
description: Pull updates from the upstream starterbrain template into this vault, covering scripts, templates, bases, folder design, settings, and the plugin list, without overwriting your own notes or customizations. Use for "sync starterbrain", "update my vault from the template", "is my vault out of date", "get the latest starterbrain", "what changed upstream", "brain-sync".
user-invocable: true
disable-model-invocation: false
allowed-tools: Read, Glob, Grep, Bash, Edit, Write, AskUserQuestion
---

# brain-sync (starterbrain → this vault)

Bring this vault up to date with the starterbrain template it was built from.

This vault is yours. Upstream only ever *offers* changes: it never wins by
default. Your notes, and the edits you made to templates and settings, are
the reason the vault is useful, so nothing is written without you approving
it first.

The skill works the same whether you cloned starterbrain, forked it, or
copied the files by hand, because it compares file trees rather than git
history. It does not need a shared ancestry and does not touch your remotes.

## Before touching anything

**Confirm there is a way back.** Check whether the vault is a git
repository (`git rev-parse --git-dir`).

- **It is a repo**: confirm the working tree is clean, then work on a branch
  cut from the current one. Anything you dislike is one `git checkout` away.
- **It is not a repo** (common when the files were copied): stop. There is no
  undo, and this skill moves notes during a migration. Offer to run
  `git init`, commit everything as a restore point, and continue from there.
  If the user declines, do not proceed to any step that writes.

## Finding upstream

Shallow-clone the template into a temporary directory and compare against
that:

```
git clone --depth 1 https://github.com/jarryngandjui/starterbrain.git <tmp>
```

Use a scratch directory, never a folder inside the vault, or Obsidian will
index the clone as notes. Remove it when finished.

## Knowing which version this vault is on

Read `.starterbrain-version` at the vault root. It holds a single tag, such
as `v2.0.0`, and this skill writes it after every successful sync.

**No such file** means either a first sync or a hand-copied vault. Do not
guess a version from the file contents. Treat the baseline as unknown, say so,
and fall back to comparing against the latest tag: every difference becomes a
candidate, and the user decides what is a missing update versus a
customization they want to keep.

The baseline matters because it is what separates *you changed this* from
*this is simply old*, which is the distinction the whole conflict rule below
rests on. Without it, every difference looks like a conflict.

## What is in scope

Offered as updates:

- `9.scripts/`, `6.templates/`, `5.bases/`
- `.obsidian/*.json` settings, `.obsidian/themes/`, `.obsidian/icons/`
- The **folder skeleton**, meaning directories, not the notes inside them
- The plugin list, as a recommendation (see below)

**Never touched, except by an approved migration**: everything in
`1.daily`, `2.external`, `3.indexes`, `4.tasks`, `7.zettelkasten`, `8.files`,
`10.archive`, `Excalidraw`. These hold your notes.

**Never touched at all**: any plugin's `data.json`, `workspace.json`,
`workspace-mobile.json`, `.obsidian/plugins/*/` contents, and
`.starterbrain-version` except as the final step.

## Deciding what to take

Compare each in-scope file against upstream, and sort it into one of three
states. The state determines how much ceremony it gets, because asking about
eighty files individually guarantees the user stops reading.

1. **Missing here.** Upstream has it, this vault does not. A genuinely new
   script, template, or base. Group these into a bucket and take one yes for
   the bucket.
2. **Unchanged since the baseline.** The file matches what upstream shipped
   at `.starterbrain-version`, so the user never edited it and the update is
   clean. Group these into a bucket too, and name what changes in each.
3. **Diverged.** The file differs from the baseline, so the user edited it
   *and* upstream moved. This is a conflict. Handle these one at a time: show
   their version against upstream's, and offer three choices: keep mine, take
   upstream's, or merge by hand. Never fold a conflict into a bucket.

When the baseline is unknown, state 2 cannot be distinguished from state 3.
Say so plainly and treat every existing file as a conflict, because silently
overwriting a customization is the one outcome this skill exists to prevent.

Present buckets with `AskUserQuestion` (multiSelect). Never apply everything
because it "looks clean".

## Migrations

A MAJOR version bump means the folder structure changed and existing notes are
now in the wrong place. `MIGRATIONS.md` at the upstream root describes each
one, with a section per breaking version.

Read it, work out which sections sit between this vault's version and the
target, and apply them **in order**. Skipping to the newest section leaves a
vault that matches no released structure.

Migrations are the one place this skill moves notes, so:

- **Dry-run first, always.** List every file that would move, showing source
  and destination, and a count. Then wait for an explicit yes. A migration
  applied halfway is worse than one never started.
- Move, never copy and delete, so the note's history survives. Use `git mv`
  when the vault is a repo.
- **Never overwrite a note at the destination.** If something is already
  there, stop and report it rather than merging blind.
- Create destination folders as needed, and remove source folders only once
  they are empty.
- If the move is interrupted, report exactly which files moved and which did
  not. Do not retry from the top.

Tell the user to close Obsidian first, or to move the files through Obsidian's
own explorer, since moving notes on disk while it runs leaves its cache stale.

## Plugins

Sync the plugin *list*, not plugin state.

Compare `community-plugins.json` and report which plugins upstream added or
dropped, with one line on what each is for, so the user can install or remove
them through Obsidian's own settings. Installing through Obsidian is what
fetches the correct version for their platform and registers it properly.

Leave every `data.json` alone. That is where their toolbars, QuickAdd macros,
and task settings live, and those are theirs.

Note that `community-plugins.json` lists only **enabled** plugins. A plugin
directory with no entry is installed but disabled, which is normal and is not
a difference worth reporting.

## Finishing

- Verify every `.json` this skill wrote still parses (`jq empty`). A malformed
  settings file stops the vault from opening, which is worse than being out of
  date.
- Write the new tag to `.starterbrain-version`.
- Commit, with a message naming the version moved to and what was taken. Keep
  the migration in its own commit, separate from file updates, so it can be
  reverted on its own.
- Delete the temporary clone.
- Tell the user to reopen the vault so Obsidian reindexes, and say plainly
  what was skipped and what still needs doing by hand.
