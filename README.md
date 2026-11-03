# empo-deps

Prebuilt binary dependencies for [empo-app](https://github.com/mateo-m/empo-app). The files are too large for the git history of empo-app, so this repo keeps them as release assets. The repo is public.

The repo has no source code. It has the releases and the workflow that builds the ANGLE release.

## Releases

Run `gh release list -R mateo-m/empo-deps -L 100` to see all releases.

### ANGLE

Each `angle-*` release has a tarball of the ANGLE static libraries and headers. empo-app pins one release in `ios/Dependencies/ANGLE/.version`.

| Tag | Asset | Contents |
|---|---|---|
| `angle-2026-11-01` | `angle-apple-prebuilt.tar.gz` | iphoneos, iphonesimulator and macosx, arm64. This is the release that empo-app uses. |
| `angle-2026-10-01` | `angle-ios-prebuilt.tar.gz` | iphoneos and iphonesimulator, arm64 |
| `angle-2026-05-04` | `angle-ios-prebuilt.tar.gz` | iphoneos and iphonesimulator, arm64 |

The tarball has an `include/` and a `lib/` folder for each SDK. `lib/` has `libANGLE_static.a`, `libGLESv2_static.a` and `libEGL_static.a`. The release notes give the sha256 of the tarball. Newer notes also give the ANGLE commit.

### Native deps

The 38 `native-*` releases, from `native-2026-06-22` to `native-2026-09-27-r2`, have the asset `native-ios-prebuilt.tar.gz`. It holds the iOS build folders of OpenSSL, SDL2, Ruby and the mkxp core. The `deps-publish.yml` workflow of empo-app made these releases. empo-app stopped using them in [empo-app#164](https://github.com/mateo-m/empo-app/pull/164), and now gets the engine from the releases of [mkxp-z-apple-mobile](https://github.com/mateo-m/mkxp-z-apple-mobile). Older commits of empo-app still download them, so do not delete them.

## Make an ANGLE release

The "ANGLE prebuilt" workflow (`.github/workflows/angle.yml`) runs `tools/angle/build.sh` of empo-app on the GitHub Mac runner `xcode-27`, with Xcode 27.2. On that image, Xcode 27.2 is the release candidate (build 27B48). The release notes give the Xcode version of each build.

1. Open **Actions** > **ANGLE prebuilt** > **Run workflow**.
2. Set **empo-app branch, tag, or full commit SHA** to the empo-app version to build from. A short commit SHA fails.
3. Set **Tag suffix** to the part after `angle-`, for example `2026-12-01-r2`. Use only letters, digits, `.`, `_` and `-`. Leave it empty to use the UTC date of today.
4. To only test the build, leave **Create the release** off. The run keeps `angle-apple-prebuilt.tar.gz` as an artifact for 90 days. The run summary gives its sha256.
5. To publish, turn on **Create the release**. The workflow creates the release with the tarball, writes the sha256 in the notes, and attests the build provenance. It stops before the build when the release or the tag exists. Only one run goes at a time.

The build takes about 20 minutes.

The artifact is the tarball itself, not a zip, so `gh run download` cannot get it. Use the API. Put the run ID in `RUN`:

```sh
ID=$(gh api repos/mateo-m/empo-deps/actions/runs/$RUN/artifacts --jq '.artifacts[0].id')
gh api repos/mateo-m/empo-deps/actions/artifacts/$ID/zip >angle-apple-prebuilt.tar.gz
```

empo-app uses the ANGLE release that its mkxp-z core was built with. When the two differ, the empo-app build stops. To move empo-app to a new release:

1. Set `ANGLE_VERSION` in `deps/common.make` of [mkxp-z-apple-mobile](https://github.com/mateo-m/mkxp-z-apple-mobile) to the new tag, and make a new core release.
2. The "Dependency updates" workflow of empo-app runs each day. It reads the ANGLE tag from the new core and opens a pull request that moves both pins. To start it now, run that workflow by hand.

To check that the workflow of this repo built a tarball:

```sh
gh attestation verify angle-apple-prebuilt.tar.gz -R mateo-m/empo-deps
```

Releases before the workflow, up to `angle-2026-11-01`, were built by hand. They have no attestation.

## How empo-app downloads ANGLE

`tools/fetch-angle.sh` in empo-app reads `ios/Dependencies/ANGLE/.version`. It downloads the release asset, checks the sha256, and extracts the tarball into `ios/Dependencies/ANGLE/`. It does nothing when a stamp file has the same version and the libraries are on disk.
