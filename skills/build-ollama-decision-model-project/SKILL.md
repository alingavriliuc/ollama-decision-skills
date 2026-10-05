---
name: build-ollama-decision-model-project
description: "Build a project with Ollama decision models through an interactive requirements interview and a tested prototype. Use when the user wants to turn an idea into an application or prototype using Nimble, Tev1, or Ollama decision models."
---

# Build an Ollama decision model project

Guide the user from their initial idea to a runnable prototype that calls a real decision model. Follow the two stages in order. Each stage has a completion criterion.

This skill depends on two companion skills from the same repository: `ollama-decision-models` (API contract) and `ollama-local-model-import` (download and import fallback). If either is unavailable when needed, tell the user to install the full set, for example with `npx skills add alingavriliuc/ollama-decision-skills`, and do not invent the missing contract.

## Conduct the conversation

- Speak the user's language. Explain technical terms when they become relevant.
- Ask one question at a time using the question tool when available. Wait for the answer before continuing the interview. Offer concrete choices when helpful, with a free-text answer option.
- Reuse information already provided and the project context. Ask the question that resolves the most important uncertainty about the expected result.
- If the user is unsure, propose an example based on their idea and ask what needs correcting.
- Distinguish the user's decisions from your proposed assumptions. Confirm assumptions that change the prototype's behavior.
- When resuming, read the existing brief and README. Summarize progress and continue from the first unresolved point.

## 1. Refine the requirements

Start with the user's idea. If none was provided, ask: "What task do you want this project to help with, and who is it for?"

Use their answers to clarify the following:

| Topic | Concrete outcome |
| --- | --- |
| Use case | Who uses the project, in what situation, and what problem it solves. |
| Inputs | The available data, its format, and a representative example. |
| Decision | What the model must choose, assess, or score using that data. |
| Output | What the user sees or receives and how they use it. |
| Rules | The categories, criteria, or levels that define a good decision. |
| Uncertainty | The expected behavior for ambiguous, incomplete, or out-of-scope data. |
| Success | Examples with expected results and an observable criterion for evaluating the prototype. |
| Constraints | The useful interface, technical constraints affecting this first attempt, and features to defer. |

Start with the business need. Discuss technology after identifying the decision and expected result. If a deterministic rule is enough for part of the task, propose keeping it in code and using the model for the decision that requires interpretation.

Obtain at least one typical example with an expected answer, one ambiguous example, and one out-of-scope example. For the latter two, clarify the desired behavior even if no single category is expected.

Then present a short brief:

```markdown
## Requirements
User and problem to solve: ...
Inputs, with an example: ...
Decision to produce and business rules: ...
Visible output and how it is used: ...
Behavior under uncertainty: ...
Examples and expected results: ...
Success criterion for the first attempt: ...
Prototype scope and deferred features: ...
Constraints and assumptions to confirm: ...
```

Ask the user to confirm or correct the brief. Incorporate corrections before moving to stage 2.

**Completion criterion:** the user has approved a brief defining the input, decision, output, and success criterion. Remaining uncertainties do not prevent choosing the first prototype's behavior.

## 2. Propose, prepare, and build the prototype

### 2.1. Propose the smallest useful workflow

Inspect the existing project's conventions and stack. Propose a prototype that accepts an input, calls the model, and displays its decision in the agreed format.

Choose a minimal interface suited to the use case. A script or CLI is enough if a graphical interface is unnecessary to evaluate the idea. For a new project without a technology preference, recommend a simple stack and briefly explain the choice.

Present the demonstration scenario, main files, dependencies, and destination directory. Ask the user to confirm the proposed prototype and the directory if it has not already been established.

**Completion criterion:** the user has accepted a prototype whose workflow and scope allow evaluating the approved requirements.

### 2.2. Verify Ollama before implementation

Load `ollama-decision-models` using the skill tool. If unavailable, read [../ollama-decision-models/SKILL.md](../ollama-decision-models/SKILL.md). It defines model selection, the `/v1/systemone` contract, and response interpretation.

Run checks in the environment where the prototype will call Ollama:

1. Identify any remote server, configured address, and model specified by the project. For local use, check command availability and run `ollama --version`.
2. Check server availability with a bounded request. For the default local server, use `curl --fail-with-body --show-error --connect-timeout 5 --max-time 10 http://localhost:11434/api/version`. Adapt the address and authentication to the actual configuration.
3. Check `ollama list` for the local server or `/api/tags` for the configured server. Identify a suitable decision model following `ollama-decision-models`. An arbitrary chat model does not satisfy this prerequisite.
4. Send a small real request to `/v1/systemone` using the selected model and an example from the brief. Validate the HTTP status, response structure, and answer types according to the dependency skill.

Handle each missing prerequisite where it occurs:

| Observed situation | Next action |
| --- | --- |
| Local Ollama is missing | Offer to install it. After agreement, use the official method for the operating system, then repeat the checks. |
| Local server is stopped | Start the service or `ollama serve` using the available process mechanism, then check that it responds. |
| Version or endpoint is incompatible | Verify the address and the dependency skill's requirements. Offer an update if needed, then retest the endpoint. |
| Suitable model is missing | Offer to download the selected model, stating its known or estimated download size. After agreement, run `ollama pull` for that model. |
| Model download or installation fails, stalls, or stops progressing | Load `ollama-local-model-import`, or read [../ollama-local-model-import/SKILL.md](../ollama-local-model-import/SKILL.md). Follow its diagnosis and import workflow, then retest `/v1/systemone`. |
| Installation of Ollama itself fails | Capture the error and resolve the installation problem. Importing weights requires a working Ollama installation. If obtaining the model subsequently fails, use `ollama-local-model-import`. |
| Installation is declined or execution is unavailable | Explain the missing prerequisite and the next action needed. Stay at this step and resume after the blocker is resolved. |

For downloads, use an execution mechanism that allows checking progress and stopping a stalled attempt. A tool timeout alone does not prove that a progressing download failed.

Reuse existing models and artifacts when suitable. After a local import, retain the verified model name for the prototype's configuration.

**Completion criterion:** the target server responds and the selected model returns a valid response through `/v1/systemone`. A displayed version and a model listing are insufficient. Begin implementation after this verification.

### 2.3. Build the prototype

Create the project in the agreed directory and follow its conventions. Implement the accepted workflow through to a visible output.

Use `ollama-decision-models` to define typed questions, separate data from rules, build the request, and interpret responses. That skill remains the source of truth for the API contract. Keep the server address and model name in configuration rather than scattering them through the code.

Include a timeout, an actionable message when the server is unavailable, and response validation before using answers. Apply the approved business behavior to ambiguous or out-of-scope inputs. Distinguish an uncertain decision from an HTTP failure or invalid response.

Save the approved brief in `docs/decision-project-brief.md`, or the equivalent document already used by the project. Add prerequisites, configuration, exact launch commands, and a usage example to the README. Document the model actually verified and any observed limitations.

**Completion criterion:** the prototype includes the complete workflow, configuration, and instructions the user needs to run it.

### 2.4. Test the result and collect feedback

Run the prototype with the examples from the brief. Compare results with expectations, including ambiguous and out-of-scope cases. Also check its handling of an invalid response and an unavailable server without stopping a shared service. Use targeted simulation for these errors and identify it as such in the report.

Run the project's relevant checks. If a result differs from the brief, identify whether the cause is the code, decision criteria, or an unclear business rule. Correct the code or criteria and rerun the affected cases. Ask the user to confirm changes to business rules.

Report the created files, the command to try the prototype, the model used, and observed results. State the actual status of any blocker or unmet criterion.

Then ask: "Which parts of this first result meet your needs, and what should we adjust?" Use the answer to refine the prototype. If it changes the expected input, decision, or output, update the brief with the user before continuing.

**Completion criterion:** the prototype has run with the real model, the accepted criteria have been verified, and the user has the commands to try it. The feedback question starts the next iteration.
