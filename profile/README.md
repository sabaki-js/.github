<img src="https://raw.githubusercontent.com/sabaki-js/sabaki/main/assets/logo.svg" width="120" alt="sabaki logo" />

# sabaki-js

**Typed decisions on CPU, in milliseconds — Jev-compatible, plugin-routed, open.**

Sabaki routes an unstructured state to the right tiny classifier and returns a **typed answer** (`choice` / `score` / `noul`) with calibrated confidence — over an HTTP API that speaks the [System One](https://docs.typesafe.ai/concepts/system-one) contract. No GPU, no API key, no per-call cost.

```ts
import { Sabaki } from "sabaki";
import mail from "@sabaki-js/mail";

const sabaki = new Sabaki().register(mail).setFallback(generalist);
await sabaki.decideAsync({ state: "Invoice charged twice, need a refund", question: "bucket" });
// { decision: "billing", confidence: 0.89, domain: "mail", plugin: "@sabaki-js/mail" }
```

Live: `https://sabaki.fox3000foxy.com/v1/systemone` — try the [GUIDE](https://github.com/sabaki-js/sabaki/blob/main/GUIDE.md).

## Packages

| package | what |
|---|---|
| [sabaki](https://github.com/sabaki-js/sabaki) | router core + generalist fallback + Jev shapes |
| [@sabaki-js/customer-service](https://github.com/sabaki-js/plugin-customer-service) | tickets (category 0.900 nano) |
| [@sabaki-js/invoice-processing](https://github.com/sabaki-js/plugin-invoice-processing) | invoices (duplicate 0.89) |
| [@sabaki-js/security-incidents](https://github.com/sabaki-js/plugin-security-incidents) | triage (disposition 0.73) |
| [@sabaki-js/agent-trace-observability](https://github.com/sabaki-js/plugin-agent-trace-observability) | agent traces |
| [@sabaki-js/mail](https://github.com/sabaki-js/plugin-mail) | email triage (8 buckets, 0.889) |
| [@sabaki-js/doom](https://github.com/sabaki-js/plugin-doom) | Doom decisions (MAP01 exits on video) |
| [@sabaki-js/generalist](https://github.com/sabaki-js/workspace) | open questions (cross-encoder, in core) |
| [@sabaki-js/plugin-template](https://github.com/sabaki-js/plugin-template) | starter template |

## How it decides

```mermaid
flowchart LR
    A[state + questions] --> B{route()\nkeyword score\n>= threshold?}
    B -- specialist wins --> C[nano <1ms\nor mini ~20ms]
    B -- unknown --> D[generalist fallback\nruntime-defined questions]
    C --> E["Jev answers\nchoice / score / noul"]
    D --> E
    E --> F["/v1/systemone\n{model, answers, usage}"]
```

Specialists answer their trained questions; the generalist answers **anything** (`instructions` + `criteria`, truly read by the model). The `model` field always names the actual responder.

## Benchmarks (typed-decisions, test splits)

| system | score | params | CPU | note |
|---|---|---|---|---|
| OpenDecider-nano | 0.796 | 400M | GPU | fine-tuned, published |
| Laya typed-decisions | 0.766 | 421M | ~25 s/state ❌ | fine-tuned, published |
| TypeSafe Jev | 0.754 | API | API only | zero-shot, published |
| **sabaki mini** | **0.69** | 22M int8 | **~20 ms** | ECE < 0.14, healthy OOD doubt |
| **sabaki nano** | **0.61** | ~0 | **<1 ms** | zero deps, ~18k req/s |
| **sabaki generalist** | 0.56 | 22M | ~20-36 ms | ep0, training in progress |
| Laya base | 0.362 | 421M | GPU | zero-shot, published |

A few points less for **1000× lower latency**: nano ~18 000 req/s, mini ~71 req/s (1 thread) — a 35 Hz game tick fits on any CPU. See [@sabaki-js/doom](https://github.com/sabaki-js/plugin-doom): the same stack exits MAP01 end-to-end (video in repo).

Mail bucket8: nano **0.865** anonymized (0 PII). All packages via npm trusted publishing (provenance). MIT.
