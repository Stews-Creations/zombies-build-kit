# Workspace development

This workspace uses Git submodules so each component has its own history and the parent records a reproducible combination of component commits. The [architecture overview](../ARCHITECTURE.md) defines ownership.

## Initialize or update components

From a clean parent checkout, retrieve the recorded revisions:

```powershell
git pull --ff-only
git submodule sync --recursive
git submodule update --init --recursive
git submodule status --recursive
```

Inspect and preserve uncommitted work in every affected component before updating. Submodule updates normally check out the recorded commit with a detached HEAD. A leading `+` in submodule status means a component differs from the parent pin; a leading `-` means it is not initialized. Do not use `git submodule update --remote` as the normal update workflow: it selects remote branch revisions instead of the tested pins.

## Change a component

Create a branch before editing a detached submodule. For example, from the parent folder:

```powershell
git -C datapacks switch -c codex/core-change
```

Make a scoped change and run the component's relevant checks. Stage the intended files, review the staged diff, and commit within that component. Push the component branch and complete its review/merge workflow before publishing a parent revision that depends on it.

Once the component change is available on its remote `main`, return the component checkout to the merged revision:

```powershell
git -C datapacks fetch origin
git -C datapacks switch main
git -C datapacks pull --ff-only
```

If a fresh submodule clone has no local `main` branch, use `git -C datapacks switch --track origin/main` instead of `switch main`. Confirm the component is at the intended reviewed commit, and repeat for other components changed together.

## Record and publish the combination

Run integration checks for the selected component revisions, then create a parent branch and record the updated pins:

```powershell
git switch -c codex/core-integration
git add datapacks
git diff --cached --submodule=log
git diff --cached --check
git commit -m "Update core datapack revision"
git push --recurse-submodules=check -u origin codex/core-integration
```

Stage every intentionally updated component, not only `datapacks` when changes span multiple repositories. The push check helps detect missing remote component commits; it does not replace review or compatibility checks. Publish component commits first and the parent pins second. Never force-update shared branches to make pins appear synchronized.

## Local and generated files

Every repository owns its own ignore and attribute rules. Parent rules do not replace a submodule's configuration. Before committing, inspect both the parent status and component status; an ignored file can still be added with force, and ignore rules do not untrack existing files.

Keep local instructions, assistant settings, prompts, and reports outside tracked source. Store ad hoc planning artifacts in the ignored `.codex/` directory. Keep durable product architecture and maintained validation tools versioned. When configuring a repository-local exclusion, resolve its location with `git rev-parse --git-path info/exclude`; a submodule's `.git` entry may be a file.

Text files use LF, with Windows batch scripts using CRLF. Binary assets are marked explicitly. Keep runtime assets and build inputs; ignore generated deliverables and caches by their output locations.

## Validation before publishing

Check each changed repository's diff and references. READMEs use one H1, balanced fenced blocks, working links, and current command paths. Run `git diff --check` and inspect staged files for local-only content.

For foundation changes, verify a fresh recursive clone resolves every submodule pin. For later implementation changes, also run component tests and installation/gameplay checks appropriate to the change. Runtime exports must be inspected independently of Git to ensure they contain only intended deliverables.
