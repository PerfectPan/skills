---
name: release-setup
description: Set up or migrate an npm repository's release flow to Changesets with a draft version PR and Trusted Publishing. Use when adding releases to a new package repository, aligning an existing release flow with this standard, or auditing why a version PR, changelog or publish behaves differently from it.
---

# Changesets Release Setup

The standard flow: a feature PR adds a `.changeset/*.md` record; every push to `main` runs one Release workflow; with records pending, `changesets/action` opens or updates the Draft PR `chore(release): version packages`, whose body shows each package's new changelog section with PR links and `Thanks @author!`; merging that PR publishes every unpublished version through npm Trusted Publishing and creates one tag and GitHub Release per package (`<name>@<version>`; a single package at the repository root gets `v<version>`). The tools do the work: `@changesets/cli`, `@changesets/changelog-github` and `changesets/action`. A repository adds only configuration, its own gate commands and, when it needs one, the small release-plan check in [assets/check-release-plan.mjs](assets/check-release-plan.mjs). It does not need a shared release library or custom versioning scripts.

[assets/](assets/) holds the standard files with placeholders. `PerfectPan/project-template-pnpm` becomes the canonical source once its Release workflow uses `changesets/action` and its `.changeset/config.json` uses changelog-github; until then the assets win, and the template is migrated like any other repository. After that, when the two differ, the template wins; update the skill afterwards.

Changing workflows, npm trusted publishers or repository settings, pushing, and merging stay within the user's current authorization; this skill grants none. Follow the target repository's own AGENTS.md, CI and PR rules.

## 1. Audit the target

Record each finding before editing; every later step depends on it.

- **Visibility**: `gh api repos/OWNER/REPO -q .visibility`. npm generates provenance only for packages published from a public repository. A private repository still publishes through Trusted Publishing but gets no attestation; say so instead of promising provenance.
- **Packages**: list every workspace package with its `name`, `version` and `private` flag. Decide whether the public packages share one version (a `fixed` group) or version independently (`fixed: []`, also used for a single package). Private and `ignore`d packages are never versioned or tagged.
- **Current release machinery**: release workflows, version or publish scripts, ledgers, pnpm `versioning` settings, release-related `package.json` scripts, tests of those scripts, and the runbook that describes them. Everything Changesets replaces gets deleted, not kept beside it. A repository may be half-migrated (for example an unpinned `changesets/action@v2` or a `release` script beside the old flow); record which parts already match.
- **Record producers and tag readers**: automation that writes `.changeset/*.md` or reads `.changeset/config.json` (for example a changesets-renovate workflow) must keep working with the new config. Scripts, docs or workflows that read the old tag format (such as `v<version>` in a workspace that moves to `<name>@<version>`) must change with it.
- **Trusted publisher binding**: on npmjs.com, each package's trusted publisher names a workflow **file name**. Keep publishing from that file (rename only after changing every package's trusted publisher first). A package name that was never published needs a one-time manual first publish before OIDC can apply to it.
- **CI setup and gates**: copy the Node/pnpm setup step and the gate commands from the repository's `ci.yml`; the asset's `.node-version` and `pnpm run check` are placeholders.
- **CI trigger**: the version PR is a Draft and is updated by `GITHUB_TOKEN`, which starts no workflows. CI must run on `pull_request` `ready_for_review` (or `workflow_dispatch`) so the maintainer can validate the version PR before merging.
- **Repository setting**: Settings → Actions → General → Workflow permissions → "Allow GitHub Actions to create and approve pull requests" must be on, or the action cannot open the version PR.
- **Open release state**: an open version PR or release branch left by the old flow is not reused; plan to close it after the migration merges.

Done when every item above has a recorded value or an explicit unknown.

## 2. Apply the standard

1. Add dev dependencies `@changesets/cli` (3.x; `changesets/action` v2 rejects 2.x) and `@changesets/changelog-github` (plus `@changesets/get-release-plan` 5.x if you keep the release-plan check; the script uses its named export), pinned to exact versions: the workflow parses the output of `changeset publish-plan`, which the CLI marks experimental.
2. Write `.changeset/config.json` from [assets/config.json](assets/config.json): set the `repo` for changelog-github, the `fixed` group (or `[]` for independent versions), `access: "public"`, `baseBranch`, and `privatePackages: { version: false, tag: false }`. Add `.changeset/README.md` only if the repository documents change records there.
3. Write the Release workflow from [assets/publish-npm.yml](assets/publish-npm.yml) under the trusted publisher's file name. Replace the setup step and the gate steps with the repository's own (from `ci.yml`), so the release runs the complete gate on the same toolchain. Keep:
   - `fetch-depth: 0`: changelog-github looks up the commit that added each record;
   - `persist-credentials: false`: repository code runs during the gates; the action pushes through the API with its own token;
   - `id-token: write` with no npm token secret;
   - `pr-draft: always` and a `publish-script` that is set even when the gates are skipped, otherwise the version PR body tells maintainers to publish by hand;
   - the action pinned to an exact release tag (for example `changesets/action@v2.1.2`), because the job holds write and OIDC permissions.
4. Add `"version-packages": "changeset version"` (append repository steps that must see the new versions, such as generated descriptors) and `"changeset": "changeset"` to the root scripts.
5. If the workspace has any private or `ignore`d package, add [assets/check-release-plan.mjs](assets/check-release-plan.mjs) to the complete gate. A record naming only such a package is never consumed by `changeset version`, so it stays pending and the action never publishes again. The script also caps 0.x releases at minor, since Changesets has none; delete that block for packages at 1.x or later.
6. Delete the replaced machinery from step 1, its tests and docs references. Rewrite the release runbook around the new procedure; [references/operations.md](references/operations.md) has the procedure and recovery text to adapt.
7. Convert any pending release notes from the old flow into `.changeset/*.md` records so the first version PR carries them.

Done when the repository has no release code besides the workflow, the config, the scripts entry and the optional check, every audited record producer and tag reader works with the new setup, and the runbook describes only the new flow.

## 3. Verify before merging

- `pnpm changeset status --verbose` lists the expected pending bumps; `pnpm changeset publish-plan` lists nothing while every committed version is already on npm.
- Optionally run `pnpm run version-packages` on a scratch branch and inspect the generated changelogs, then discard it. changelog-github calls the GitHub API, so locally it needs `GITHUB_TOKEN` (for example `GITHUB_TOKEN=$(gh auth token)`).
- Run the repository's complete gate and get the migration PR reviewed.

Done when the status output matches the expected bumps, `publish-plan` is empty, the gate passes and the review has no open blocking finding.

## 4. Verify the first real release

The migration is proven only by a release that went through it end to end:

1. After the migration merges, the Release run opens a Draft version PR from `changeset-release/main`. Check its body for per-package sections, PR links and `Thanks @author!`. Close any old-flow version PR. If no PR appears, check the run log and the "Allow GitHub Actions to create and approve pull requests" setting.
2. Mark it Ready for review, wait for CI, and merge only when the user asks.
3. Confirm the publish run succeeded, then check each package: `npm view <name>@<version> version`, `npm view <name> dist-tags.latest`, the tags (`git ls-remote --tags origin`), and one GitHub Release per package with the changelog section. For a public repository also check `npm view <name>@<version> dist.attestations`.

Report each check with its evidence. A started workflow or one published package is not a completed release.
