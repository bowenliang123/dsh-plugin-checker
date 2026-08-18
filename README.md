# DeepSeek Harness Plugin Checker - GitHub Action

A GitHub Action that checks the correctness of a [DeepSeek Harness](https://www.deepseek.com/harness/) (dsh) plugin.

It does the following steps:

1. Prepare Node.js, pnpm, and the `@deepseek-ai/dsh` CLI.
2. Run `dsh plugin --profile web add` to install the plugin from your repository.
3. Fail the check if `dsh` cannot install it (e.g. missing/invalid `package.json`, failed pnpm install).

## Usage

Add the action to your plugin repository's workflow:

```yaml
name: ci

on:
  push:
    branches: [main]
  pull_request:

jobs:
  check-plugin:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - name: Check the DSH plugin
        uses: bowenliang123/dsh-plugin-checker@v1
```

This checks the repository root. To check a plugin that lives in a subdirectory, pass `rootPath`:

```yaml
      - name: Check the DSH plugin
        uses: bowenliang123/dsh-plugin-checker@v1
        with:
          rootPath: plugins/my-plugin
```

## Inputs

| Input          | Description                                                                        | Required | Default    |
| -------------- | ---------------------------------------------------------------------------------- | -------- | ---------- |
| `rootPath`     | Path to the plugin root directory, relative to the repository root.                | No       | `.`        |
| `profile`      | dsh profile to install the plugin into.                                            | No       | `web`      |
| `dshVersion`   | Version of the `@deepseek-ai/dsh` npm package to install. Use `latest` for newest. | No       | `latest`   |
| `pnpmVersion`  | Version of pnpm to install (`dsh` plugin management runs pnpm internally).         | No       | `latest`   |
| `nodeVersion`  | Node.js version to run the dsh CLI with.                                           | No       | `24`       |

## Outputs

| Output       | Description                        |
| ------------ | ---------------------------------- |
| `pluginName` | The package name of the checked plugin. |

## How it works

1. Sets up Node.js and installs pnpm and the `@deepseek-ai/dsh` CLI (`npm install @deepseek-ai/dsh`) from npm.
2. Runs `dsh plugin --profile <profile> add <rootPath>` to install the plugin into a fresh dsh profile under the runner home.
3. Verifies the plugin's package name appears in the profile's `dsh.profile.bundles` layer list and that its `dsh.bundle.patch` file exists.

A plugin `package.json` looks like:

```json
{
  "name": "my-dsh-plugin",
  "version": "0.1.0",
  "type": "module",
  "main": "index.js",
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

## Examples

- `examples/hello-plugin` — a minimal valid plugin that passes the check.
- `examples/broken-plugin` — a directory without a `package.json` that must fail the check.

## License

MIT
