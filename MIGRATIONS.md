# Migrations

Breaking changes to the vault structure, one section per MAJOR version.

Every other release (MINOR, PATCH) adds or edits files and needs no
migration: the `brain-sync` skill handles those on its own.

A section here describes moves that touch **your own notes**, which is why
they need your explicit approval. `brain-sync` reads this file, works out
which sections apply to your version, shows you every file it intends to
move, and waits for a yes before moving anything.

Each section states the version it upgrades **to**, what changed, and the
rule for relocating existing notes. Apply sections in order if you are more
than one MAJOR behind.

---

## v3.0.0

**What changed:** Excalidraw drawings moved from a top-level `Excalidraw/`
folder into `8.files/excalidraw/`.

Drawings are attachments, and every other attachment already lives under
`8.files/`. A top-level folder for one file type put drawings on the same
footing as the numbered sections of the vault, which they are not.

**Moving your existing drawings:**

Everything under `Excalidraw/` moves to `8.files/excalidraw/`, keeping any
subfolder structure:

| From | To |
| :-- | :-- |
| `Excalidraw/<drawing>.md` | `8.files/excalidraw/<drawing>.md` |
| `Excalidraw/Scripts/` | `8.files/excalidraw/Scripts/` |
| `Excalidraw/Libraries/` | `8.files/excalidraw/Libraries/` |
| `Excalidraw/CJK Fonts/` | `8.files/excalidraw/CJK Fonts/` |
| `Excalidraw/Template.excalidraw` | `8.files/excalidraw/Template.excalidraw` |

Once it is empty, `Excalidraw/` is removed.

**The plugin settings must move with the files.** The Excalidraw plugin
stores five separate paths, and moving the folder without updating them
leaves the plugin writing new drawings to a folder that no longer exists.
This release updates all five, so if you take the settings update you need
do nothing. If you keep your own settings, change these by hand in Obsidian
under Excalidraw's settings:

| Setting | New value |
| :-- | :-- |
| Excalidraw folder | `8.files/excalidraw` |
| Template file | `8.files/excalidraw/Template.excalidraw` |
| Script folder | `8.files/excalidraw/Scripts` |
| Library folder | `8.files/excalidraw/Libraries` |
| Font assets | `8.files/excalidraw/CJK Fonts` |

**Embedded drawings keep working.** Embeds resolve by note name, not path, so
`![[my drawing]]` still finds the drawing after it moves.

---

## v2.0.0

**What changed:** archiving moved out of the zettelkasten and into a
top-level `10.archive/` folder that mirrors the path a note came from.

Before, everything archived was collected under `7.zettelkasten/archive/`,
grouped by note kind. That lost the note's origin: an archived daily note
and an archived sprint note ended up in the same shape, and there was
nowhere for archived notes from `1.daily` or `2.external` to go at all.

Now an archived note keeps its original path underneath `10.archive/`, with
a year folder at the end:

```
10.archive/<original folder path>/<year>/<note>
```

So `7.zettelkasten/sprint/week-12.md` archives to
`10.archive/7.zettelkasten/sprint/2026/week-12.md`. The year folder is
created on demand by `9.scripts/script.archive.md`.

**Moving your existing notes:**

Every note under `7.zettelkasten/archive/<kind>/` moves to
`10.archive/7.zettelkasten/<kind>/<year>/`, where `<year>` is the year the
note was created, matching what `9.scripts/script.archive.md` does from now
on.

**Read the creation date, not the modified date.** Git does not preserve
timestamps, so in a vault that was cloned or copied every file's modified
date is the day you cloned it. If the dates you find are all identical, or
all land on the day you set the vault up, they are not real: fall back to a
date in the note's frontmatter or filename. When there is no trustworthy
date at all, move the note to `10.archive/7.zettelkasten/<kind>/` with no
year folder rather than inventing one. A wrong year is harder to undo than
a missing one.

**Files this template shipped do not get a year folder.** The two notes in
`7.zettelkasten/archive/script/` (`migrate.kind.sprint.md` and
`script.sprint.migrate.md`) came with the template, and it keeps them at
`10.archive/7.zettelkasten/script/` directly. Move them there. Filing them
under a year would leave your vault a different shape from the template, and
the next sync would then read them as missing and add a second copy at the
correct path.

| From | To |
| :-- | :-- |
| `7.zettelkasten/archive/project/` | `10.archive/7.zettelkasten/project/<year>/` |
| `7.zettelkasten/archive/script/` | `10.archive/7.zettelkasten/script/<year>/` |
| `7.zettelkasten/archive/sprint/` | `10.archive/7.zettelkasten/sprint/<year>/` |
| `7.zettelkasten/archive/story/` | `10.archive/7.zettelkasten/story/<year>/` |

If you added your own subfolders under `7.zettelkasten/archive/`, they follow
the same rule: `7.zettelkasten/archive/<anything>/` becomes
`10.archive/7.zettelkasten/<anything>/<year>/`.

Once the folder is empty, `7.zettelkasten/archive/` is removed.

**Also added in this version:** `7.zettelkasten/blog/`,
`7.zettelkasten/content/`, and `7.zettelkasten/workout/`. These are new empty
folders, so nothing moves into them.

**Move your notes with Obsidian closed**, or use Obsidian's own file
explorer to drag the folders. Moving files on disk while Obsidian is running
can leave its link cache stale. If you move them outside Obsidian, reopen the
vault afterwards and let it reindex.

**Links are preserved** either way: this vault uses wikilinks by note name,
not by path, so moving a note does not break links to it.
