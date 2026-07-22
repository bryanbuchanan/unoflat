---
name: publish
description: Publish a new version of the Uno Flat theme to the Zed extension registry. Verifies the version bump and changelog, pushes this repo, then opens a PR to zed-industries/extensions that updates the submodule and version. Use when the user wants to release/publish/ship a new version of the theme.
---

# Publish Uno Flat to the Zed extension registry

Publishing has two halves: releasing this repo (bryanbuchanan/unoflat), then opening a PR to
`zed-industries/extensions` from the existing fork `bryanbuchanan/extensions` that bumps the
submodule pin and version. Follow the steps in order; each later step depends on the earlier ones.

If anything is ambiguous (version jump looks wrong, unclear what changed, unexpected diff), stop
and ask the user before opening the external PR. Otherwise proceed without pausing.

## 1. Determine and validate the version

1. Read `NEW_VERSION` from `version` in `extension.toml`.
2. Fetch the currently published version:
   ```sh
   gh api repos/zed-industries/extensions/contents/extensions.toml --jq .content | base64 -d | grep -A2 '^\[unoflat\]'
   ```
3. Validate as semver: `NEW_VERSION` must be strictly greater than the published version.
   - If they are equal (the user forgot to bump), bump the patch version in `extension.toml`.
   - If `NEW_VERSION` is *lower* than published, or skips ahead oddly (e.g. patch release jumping
     a minor), stop and ask the user which version is intended.
4. `CHANGELOG.md` must have a `### NEW_VERSION` heading at the top. If it's missing, write one:
   summarize the actual changes since the last release (`git diff <last-release-commit> -- src/theme.scss`
   and recent commits), matching the terse style of existing entries.

Compiling `src/theme.scss` → `themes/unoflat.json` is handled manually by the user before
publishing — do not recompile or hand-edit the JSON.

## 2. Commit and push this repo

The extensions repo pins a submodule commit that must be on `main` of bryanbuchanan/unoflat.

1. Make sure you're on `main`. Commit all release changes (`extension.toml`, `CHANGELOG.md`,
   `src/theme.scss`, `themes/unoflat.json`) with message `v{NEW_VERSION}: <one-line summary>`.
2. `git push origin main`
3. Record the release commit: `TARGET_COMMIT=$(git rev-parse HEAD)`

## 3. Open the PR to zed-industries/extensions

Work in a temp directory (use the session scratchpad), not inside this repo.

1. Sync the fork's main with upstream:
   ```sh
   gh repo sync bryanbuchanan/extensions --source zed-industries/extensions --branch main
   ```
2. Shallow-clone the fork and branch:
   ```sh
   git clone --depth 1 https://github.com/bryanbuchanan/extensions.git
   cd extensions
   git checkout -b unoflat-v{NEW_VERSION}
   ```
3. Update only the unoflat submodule to the latest main commit:
   ```sh
   git submodule update --init -- extensions/unoflat
   git submodule update --remote -- extensions/unoflat
   ```
   Verify `git -C extensions/unoflat rev-parse HEAD` equals `TARGET_COMMIT`. If it doesn't, stop
   and figure out why (unpushed commit, wrong branch) before continuing.
4. In the top-level `extensions.toml`, set `version = "{NEW_VERSION}"` in the `[unoflat]` entry.
   It must exactly match `extension.toml` at the pinned submodule commit.
5. If `pnpm` is available, run `pnpm sort-extensions` from the repo root (keeps `extensions.toml`
   and `.gitmodules` sorted). A version-only change shouldn't reorder anything; skip gracefully if
   pnpm isn't installed.
6. Commit and push to the fork, then open the PR against upstream:
   ```sh
   git add extensions/unoflat extensions.toml
   git commit -m "Update unoflat to v{NEW_VERSION}"
   git push -u origin unoflat-v{NEW_VERSION}
   gh pr create --repo zed-industries/extensions \
     --head bryanbuchanan:unoflat-v{NEW_VERSION} \
     --title "Update unoflat to v{NEW_VERSION}" \
     --body "<changelog entry for this version>"
   ```

## 4. Report

Tell the user: the released version, the unoflat release commit, and the PR URL. Once the PR merges,
Zed packages and publishes the extension automatically — no further action needed. Clean up the
temp clone.
