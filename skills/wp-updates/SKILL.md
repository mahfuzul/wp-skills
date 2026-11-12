---
name: wp-updates
description: Run the routine WordPress core, plugin and theme update workflow for any git-tracked WordPress project using WP-CLI. Creates a dated wp_updates_{month}_{year} branch from the base branch, commits core, plugin and (with approval) theme updates separately, merges into the stage/staging branch, and pushes only after the user approves. Use this skill whenever the user mentions WordPress updates, wp updates, updating WP core or plugins, monthly or routine WordPress maintenance, "wp-updates", or wants updates merged and pushed to staging, even if they don't name the workflow explicitly.
---

# WordPress updates workflow

A routine update cycle for a WordPress project tracked in git. Updates go on a dated branch, and each kind of update (core, plugins, theme) gets its own commit so any one can be reverted alone. The result is merged into the staging branch. Nothing is pushed without the user's explicit approval, because a push to staging may trigger a deploy or affect teammates.

This skill is repo-agnostic. Detect names and paths at run time instead of assuming them.

## 0. Pre-flight

- Find the WordPress root (the directory with `wp-config.php`) and run WP-CLI from there, with `--path` if needed. If `wp --info` fails or WP-CLI can't reach the database, stop and tell the user.
- `git status --short`: note existing uncommitted or untracked files. Leave unrelated files alone. Never use `git add -A` or `git add .`; stage only the paths each update changed, so unrelated work doesn't end up in an update commit. If tracked files have unrelated edits, tell the user before continuing.
- Detect branches with `git branch -a` (and `git symbolic-ref refs/remotes/origin/HEAD`):
  - Base branch: `master` or `main`, whichever exists.
  - Staging branch: `stage`, `staging` or similar. If there are several candidates or none, ask.
- Read `git log --oneline -5` and match the repo's commit-message style. The examples below are defaults.

## 1. Branch

1. Check out the base branch and `git pull`.
2. Name the new branch `wp_updates_{mon}_{yy}`, using a lowercase 3-letter month and 2-digit year, e.g. `wp_updates_nov_26`. Get it with `date +wp_updates_%b_%y | tr 'A-Z' 'a-z'`.
3. `git checkout -b <branch>`. If it already exists, ask whether to reuse it instead of overwriting it.

## 2. Core update

1. `wp core check-update`. If nothing is available, say so and go to step 3.
2. `wp core update`, then `wp core update-db`, then `wp core version` to confirm.
3. Check `git status`, stage the paths the update changed, and commit: `Updated WordPress core to <version>`. Layouts differ between projects (standard root, subdirectory or Bedrock-style installs), so let `git status` show what changed instead of guessing paths.

## 3. Plugin updates

1. `wp plugin list --update=available` to record the "before" state.
2. `wp plugin update --all`
3. Run `wp plugin list --update=available` again. Anything still listed is usually a pro or premium plugin (license-gated, not on wordpress.org) that WP-CLI couldn't update. Report these to the user in a table: plugin, current version, available version. Don't try to bypass licensing. If the user supplies updated pro plugin files or says they updated them manually, include those changes.
4. Stage the changed plugin directories and commit: `Updated plugins`, with a body listing `name: old -> new` for each. If pro plugins are updated separately, make a second commit: `Updated pro plugins`.

## 4. Theme updates (ask first)

1. `wp theme list` and check the `update` column.
2. Tell the user which themes have updates (name, current -> new version). If none, say so and move on.
3. Only if the user approves, run `wp theme update <name>` (or `--all` if they ask). Warn first if the theme is the active parent theme and has local modifications in git, since the update would overwrite them.
4. Commit separately: `Updated theme <name> to <version>`.

## 5. Merge into staging

1. Check out the staging branch and `git pull`.
2. `git merge <wp_updates branch>`. If there are conflicts, resolve them with the user and don't silently drop either side.
3. Summarise for local review: commits merged, core version, plugins updated, plugins still pending (pro), theme status. Suggest the user check the site locally (admin loads, front end renders, key plugin pages work).

## 6. Push (only with permission)

Ask: "Push `<staging-branch>` to its remote?" Push only after a clear yes, using the branch's configured upstream (`git push`, or `git push <remote> <branch>` if no upstream is set). Don't push the base branch or the update branch unless the user asks.

## Notes

- If a WP-CLI command errors (for example a fatal PHP error after a plugin update), stop, show the error, and offer to revert that commit instead of continuing.
- End with a short report: branch name, commits created, what's still pending, and whether anything was pushed.
