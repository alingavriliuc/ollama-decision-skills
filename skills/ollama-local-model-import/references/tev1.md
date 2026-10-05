# Tev1 0.8B reference

## Download sources

Official weights are published in Safetensors format:
https://huggingface.co/togethercomputer/Tev1-0.8B-experimental

At the time of writing, a community GGUF conversion provided `tev1-Q8_0.gguf`:
https://huggingface.co/DreamBlooms/Tev1-0.8B-experimental-GGUF/tree/main

This repository is maintained by a third party, not by Together AI or Ollama. Before downloading from it:

1. Check that the repository still exists, still targets the same base model, and is owned by the same publisher.
2. Retrieve the current file list, file size, and digest from the repository.
3. Tell the user the artifact is a community conversion and obtain their agreement before downloading it.

If those checks fail or the user declines, look for another source or ask for a local file. Do not substitute a different community repository without telling the user.

The Ollama package uses a Q8_0 model with 752M parameters and a model layer of 811,843,424 bytes. The community artifact also uses Q8_0; equivalence of its bytes and package behavior has not been verified.

## Known failure: proxy blocks the model file host

`ollama pull` and `ollama run` first fetch a small registry manifest, then follow a redirect to a separate file host for the model layers. A corporate proxy or web filter can allow the manifest but block the file host, for example by classifying it as file sharing.

Typical symptoms:

- The download starts, then stalls or restarts repeatedly.
- `server.log` shows `failed: EOF` followed by retries with increasing delays.
- A `curl` request to the redirected file URL returns HTTP 403 or an HTML warning page instead of binary data.

This is a network access problem, not slow inference. Confirm it by reproducing the file request as described in the main skill before applying this diagnosis. The fix is either access to the file host, approved by whoever manages the network, or a local import from a reachable source.

## Local import example

A minimal `Modelfile` for the community GGUF placed in the same directory:

```dockerfile
FROM ./tev1-Q8_0.gguf
PARAMETER num_ctx 2048
```

If a `Modelfile` already exists, inspect it before reusing or replacing it. Once the artifact has been validated:

```sh
ollama create tev1-local:0.8b -f Modelfile
ollama show tev1-local:0.8b
```

## Decision API verification

Tev1 requires Ollama 0.35 or later. Its documented decision interface is `/v1/systemone`. Regular chat does not exercise the decision contract. A minimal verification request is:

```sh
curl --fail-with-body --show-error \
  --connect-timeout 5 --max-time 120 \
  http://localhost:11434/v1/systemone \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "tev1-local:0.8b",
    "state": "Hello world",
    "questions": {
      "greeting": {
        "type": "noul",
        "instructions": "Does the text contain a greeting?"
      }
    }
  }'
```

Check that `answers.greeting.noul` is a numeric probability between 0 and 1. A minimal GGUF import is not guaranteed to support this endpoint. Preserve and test the official package metadata if the local import lacks the required capabilities.

Official package documentation:
https://ollama.com/library/tev1:0.8b
