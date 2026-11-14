---
name: release-setup
description: Set up or migrate an npm repository's release flow to Changesets with a draft version PR and Trusted Publishing. Use when adding releases to a package repository, aligning an existing flow with this standard, or auditing version PR, changelog or publish behavior.
---

# Changesets Release Setup

Use `@changesets/cli`, `@changesets/changelog-github` and `changesets/action` for change records, version PRs,
changelogs and publication. The repository supplies configuration and its own checks. This skill describes adoption;
[PerfectPan/project-template-pnpm](https://github.com/PerfectPan/project-template-pnpm) owns the implementation.

## Authoritative source

The template migration is under review in [PR #5](https://github.com/PerfectPan/project-template-pnpm/pull/5).
Until it merges, use the implementation at
[commit 9c57f3d](https://github.com/PerfectPan/project-template-pnpm/tree/9c57f3d580537f2474ac97bbe4b691d59381d73d),
which includes these files:

- [Release workflow](https://github.com/PerfectPan/project-template-pnpm/blob/9c57f3d580537f2474ac97bbe4b691d59381d73d/.github/workflows/publish-npm.yml).
- [Changesets config](https://github.com/PerfectPan/project-template-pnpm/blob/9c57f3d580537f2474ac97bbe4b691d59381d73d/.changeset/config.json).
- [Release-plan check](https://github.com/PerfectPan/project-template-pnpm/blob/9c57f3d580537f2474ac97bbe4b691d59381d73d/scripts/check-release-plan.mjs)
  and [first-publish check](https://github.com/PerfectPan/project-template-pnpm/blob/9c57f3d580537f2474ac97bbe4b691d59381d73d/scripts/check-first-publish.mjs),
  with regression tests in `scripts/test/`.
- [Release runbook](https://github.com/PerfectPan/project-template-pnpm/blob/9c57f3d580537f2474ac97bbe4b691d59381d73d/docs/development/release.md)
  for setup, operation and recovery.

After the migration merges, read the template's current `main`. Read these sources before editing the target;
adapt its toolchain and package policy rather than maintaining another implementation in this skill.

Changing workflows, npm trusted publishers or repository settings, pushing, merging and publishing stay within
the user's current authorization. Follow the target repository's AGENTS.md, CI and PR rules.

## 1. Audit the target

Record the following before editing:

- Every workspace package's name, version and private flag; the existing `fixed`, `linked`, `ignore` and
  `privatePackages` settings. Decide whether public packages version independently or share a fixed group.
- Release workflows, scripts, tests, ledgers, pnpm `versioning` settings and runbooks that Changesets replaces.
  Include automation that creates records and consumers of the existing tag format.
- Each package's npm first-publish state and trusted publisher binding. npm binds the workflow file name per
  package; retain that name unless its publisher configuration changes too. Preserve an existing requirement
  to complete first-publish setup before opening a version PR.
- The repository's Node/pnpm setup and complete gate commands. CI needs `pull_request: ready_for_review` so
  maintainers can validate the draft version PR; an updated draft does not prove CI ran.
- Repository visibility, whether Actions may create PRs, and any open version PR or release branch from the old
  flow. Public repositories can produce npm provenance; private repositories do not get that attestation.

Resolve any unknown that changes package selection, publishing authorization or required gates before that step.

## 2. Apply the standard

1. Adapt the template config and workflow. Keep `changesets/action/select-mode@v2.1.2` and
   `changesets/action@v2.1.2` pinned. Use the selector's `mode` output: run the complete gate and production audit
   only for `publish`, skip the main action for `none`, and let the main action own versioning and publication.
2. Keep full Git history for changelog lookup, `persist-credentials: false`, `id-token: write`, the `main` job
   restriction, `pr-draft: always` and a configured `publish-script`. Use the target's setup and gate commands.
   npm Trusted Publishing uses OIDC; the workflow needs no npm token secret.
3. Add exact Changesets dependency versions from the template and the scripts `changeset` and `version-packages`.
   Keep target-specific work that must run after `changeset version`, such as generating package descriptors.
   Let changelog-github read `GITHUB_REPOSITORY`; local previews also need that variable and `GITHUB_TOKEN`.
4. For private or ignored packages, adapt the template release-plan check and its dependencies. It uses parsed
   Changesets config and native `shouldSkipPackage` to reject records Changesets will leave pending, including
   `none` records. Preserve an intentional target bump policy; the template itself supports major releases.
5. Where first-publish setup is a prerequisite, adapt the template's thin check using native release candidates
   and `npm view`. In version mode, an unrelated new package must not block an existing package's version PR;
   the affected new package must still stop until setup is complete. Registry errors must remain errors.
6. Remove replaced release machinery and its obsolete tests, required-file checks and documentation references.
   Convert pending notes to Changesets records. Adapt the template runbook to the target's package names, workflow
   filename and gates; retain recovery instructions that apply to its release state.
7. Update tag consumers using Changesets' workspace detection: a pnpm workspace uses `<name>@<version>` even
   when it has one published package. `v<version>` is for a standalone root package without a workspace.

Done when config, workflow, checks, record producers, tag consumers and runbook describe the same flow, with no
second release implementation left behind.

## 3. Verify the migration

- Run `pnpm changeset status --verbose` and compare every pending package and bump with the intended release.
- Run the target's complete gate and tests for changed release behavior. Cover skipped-package `none` records;
  when first-publish checks apply, cover an affected new package, an unrelated new package and registry failure.
- Optionally preview `version-packages` in a disposable checkout with `GITHUB_REPOSITORY` and `GITHUB_TOKEN`,
  inspect generated versions and changelogs, and discard the output. This does not publish.
- Follow the repository's commit, push and PR workflow. Stop at the user's authorized delivery boundary.

## 4. Verify an authorized real release

After the migration merges, confirm the Release run opens or updates the draft version PR, whose body contains
per-package changelog sections, PR links and author thanks. Mark it Ready, merge and publish only within the
user's authorization. Close an obsolete old-flow version PR only when authorized.

After publication, verify each exact npm version, `dist-tags.latest`, its tag and GitHub Release. For a public
repository also check `dist.attestations`. Use the template runbook for recovery; a started workflow or one
published package does not prove the entire release completed.
