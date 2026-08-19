# Experiment: What happens when agents have fewer rounds to find a deal?

**Scenario:** `marketplace` (built-in)  
**Setting changed:** `task.config.rounds` from `10` → `3`  
**Branch:** `aistudio/niki-papadopoulos-rounds`

---

## Why I chose this setting

The marketplace scenario uses `alternating_offers` as its negotiation layer and `contract_net` for coordination. `rounds` sits inside `task.config` and controls how many attempts each buyer gets to find and negotiate with a seller before the simulation ends.

I was curious whether `rounds` controls the depth of a single negotiation (back-and-forth exchanges per deal) or the breadth of a buyer's search (how many different sellers they can approach). The distinction matters for understanding how exploration budgets affect multi-agent market outcomes.

---

## Hypothesis

Cutting rounds from 10 to 3 would reduce deal volume, since buyers have fewer chances to find a match. I expected roughly a 30-50% drop in sales and a proportional drop in message count, with the protocol itself staying clean (all validators passing).

---

## Results

| Metric | Baseline (rounds: 10) | Experiment (rounds: 3) | Change |
|---|---|---|---|
| Total messages | 2,000 | 600 | -70% |
| Correlation IDs | 1,000 | 300 | -70% |
| Sales | 266 | 76 | -71% |
| Requests answered | 500 | 150 | -70% |
| Top seller sends | 15 | 7 | -53% |
| Validators | 3/3 PASS | 3/3 PASS | - |

Validator output (experiment run):

PASS marketplace_no_double_sell - checked 76 sales
PASS marketplace_all_responded - all 150 requests answered
PASS marketplace_price_agreement -

---

## What was surprising

The drop across every metric was almost exactly proportional to the round reduction (10 to 3 = 70% reduction, and every metric dropped ~70%). That precision revealed something I had not expected: `rounds` is not a negotiation depth parameter. It controls how many sellers each buyer can approach in total. It is an exploration budget.

This means the `alternating_offers` negotiation layer converges in far fewer than 10 exchanges when it does converge. Cutting rounds did not break any negotiations already in progress, it simply prevented buyers from initiating new ones. The protocol stayed completely clean across both runs.

The practical implication for agentic system design: in a multi-agent marketplace, restricting exploration budget collapses throughput more than restricting negotiation depth. Giving agents more counterparties to reach matters more than giving them more time per negotiation.

---

## How to reproduce

pip install "nest-core[plugins]" --break-system-packages
nest run rounds-experiment.yaml
nest inspect ./traces/rounds_experiment.jsonl

---

## AI tools used

Claude (Anthropic) was used throughout to understand the NANDA Town architecture, interpret trace outputs, and identify what the proportional drop across metrics actually meant. All experiment design choices, the hypothesis, and the interpretation of surprising results were my own. Claude helped me articulate what I was seeing, not decide what to look for.
