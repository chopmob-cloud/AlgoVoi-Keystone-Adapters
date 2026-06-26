# AlgoVoi Keystone adapters

Open adapters that carry an AlgoVoi keystone decision into the common agent and observability
standards, so the decision stays verifiable wherever it travels. Each adapter maps the
content addressed `execution_ref` of a keystone decision into a different ecosystem format; a
consumer recomputes `execution_ref` from the carried fields alone with only JCS (RFC 8785) plus
SHA-256, with no issuer contact.

```
execution_ref = "sha256:" + SHA-256(JCS({decision_ref, action_type, scope, outcome, executed_at_ms}))
```

Integer epoch milliseconds are hashed directly. Each adapter ships a Python and a TypeScript
implementation that produce the same `execution_ref` byte for byte.

## The adapters

| Directory | Package | Carries the keystone decision as |
|---|---|---|
| `otel-genai/` | `algovoi-otel-genai` / `@algovoi/otel-genai` | OpenTelemetry `gen_ai.agent.*` attributes, with the keystone refs under the vendor namespace `algovoi.keystone.*` (never inside the controlled `gen_ai.agent.*` namespace) |
| `cloudevents/` | `algovoi-cloudevents` / `@algovoi/cloudevents` | a CloudEvents 1.0 event (the `execution_ref` is the event `id`; refs live in `data`, no invented extension attributes) |
| `keystone-vc/` | `algovoi-keystone-vc` / `@algovoi/keystone-vc` | a W3C Verifiable Credential (Data Model 2.0) of type `KeystoneExecutionCredential`, with `execution_ref` as `credentialSubject.id` |
| `keystone-mcp/` | `algovoi-keystone-mcp` / `@algovoi/keystone-mcp` | MCP tools (`keystone_execution_ref`, `keystone_verify`) so any MCP client recomputes and checks an `execution_ref` offline |

## Design

- **Recomputable, not asserted.** Every adapter carries the keystone fields so the consumer
  recomputes `execution_ref` independently. The format conveys it; it does not vouch for it.
- **Own namespace.** Vendor specific values live under AlgoVoi owned identifiers, never inside a
  namespace governed by another standards body.
- **Open verifier, not the engine.** These adapters carry and verify the content addressed
  reference. The cross party composition engine is not part of them.

Each package directory has its own README, LICENSE, and `python/` plus `typescript/` sources.

## Status

These are reference implementations. They are unpublished; install from source for review.

Apache-2.0.
