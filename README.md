# DeepSeek Harness Plugin Checker - GitHub Action

A GitHub Action that checks the correctness of a [DeepSeek Harness](https://www.deepseek.com/harness/) (dsh) plugin.

It checks the plugin in the following order, cheapest first:

1. **Check the manifest** — `package.json` exists, is valid JSON, declares a `name`, declares `dsh.bundle.patch`, and the referenced patch file exists. A package without `dsh.bundle` is not a dsh plugin.
2. **Install with dsh** — runs `dsh plugin --profile web add`; fails if `dsh` cannot install it (e.g. failed pnpm install).
3. **Verify the layer applies** — runs `dsh --profile web --dump-config` and fails if the plugin does not appear as a bundle layer (`# == <name>`) in the composed config.

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

## How it works

1. Sets up Node.js and installs pnpm and the `@deepseek-ai/dsh` CLI (`npm install @deepseek-ai/dsh`) from npm.
2. Checks the plugin's `package.json` manifest statically (valid JSON, `name`, `dsh.bundle.patch` declared, patch file exists).
3. Runs `dsh plugin --profile <profile> add <rootPath>` to install the plugin into a fresh dsh profile under the runner home.
4. Runs `dsh --profile <profile> --dump-config` and verifies the plugin shows up as a bundle layer (`# == <name>`), confirming the patch actually loads.

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
- `examples/broken-plugin` — a plugin with an invalid `package.json` that must fail the check.

## License

Apache License 2.0
