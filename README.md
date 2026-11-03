# empo-deps

Prebuilt binary dependencies for [empo-app](https://github.com/mateo-m/empo-app). The files are too large for the git history of empo-app, so this repo keeps them as release assets. The repo is public.

The repo has no source code. It has the releases and the workflow that builds the ANGLE release.

## Releases

Run `gh release list -R mateo-m/empo-deps` to see all releases.

### ANGLE

Each `angle-*` release has a tarball of the ANGLE static libraries and headers. empo-app pins one release in `ios/Dependencies/ANGLE/.version`.

| Tag | Asset | Contents |
|---|---|---|
| `angle-2026-11-01` | `angle-apple-prebuilt.tar.gz` | iphoneos, iphonesimulator and macosx, arm64. This is the release that empo-app uses. |
| `angle-2026-10-01` | `angle-ios-prebuilt.tar.gz` | iphoneos and iphonesimulator, arm64 |
| `angle-2026-05-04` | `angle-ios-prebuilt.tar.gz` | iphoneos and iphonesimulator, arm64 |

The tarball has an `include/` and a `lib/` folder for each SDK. `lib/` has `libANGLE_static.a`, `libGLESv2_static.a` and `libEGL_static.a`. The release notes give the sha256 of the tarball and the ANGLE commit.

### Native deps

The 38 `native-*` releases, from `native-2026-06-22` to `native-2026-09-27-r2`, have the asset `native-ios-prebuilt.tar.gz`. It holds the iOS build folders of OpenSSL, SDL2, Ruby and the mkxp core. The `deps-publish.yml` workflow of empo-app made these releases. empo-app stopped using them in [empo-app#164](https://github.com/mateo-m/empo-app/pull/164), and now gets the engine from the releases of [mkxp-z-apple-mobile](https://github.com/mateo-m/mkxp-z-apple-mobile). Older commits of empo-app still download them, so do not delete them.

## Make an ANGLE release

The "ANGLE prebuilt" workflow (`.github/workflows/angle.yml`) runs `tools/angle/build.sh` of empo-app on a GitHub Mac runner with Xcode 27.2.

1. Open **Actions** > **ANGLE prebuilt** > **Run workflow**.
2. Set **empo-app branch, tag, or commit** to the empo-app version to build from.
3. Set **Tag suffix** to the part after `angle-`, for example `2026-12-01-r2`. Leave it empty to use the UTC date of today.
4. To only test the build, leave **Create the release** off. The run keeps `angle-apple-prebuilt.tar.gz` as an artifact for 90 days. The run summary gives its sha256.
5. To publish, turn on **Create the release**. The workflow creates the release with the tarball, writes the sha256 in the notes, and attests the build provenance. It stops before the build when the release exists.

The build takes about 20 minutes.

The artifact is the tarball itself, not a zip, so `gh run download` cannot get it. Use the API. Put the run ID in `RUN`:

```sh
ID=$(gh api repos/mateo-m/empo-deps/actions/runs/$RUN/artifacts --jq '.artifacts[0].id')
gh api repos/mateo-m/empo-deps/actions/artifacts/$ID/zip >angle-apple-prebuilt.tar.gz
```

Then pin the new release in empo-app. Use the tag and the sha256 from the release notes:

```sh
cat > ios/Dependencies/ANGLE/.version <<VERSION
ANGLE_VERSION=angle-2026-12-01
ANGLE_SHA256=<sha256 from the release notes>
ANGLE_ASSET=angle-apple-prebuilt.tar.gz
VERSION
```

To check that the workflow of this repo built a tarball:

```sh
gh attestation verify angle-apple-prebuilt.tar.gz -R mateo-m/empo-deps
```

Releases before the workflow, up to `angle-2026-11-01`, were built by hand. They have no attestation.

## How empo-app downloads ANGLE

`tools/fetch-angle.sh` in empo-app reads `ios/Dependencies/ANGLE/.version`. It downloads the asset with `gh release download`, checks the sha256, and extracts the tarball into `ios/Dependencies/ANGLE/`. It does nothing when a stamp file has the same version and the libraries are on disk.
