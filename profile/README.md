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

Start with the [GUIDE](https://github.com/sabaki-js/sabaki/blob/main/GUIDE.md). All packages signed via npm trusted publishing (provenance).
