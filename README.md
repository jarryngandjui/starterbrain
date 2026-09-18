# Obsidian Vault Template
This repository serves as a template for my Obsidian Vault, which includes a collection of icons, automation, templates, and other resources used for learning, productivity, and  projects.

## Apple shortcuts
- [Sprint Apple Shortcut](https://www.icloud.com/shortcuts/0304a388e6ac420095d33abc4ec34d1c)
- [Inbox Apple Shortcut](https://www.icloud.com/shortcuts/6c088c8761a34b78b82953b09ed9d17e)

## Staying up to date

This template keeps changing. To pull those changes into a vault you already
built from it, run the `brain-sync` skill in Claude Code:

```
/brain-sync
```

It compares your vault against the latest release and offers what is new:
scripts, templates, bases, settings, folder structure, and plugin
recommendations. Your notes are never touched, and anything you customized is
shown to you rather than overwritten. Releases are tagged, and
`.starterbrain-version` records which one your vault is on.

Breaking changes to the folder structure are documented in
[MIGRATIONS.md](MIGRATIONS.md), one section per MAJOR version. `brain-sync`
applies them for you, after showing you every file it wants to move.

**If you copied these files rather than cloning**, make sure you also copied
the hidden `.claude/` directory. That is where the skill lives, and file
managers hide it by default.
