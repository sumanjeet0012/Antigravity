# Creating a Symlink for Antigravity Skills

This guide explains how to keep your Antigravity skills under Git while
allowing Antigravity to continue loading them from the default location.

## Why use a symlink?

Antigravity loads skills from:

``` text
~/.gemini/config/skills/
```

Instead of storing your skills there directly, you can store them inside
your Git repository and create a symbolic link (symlink). This gives
you:

-   ✅ Version control with Git
-   ✅ No duplicate copies of your skills
-   ✅ Antigravity continues working without any configuration changes

------------------------------------------------------------------------

## Example Directory Structure

Assume your Antigravity repository is located at:

``` text
~/code/antigravity/
```

Create a `skills` directory inside it:

``` text
~/code/antigravity/
└── skills/
    ├── protocol-design/
    ├── libp2p-review/
    ├── github-actions-debug/
    └── ...
```

------------------------------------------------------------------------

## Step 1 -- Move Existing Skills

Move your existing skills into the repository:

``` bash
mkdir -p ~/code/antigravity/skills
mv ~/.gemini/config/skills/* ~/code/antigravity/skills/
```

If the directory is empty, you can simply start creating skills inside:

``` text
~/code/antigravity/skills/
```

------------------------------------------------------------------------

## Step 2 -- Remove the Original Directory

Once you've confirmed everything has been moved:

``` bash
rm -rf ~/.gemini/config/skills
```

------------------------------------------------------------------------

## Step 3 -- Create the Symlink

Create a symbolic link pointing to the repository directory:

``` bash
ln -s ~/code/antigravity/skills ~/.gemini/config/skills
```

Verify it:

``` bash
ls -l ~/.gemini/config
```

Expected output:

``` text
skills -> /Users/<your-user>/code/antigravity/skills
```

------------------------------------------------------------------------

## Step 4 -- Verify

Create a new skill:

``` text
~/code/antigravity/skills/test-skill/SKILL.md
```

Run Antigravity and confirm it detects the new skill.

------------------------------------------------------------------------

## Daily Workflow

Add a new skill:

``` bash
cd ~/code/antigravity

mkdir -p skills/my-new-skill
touch skills/my-new-skill/SKILL.md

git add skills
git commit -m "Add my-new-skill"
git push
```

Antigravity will automatically use the new skill because the symlink
points to the Git-tracked directory.

------------------------------------------------------------------------

## Notes

-   A symlink is only a pointer to another directory.
-   Editing files through either path edits the same files.
-   Do **not** create both a real directory and a symlink at
    `~/.gemini/config/skills`.

## Summary

    Git Repository
    ┌───────────────────────────────┐
    │ antigravity/                  │
    │ └── skills/                   │
    │     ├── protocol-design/      │
    │     ├── libp2p-review/        │
    │     └── ...                   │
    └───────────────┬───────────────┘
                    │
                    │ symlink
                    ▼
    ~/.gemini/config/skills

This setup lets Git track all of your skills while Antigravity continues
using the default skills location transparently.
