<img src="https://raw.githubusercontent.com/sabaki-js/sabaki/main/assets/logo.svg" width="120" alt="sabaki logo" />

# sabaki-js

Lightweight **typed-decision** AI for Node.js: route states to tiny classifiers that answer with calibrated probabilities — milliseconds on CPU, zero API.

```ts
import { Sabaki } from "sabaki";
const r = sabaki.decide({ state: "Refund not received, cancel my plan" });
// { domain: "customer_service", decision: "billing", confidence: 0.93 }
```

## Packages

| package | what |
|---|---|
| [sabaki](https://github.com/sabaki-js/sabaki) | router core (`nano`/`mini`/`base`) |
| [@sabaki-js/customer-service](https://github.com/sabaki-js/plugin-customer-service) | tickets (category 0.900) |
| [@sabaki-js/invoice-processing](https://github.com/sabaki-js/plugin-invoice-processing) | invoices (duplicate 0.89) |
| [@sabaki-js/security-incidents](https://github.com/sabaki-js/plugin-security-incidents) | triage (disposition 0.73) |
| [@sabaki-js/agent-trace-observability](https://github.com/sabaki-js/plugin-agent-trace-observability) | agent traces |
| [@sabaki-js/mail](https://github.com/sabaki-js/plugin-mail) | email triage (8 buckets, 0.889) |
| [@sabaki-js/doom](https://github.com/sabaki-js/plugin-doom) | Doom decisions (WIP) |
| [@sabaki-js/plugin-template](https://github.com/sabaki-js/plugin-template) | starter template |



## HTTP server (Jev-compatible `/v1/systemone`)

`sabaki serve` answers typed questions over HTTP — same shape as TypeSafe Jev,
backed by mini ONNX models with nano fallback, lazy-loaded per domain:

```sh
curl localhost:8001/v1/systemone -H 'content-type: application/json' -d '{
  "state": "Charged twice, refund the duplicate today.",
  "questions": {
    "department": {"type": "choice", "instruct": "Which team?",
      "criteria": {"billing": "payments, refunds", "technical": "bugs"}},
    "urgency": {"type": "score", "criteria": ["low", "soon", "blocking"]},
    "churn_risk": {"type": "noul", "instruct": "Threatens to leave?"}
  }}'
```

You define the questions **and** the answer types: `choice` (options),
`score` (ordered levels), `noul` (yes/no probability). New schemas need no
retraining — just different questions, like Laya's option scorer.
The server answers any question name with the plugin's primary question
(Jev-style: ask `department`, get the `category` decision).

### Automatic classification (act on confidence)

```ts
import { Sabaki } from "sabaki";
const r = await sabaki.decideAsync({ state: ticket });
if (r.domain !== "unknown" && r.confidence > 0.8) {
  await autoRoute(r); // calibrated confidences make this safe
} else {
  await escalateToHuman(r); // unsure cases go to a person
}
```

## Benchmarks (typed-decisions, test split)

Our `nano` plugins vs published reference scores (same dataset, all questions):

| system | score | params | note |
|---|---|---|---|
| OpenDecider-nano | 0.796 | 400M | fine-tuned, published |
| TypeSafe Jev | 0.754 | API | zero-shot, published |
| Laya typed-decisions | 0.766 | 421M | fine-tuned, published |
| **sabaki nano** | **0.611** | ~0 | mean over 4 workflows (0.686 / 0.650 / 0.686 / 0.422) |
| Laya base | 0.362 | 421M | zero-shot, published |


### `mini` (MiniLM ONNX int8, ~22 MB) vs published references

| workflow | sabaki mini | OpenDecider-nano | Laya-td | Jev |
|---|---|---|---|---|
| customer_service | 0.670 | — | — | — |
| invoice_processing | 0.650 | — | — | — |
| security_incidents | 0.720 | — | — | — |
| agent_trace | 0.710 | — | — | — |
| **mean** | **0.688** | 0.796 | 0.766 | 0.754 |

Calibrated (ECE < 0.14, temperature-scaled) with healthy doubt on
out-of-distribution inputs — the property that makes confident automation safe.

Honest reading: our kilobyte models sit between Laya-base and the fine-tuned
400M+ models — expected for word-count linear models. The `mini` (MiniLM)
models narrow the gap (mean ~0.69, ECE < 0.14). See GUIDE for methodology.

Start with the [GUIDE](https://github.com/sabaki-js/sabaki/blob/main/GUIDE.md). All packages signed via npm trusted publishing (provenance).
