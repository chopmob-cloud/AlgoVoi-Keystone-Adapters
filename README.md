# AlgoVoi Keystone adapters and plugins

Open integrations that carry an AlgoVoi keystone decision into common agent and observability
standards, and onto the agent frameworks agents are actually built with. Each one maps the
content-addressed `execution_ref` of a keystone decision into a different ecosystem format; a
consumer recomputes it from the carried fields alone with only JCS (RFC 8785) plus SHA-256, with
no issuer contact.

```
execution_ref = "sha256:" + SHA-256(JCS({decision_ref, action_type, scope, outcome, executed_at_ms}))
```

Integer epoch milliseconds are hashed directly. Every package ships as a `py3-none-any` wheel and
sdist, with no compiled extensions.

## The adapters

Standards adapters: they carry a keystone decision into a foreign format.

| Directory | Package | Carries the keystone decision as |
|---|---|---|
| `otel-genai/` | `algovoi-otel-genai` / `@algovoi/otel-genai` | OpenTelemetry `gen_ai.agent.*` attributes, with the keystone refs under the vendor namespace `algovoi.keystone.*` (never inside the controlled `gen_ai.agent.*` namespace) |
| `cloudevents/` | `algovoi-cloudevents` / `@algovoi/cloudevents` | a CloudEvents 1.0 event (the `execution_ref` is the event `id`; refs live in `data`, no invented extension attributes) |
| `keystone-vc/` | `algovoi-keystone-vc` / `@algovoi/keystone-vc` | a W3C Verifiable Credential (Data Model 2.0) of type `KeystoneExecutionCredential`, with `execution_ref` as `credentialSubject.id` |
| `keystone-mcp/` | `algovoi-keystone-mcp` / `@algovoi/keystone-mcp` | MCP tools (`keystone_execution_ref`, `keystone_verify`) so any MCP client recomputes and checks an `execution_ref` offline |

## The plugins live in the Keystone SDK

The agent-framework plugins moved to
[`chopmob-cloud/algovoi-keystone`](https://github.com/chopmob-cloud/algovoi-keystone/tree/main/plugins)
so that all Keystone code has one home. This repository is now adapters only.

They stamp a keystone decision onto the trace metadata a framework already collects, and expose the
same stamp-then-verify-by-recomputation contract these adapters do:

| Plugin | Stamps onto |
|---|---|
| `algovoi-keystone-langchain` | LangChain callback handler config and run metadata |
| `algovoi-keystone-crewai` | CrewAI step and task metadata |
| `algovoi-keystone-autogen` | AutoGen message metadata |
| `algovoi-keystone-adk` | Google ADK agent state updates and before-agent callbacks |

```
pip install --index-url https://pip.algovoi.co.uk/simple/ algovoi-keystone-langchain
```

### Not open, not included

The `*-journey` packages (`a2a-journey`, `autogen-journey`, `crewai-journey`,
`langgraph-journey`) are **commercial, bundle-only** and live in neither repository.
`crewai-journey` carries no licence field in its metadata at all; it is commercial by intent and
must not be treated as open. They bind a whole multi-agent run into one `journey_ref` — the
orchestrator capability — rather than a single step.

## Install

Every package is published to the AlgoVoi open package index, which serves `#sha256=` fragments so
installs can be pinned:

```
pip install --index-url https://pip.algovoi.co.uk/simple/ algovoi-cloudevents
pip install --index-url https://pip.algovoi.co.uk/simple/ algovoi-keystone-langchain
```

Adapters are also on PyPI. Requires Python 3.9+, with `algovoi-substrate` and `rfc8785` as the only
dependencies.

## Verify it yourself

```
pip install pytest
pytest otel-genai/python/tests      # adapter tamper-evidence
```

The suite is adversarial, not a smoke test. For every adapter it proves a genuine payload
verifies, a tampered preimage is refused, empty or malformed input fails closed, and a valid
`execution_ref` **copied from a different span** does not authenticate this one. The plugin
contract tests live with the plugins in the Keystone SDK repository.

## Design

- **Recomputable, not asserted.** Every adapter and plugin carries the keystone fields so the
  consumer recomputes `execution_ref` independently. The format conveys it; it does not vouch for it.
- **Own namespace.** Vendor-specific values live under AlgoVoi-owned identifiers, never inside a
  namespace governed by another standards body.
- **Open verifier, not the engine.** These packages carry and verify the content-addressed
  reference. The cross-party composition engine is not part of them.
- **Fail closed.** Missing, malformed or tampered input returns `False` rather than raising or
  assuming success.

Each package directory has its own README, LICENSE, and `python/` sources; the adapters also carry
TypeScript twins that produce the same `execution_ref` byte for byte.

## Licence

Apache-2.0. See [LICENSE](./LICENSE) and each package's `NOTICE`.
