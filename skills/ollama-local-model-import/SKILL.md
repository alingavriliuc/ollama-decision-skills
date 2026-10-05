---
name: ollama-local-model-import
description: Diagnose stalled Ollama model downloads, obtain model files from an alternative source such as Hugging Face, or import an existing GGUF into Ollama. Use when ollama pull or ollama run stalls during download, a proxy returns a block page, or a model must be installed offline.
---

# Ollama local model import

Install a model from a local file and verify it through the interface the user needs. For Tev1 download links, a known proxy failure on the model file host, and its decision API requirements, read [references/tev1.md](references/tev1.md).

Decision model verification relies on the companion skill `ollama-decision-models` from the same repository. If it is not installed, verify the decision endpoint with the request in the Tev1 reference and tell the user that the full contract lives in that skill.

## 1. Establish the model and the failing stage

Read the project's model configuration and existing `Modelfile`. Check:

```sh
ollama --version
ollama list
ollama ps
```

Reuse an installed model or an existing download when it matches the request. Preserve user-edited files and use a distinct local model name when importing an alternative artifact.

If the reported problem is a slow download, run one bounded reproduction. Capture elapsed time and download progress. On macOS, inspect `~/.ollama/logs/server.log` for the corresponding attempt. Distinguish downloading, loading, prompt processing, and generation before investigating performance.

For a network failure, inspect the HTTP status and redirects of the actual model file, rather than just its catalog page. A small range request can check access without downloading the full model:

```sh
curl -fLsS --connect-timeout 5 --max-time 20 \
  --range 0-1023 -o /dev/null \
  -w 'http=%{http_code} bytes=%{size_download} total=%{time_total}s\n' \
  "$MODEL_URL"
```

Some servers ignore range requests. Keep the timeout and distinguish HTTP errors from a transfer stopped by that timeout. Inspect response headers and a small body sample if an error or HTML block page appears. Redact signed URL query strings, credentials, and identifiers in reported artifacts.

This step is complete when the desired model and interface are known, and either a usable local artifact exists or the download failure has an observed cause.

## 2. Choose and download an artifact

Prefer an official downloadable GGUF when available. Otherwise identify a community conversion of the exact base model. Check its repository files and model card for architecture, quantization, size, and conversion details. Label community conversions explicitly and obtain the user's agreement before downloading one. Matching model names and quantization do not prove byte-for-byte equivalence with an Ollama package.

Confirm that the installed Ollama version supports the architecture. If only Safetensors weights exist, check architecture-specific import support before choosing direct import or a documented conversion to GGUF.

Test the artifact URL from the user's environment. Browser access to a model page does not prove access to the redirected file host. If the alternative is also blocked, report its observed response and request an accessible source or a local file.

Download into a verified destination directory, using a temporary filename:

```sh
curl -fL --retry 3 --connect-timeout 10 \
  -o model.gguf.part "$MODEL_URL"
```

Before promoting the file to `model.gguf`, verify that the transfer completed, its size matches source metadata when available, and its first four bytes are `GGUF`. Compare its SHA-256 with a publisher-provided digest when available. An HTML block page is not a model, even if the server returned HTTP 200. If a file already exists, inspect it before replacing or resuming it.

For offline installation of an exact Ollama package, obtain its manifest and every blob referenced by that manifest, including its config blob, from a machine where the package is installed. Use that machine's `OLLAMA_MODELS` setting or default model directory. Preserve the manifest's relative path and blob filenames in the destination model directory, and verify blob digests. A GGUF alone does not carry the package's template, parameters, or capability metadata.

This step is complete when a validated local GGUF or a complete, verified Ollama package is available.

## 3. Import the local GGUF

For a GGUF, create a dedicated `Modelfile` using the verified local path:

```dockerfile
FROM ./model.gguf
```

Add parameters, system instructions, or templates only when supported by the model's documentation or the project's intended use. Set a documented context size rather than inheriting an unnecessarily large architecture maximum. For split GGUF files, follow the installed version's documented import syntax and include every shard.

Run from the directory containing the file and `Modelfile`:

```sh
ollama create model-local -f Modelfile
ollama show model-local
```

An import whose `FROM` points to an existing GGUF uses local weights. If Ollama attempts to pull a base model, inspect `FROM` and path resolution. For an exact package copied offline, use `ollama show` to verify recognition instead of rebuilding it as a generic GGUF import.

This step is complete when Ollama recognizes the intended model and its configuration matches the selected artifact.

## 4. Verify the intended interface

Use a bounded request with a short input. For chat models, a small `/api/generate` request with a token limit is sufficient. For decision models, verify `/v1/systemone` using the contract in [../ollama-decision-models/SKILL.md](../ollama-decision-models/SKILL.md).

Check the actual response, not just model registration. Report chat generation and decision API support separately. A successful GGUF import does not prove that the imported model has the official package's capabilities. If decision support is missing, obtain the official package metadata or a documented configuration and test again.

This step is complete when the requested interface returns a valid response, or a specific compatibility or environment blocker has been captured.

## Deliverable

Report the source and whether it is official or community-maintained, the local artifact path, the imported model name, integrity checks performed, and the result of the intended API request. State any unresolved blocker. When asked only to document the process, provide the commands and mark unexecuted download or import steps as unverified.

## Sources

- Import documentation: https://docs.ollama.com/import
- Model metadata and download links: consult the selected publisher's model repository at execution time.
