# extport-dev/actions

GitHub Actions for publishing browser extensions with [extport](https://dash.extport.dev). Both actions are thin wrappers around [`@extport/cli`](https://www.npmjs.com/package/@extport/cli) run via `npx` — all the real logic lives in the CLI.

## `push`

Uploads an artifact (or, for Safari, just registers a version already delivered to App Store Connect).

```yaml
# Explicit:
- uses: extport-dev/actions/push@v1
  with:
    file: dist.zip
    extension: my-extension
    version: 1.2.3
    store: chrome # chrome | firefox | edge | safari
    api-key: ${{ secrets.EXTPORT_API_KEY }}

# In a WXT project, this pushes every enabled store configured for the
# extension, inferring each store's zip from .output/ and its version from
# the zip's own manifest.json (package.json for safari, which has no zip):
- uses: extport-dev/actions/push@v1
  with:
    api-key: ${{ secrets.EXTPORT_API_KEY }}
```

| Input | Required | Description |
| --- | --- | --- |
| `file` | no | Path to the zip to upload. In a WXT project, omit to infer `.output/{name}-{version}-{store}.zip` (always omit for `store: safari`) |
| `extension` | no | Falls back to the repo's `extport.config.json` |
| `version` | no | 1-4 dot-separated integers. Omit to infer it from the zip's manifest.json, or package.json for `store: safari` |
| `store` | no | `chrome`, `firefox`, `edge`, or `safari`. With `file` set, omit for one universal zip to every configured store. With both `file` and `store` omitted, pushes every enabled store individually |
| `source-zip` | no | Source zip for Firefox AMO review (`store: firefox` only) |
| `api-key` | yes | Pass `secrets.EXTPORT_API_KEY` |
| `api-url` | no | Defaults to `https://dash.extport.dev` |
| `cli-version` | no | Pinned `@extport/cli` version this action tag currently runs |

## `safari-build`

Builds, signs, and uploads a Safari extension to App Store Connect. Needs a macOS runner (`runs-on: macos-latest`). Doesn't submit for review, and doesn't register the version with extport — follow it with `push` (`store: safari`, no `file`) for that.

```yaml
- uses: extport-dev/actions/safari-build@v1
  with:
    project-path: ./ios
    team-id: ${{ secrets.APPLE_TEAM_ID }}
    issuer-id: ${{ secrets.APPLE_API_ISSUER }}
    key-id: ${{ secrets.APPLE_API_KEY_ID }}
    key-base64: ${{ secrets.APPLE_API_KEY }} # base64-encoded .p8 contents
```

| Input | Required | Description |
| --- | --- | --- |
| `project-path` | yes | Directory containing the `.xcodeproj` |
| `team-id` | yes | Apple Developer Team ID |
| `issuer-id` | no | App Store Connect API issuer id. Omit to fall back to the checked-out repo's `extport.config.json` |
| `key-id` | no | App Store Connect API key id. Omit to fall back to the checked-out repo's `extport.config.json` |
| `key-base64` | no | Base64-encoded `.p8` key contents — never commit the raw file |
| `platform` | no | `macos` or `ios` — omit to build every platform the project ships |
| `version` | no | Fails loudly if the built app's version doesn't match |
| `macos-deployment-target` | no | Defaults to `12.0` |
| `cli-version` | no | Pinned `@extport/cli` version this action tag currently runs |

## Versioning

`v1` moves forward in place to track fixes and the latest compatible `@extport/cli` (the `cli-version` default) — the usual convention for a major-version action tag (same as `actions/checkout@v4`). Pin `cli-version` yourself if you need a specific `@extport/cli` release regardless of what `v1` currently defaults to.
