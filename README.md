# empo-deps

Prebuilt binary dependencies for [empo-app](https://github.com/mateo-m/empo-app) that are too large to ship in the app repo's git history.

This repo holds **release tags only**. There's no source here. Each tag has a `.tar.gz` attached that the empo-app build script fetches at first build.

## Why not commit binaries to empo-app?

ANGLE static libs alone are ~80 MB. Tracking them inflates every fresh clone of empo-app and balloons the git pack. They update once every several months, so version-pinning them via tagged releases here is cheaper than carrying them in history.

## Releases

| Tag | What | Notes |
|---|---|---|
| `angle-2026-05-04` | iOS ANGLE prebuilt | iphoneos arm64 + iphonesimulator arm64 static libs + headers |

## Publishing a new ANGLE prebuilt

From an empo-app checkout with current binaries in `ios/Dependencies/ANGLE/`:

```sh
cd ios/Dependencies/ANGLE
TAG=angle-$(date +%Y-%m-%d)
tar -czf /tmp/angle-ios-prebuilt.tar.gz iphoneos/{include,lib} iphonesimulator/{include,lib}
SHA=$(shasum -a 256 /tmp/angle-ios-prebuilt.tar.gz | awk '{print $1}')

gh release create "$TAG" /tmp/angle-ios-prebuilt.tar.gz \
  --repo mateo-m/empo-deps \
  --title "ANGLE prebuilt $TAG" \
  --notes "sha256: $SHA"
```

Then bump `ios/Dependencies/ANGLE/.version` in empo-app:

```sh
cat > ios/Dependencies/ANGLE/.version <<VERSION
ANGLE_VERSION=$TAG
ANGLE_SHA256=$SHA
VERSION
```

Commit + push. Next build hydrates from the new release.

## How empo-app fetches

`tools/fetch-angle.sh` in empo-app reads `ios/Dependencies/ANGLE/.version`, calls `gh release download` against this repo, verifies sha256, extracts. Skips if a stamp file matches the requested version.

The repo is currently private; access requires `gh auth login`. When empo-app goes public this repo will go public too and the fetch script can switch to plain curl.
