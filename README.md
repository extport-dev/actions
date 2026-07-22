# extport-dev/actions

GitHub Actions for publishing browser extensions with [extport](https://dash.extport.dev). Both actions are thin wrappers around [`@extport/cli`](https://www.npmjs.com/package/@extport/cli) run via `npx` — all the real logic lives in the CLI.

## `push`

Uploads an artifact (or, for Safari, just registers a version already delivered to App Store Connect).

```yaml
- uses: extport-dev/actions/push@v1
  with:
    file: dist.zip
    extension: my-extension
    version: 1.2.3
    store: chrome # chrome | firefox | edge | safari — omit to push every configured store
    api-key: ${{ secrets.EXTPORT_API_KEY }}
```

| Input | Required | Description |
| --- | --- | --- |
| `file` | only when `store` isn't `safari` | Path to the zip to upload |
| `extension` | no | Falls back to the repo's `extport.config.json` |
| `version` | yes | 1-4 dot-separated integers |
| `store` | no | `chrome`, `firefox`, `edge`, or `safari` |
| `source-zip` | no | Source zip for Firefox AMO review (`store: firefox` only) |
| `api-key` | yes | Pass `secrets.EXTPORT_API_KEY` |
| `api-url` | no | Defaults to `https://dash.extport.dev` |
| `cli-version` | no | Pinned `@extport/cli` version this action tag was tested against |

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
| `issuer-id` | no | App Store Connect API issuer id |
| `key-id` | no | App Store Connect API key id |
| `key-base64` | no | Base64-encoded `.p8` key contents — never commit the raw file |
| `platform` | no | `macos` or `ios` — omit to build every platform the project ships |
| `version` | no | Fails loudly if the built app's version doesn't match |
| `macos-deployment-target` | no | Defaults to `12.0` |
| `cli-version` | no | Pinned `@extport/cli` version this action tag was tested against |

## Versioning

Each action tag (`v1`, `v1.1`, …) pins a specific `@extport/cli` version internally (the `cli-version` default). Bumping the CLI version and cutting a new action tag happen together, so a given action version's behavior never changes silently underneath you.
