# flash-moe

`flash-moe` is an external mesh-llm plugin for connecting mesh-llm to a
Flash-MoE OpenAI-compatible HTTP server.

The plugin is intentionally decoupled from mesh-llm internals. mesh-llm only
launches the plugin through the plugin API, passes `[[plugin]].url` as
`MESH_LLM_PLUGIN_URL`, and consumes the plugin-declared inference endpoint.

## Install

```bash
mesh-llm plugins install flash-moe
```

You can also install directly from GitHub:

```bash
mesh-llm plugins install Mesh-LLM/flash-moe
```

## Existing Endpoint Mode

If Flash-MoE is already running, attach the endpoint:

```toml
[[plugin]]
name = "flash-moe"
url = "http://127.0.0.1:8000/v1"
```

## Managed Process Mode

To let the plugin supervise a local Flash-MoE `infer` process, pass the backend
command and backend args to the plugin:

```toml
[[plugin]]
name = "flash-moe"
args = [
  "--backend-command", "/opt/flash-moe/metal_infer/infer",
  "--",
  "--model", "/models/qwen3.5-397b/model.gguf",
  "--weights", "/models/qwen3.5-397b/experts.bin",
  "--manifest", "/models/qwen3.5-397b/manifest.json",
  "--vocab", "/models/qwen3.5-397b/vocab.json"
]
```

The plugin allocates a local port and appends:

```text
--serve <port>
```

Do not pass `--serve` in backend args.

## Compatibility

Version 0.1.1 requires a MeshLLM host that supports plugin protocol 3.
Protocol 2 hosts cannot load this version. The plugin SDK is pinned to MeshLLM
commit `efd30516fd2cd92ac132dfd7b8341d24d412ee9c` for reproducible builds.

## Build

```bash
cargo build --locked
```

## Release Archives

Push a `v*` tag from a validated commit to publish a release. To recover an
interrupted release, select the reviewed `main` branch in the `Release`
workflow and enter the existing version tag. The selected branch supplies
the workflow; builds check out `refs/tags/<version>`. Confirm that the tag
still points to the validated commit and no release assets have been published
before running it manually. macOS archives use the ARM64 `macos-15` runner.

GitHub releases package archives using the same contract as
`Mesh-LLM/blackboard`:

- `flash-moe-<target>.tar.gz` or `.zip`
- `flash-moe-<version>-<target>.tar.gz` or `.zip`

Each archive contains:

- `flash-moe/flash-moe`
- `flash-moe/plugin.toml`
