# Release Operations

Adapt this text into the target repository's release runbook. Replace the gate commands and package names with the repository's own.

## Normal procedure

1. A PR that changes a published package adds a record with `pnpm changeset`, or non-interactively with `pnpm changeset add --patch <package> -m "<release note>"` (`--minor` for features; separate several packages with commas). Documentation-only and repository-only PRs need none. Reviewers check `.changeset/` when a PR changes a package.
2. Merging the PR pushes to `main`. The Release workflow runs `changeset publish-plan`; with nothing to publish it skips the gates. `changesets/action` sees pending records, runs `version-packages`, commits to `changeset-release/main` through the GitHub API (GitHub signs the commit) and opens or updates the Draft PR. `changeset version` deletes the consumed records and writes each package's `CHANGELOG.md`; it does not tag or touch the lockfile.
3. Every update turns the version PR back into a Draft and starts no CI. Review the versions, changelogs and consumed records, select Ready for review, and merge with current CI. Merging is the release decision.
4. The merge push leaves no record pending, `publish-plan` lists the new versions, the gates run, and `changeset publish` runs `pnpm publish` for each unpublished public version in dependency order through Trusted Publishing. The action pushes one tag per published package (`<name>@<version>`, or `v<version>` for a single root package) and creates one GitHub Release for each.
5. Verify each exact version on npm, the tags and the Releases before calling the release done.

Runs share one concurrency group with `cancel-in-progress: false`: a push waits for the running run, and GitHub keeps only the newest waiting run, cancelling older waiting ones.

## Recovery

The action publishes only when no record is pending; with records pending it prepares the version PR instead, even if an earlier version is still unpublished. Recovery therefore re-runs the run of the release commit itself whenever it can.

- **Version step failed** (`changeset version`, changelog lookup): nothing changed on `main` or the release branch. Re-run the latest run for a transient failure; otherwise fix the cause in a reviewed PR, whose merge regenerates the version PR. A missing `GITHUB_TOKEN` fails this step.
- **Gates failed before publication**: nothing is tagged or published. Re-run for a transient failure. Otherwise fix the cause in a reviewed PR: a fix without records publishes the same version from its merge push; a fix with records goes into the next version PR and that unpublished version is skipped.
GitHub allows re-running a run for 30 days; after that, only **Run workflow** on `main` remains, and only while no record is pending.

- **The version PR's merge run was cancelled or failed**: re-run that run. It keeps the merge commit, where no record is pending, so it publishes. If a later push with records ran instead, it prepared the next version PR and the merged version stays unpublished until this re-run. Avoid merging PRs with records while the merge run is still waiting.
- **Publication stopped part-way**: re-run the failed run; `changeset publish` skips versions already on npm and tags the rest on the run's commit. **Run workflow** on `main` does the same only while no record is pending, and it tags the current `main` commit instead of the release commit. Never bump the version to retry, and never unpublish.
- **Published but the tag or Release is missing**: `publish-plan` no longer lists the version, so re-runs do not add them. Create them by hand on the release commit: `gh release create '<tag>' --target <release commit>` (the tag format from step 4) with the package's changelog section as notes.
- **Version PR closed by hand or out of date**: **Run workflow** on `main` rebuilds it from the pending records.
- **Trusted Publishing rejected one package**: fix that package's trusted publisher on npmjs.com (owner, repository, workflow file name, environment) and re-run. One package's authorization does not cover the others.
- **A record is never consumed**: it names a private or ignored package. Remove it or move its note to a released package; the release-plan check names it.
