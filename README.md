# Ollama decision model skills

Agent skills for building projects with Ollama's decision models (`nimble`, `tev1`, `tev1:0.8b`) through the `/v1/systemone` endpoint.

| Skill                                                                                        | What it does                                                                                                                           |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| [`build-ollama-decision-model-project`](skills/build-ollama-decision-model-project/SKILL.md) | Interviews you about your idea, writes a requirements brief, then builds and tests a runnable prototype against a real decision model. |
| [`ollama-decision-models`](skills/ollama-decision-models/SKILL.md)                           | The API contract: typed questions (`choice`, `noul`, `score`), request building, and response validation.                              |
| [`ollama-local-model-import`](skills/ollama-local-model-import/SKILL.md)                     | Diagnoses stalled `ollama pull` downloads and imports a local GGUF or offline package.                                                 |

## Install

Install all three together. `build-ollama-decision-model-project` depends on the other two, and the other two reference each other.

```sh
npx skills add alingavriliuc/ollama-decision-skills
```

## Requirements

- [Ollama](https://ollama.com/download) 0.35 or later
- A decision model, for example `ollama pull nimble`

## Usage

Ask your agent something like:

> I want to build a tool that triages support tickets with an Ollama decision model.

The agent loads `build-ollama-decision-model-project`, asks one question at a time, and pulls in the other skills when it needs them.

## Notes

- `ollama-local-model-import` mentions a community GGUF conversion of Tev1 0.8B. It is maintained by a third party, and the skill asks for your agreement before downloading it.
- The API details come from Ollama's announcement of September 29, 2026: https://ollama.com/blog/ollama-now-supports-jev-style-decision-models

## License

[MIT](LICENSE)
