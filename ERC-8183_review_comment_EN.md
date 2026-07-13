# A constructive review of ERC-8183's trust model: residual human-judgment assumptions

**Posted to:** `ethereum/ERCs#1581` & Ethereum Magicians thread `erc-8183-agentic-commerce`

**Date:** 2026-07-12

First, thank you to the authors (Davide Crapis, Bryan Lim, Tay Weixiong, Chooi Zuhwa) and the dAI team for a clean, minimal escrow primitive. This is **not a rejection** of ERC-8183 — it is a concern about one specific layer, raised for consideration while the EIP is still in Draft.

---

## The concern, in one sentence

ERC-8183's **trust layer** (the "evaluator who alone may mark completed") re-encodes the *human* trust model — judge, reputation, expert score, subjective QA, value-tier risk control — as programmable on-chain primitives, rather than replacing it with a **structurally necessary (mathematical)** criterion.

For agent-to-agent commerce this matters: agents need not "trust *who*," only "whether *structure holds*."

## What I am NOT criticizing

The escrow + state machine (`Open → Funded → Submitted → Terminal`) is a neutral technical mechanism. My concern is **only the trust / decision layer built on top of it** — the part the spec itself calls the trust layer.

---

## Five residual human-judgment points (with spec evidence)

### R1 — Judge-style evaluator ("whoever decides wins")

- **Spec evidence**: Abstract — *"an evaluator who alone may mark the job completed"*. Security Considerations — *"a malicious evaluator can complete or reject arbitrarily"*.
- **The problem**: Trust = picking the right decider. This is a direct on-chain copy of judge / arbiter / QA-inspector from human commerce.
- **Why it conflicts with agent logic**: Agents need no decider *subject* — only whether structure holds. The spec's own security section admits this is an unfixable weakness.

### R2 — Subjective quality judgment ("is this report good?")

- **Evidence**: DecipherClub explicitly states: *"For subjective tasks — evaluation requires judgment that current on-chain mechanisms cannot reliably provide."*
- **The problem**: For writing, design, analysis etc., the loop closes on *human taste*, not on mathematical fact.
- **Conflict**: Agent-native trust should be structural verifiability, not "did a human-like agent like this."

### R3 — Reputation as trust basis

- **Evidence**: `ReputationHook` writes job outcomes to ERC-8004 ReputationRegistry. TryEthernal: *"The evaluating agent's own ERC-8004 reputation becomes the trust basis."*
- **The problem**: Reputation = gossip / social proof / brand loyalty, copied on-chain. It can be gamed, colluded, and is empirical not necessary.
- **Conflict**: Agents only need verifiable on-chain structural facts (e.g. conservation-pass rate), not social proof.

### R4 — Expert risk-score gatekeeper

- **Evidence**: ERC-8126 (Final) outputs a 0–100 *empirical* risk score used as evaluator pre-filter.
- **The problem**: Trust delegated to authority's *opinion*, not mathematical necessity.
- **Conflict**: Transaction-level trust should be independent of subject-level safety scores. A "score 90" agent with broken structure = untrustworthy; a "score 5" agent with valid structure = trustworthy *for that transaction*.

### R5 — Value-tier risk control

- **Evidence**: Security Considerations — *"Use reputation or staking for high-value jobs."*
- **The problem**: Human risk-management intuition (big money = more scrutiny, small = exempt).
- **Conflict**: Conservation laws are universal and amount-independent. A $0.5 digital-good transaction needs the same structural verification as a $50k physical-goods transaction.

---

## Two gaps the spec itself admits

1. **"Who is the evaluator?" is left open.** DecipherClub: *"The Evaluator is the linchpin of the entire ERC-8183 economy. And it remains an open design space."* — The hardest human decision ("who do we trust to decide?") is pushed to implementers because the spec cannot solve it mathematically.

2. **No built-in arbitration.** Security: *"No dispute resolution or arbitration; reject/expire is final."* — Implicitly falls back on off-chain human courts (UMA DVM, Kleros). The trust anchor remains in human society.

---

## To be fair: the mathematical space already exists

The spec explicitly permits a **smart-contract evaluator** for deterministic tasks (TryEthernal: *"No human judgment required"* for ZK-proof / hash-comparison / on-chain-test-harness outcomes). That part is genuinely agent-native.

**But three limitations remain:**

1. Subjective-task evaluation and reputation are still the **default path**; contract evaluators are just one option.
2. Subjective tasks (writing, design, analysis — a large share of agent commerce) the spec admits *"cannot reliably be provided"* by on-chain mechanisms.
3. Even with contract evaluators, trust is still anchored in **"pick the right evaluator"**, not in **"structure necessarily holds."**

**Conclusion:** ERC-8183 is a hybrid standard — it gives a mathematical interface but defaults to human-experience trust. It is best described as "**programmable encapsulation of human trust models,**" not a pure agent-native trust standard.

---

## A constructive suggestion

For the trust layer, consider encouraging or referencing an **objective verification interface** as the default — where "completion" means *"satisfies a verifiable invariant"* rather than *"an address called `complete()`."*

Concretely:

- Define completion as a checkable structural condition (e.g. a conservation / state-coupling invariant) that **any** evaluator must verify, instead of free-form `complete()` / `reject()`.
- Keep subjective / reputation-based paths as **opt-in hooks**, not the implied default.

This is the direction our Feynman-diagram model takes: **transaction trust = dNFT conservation Σ(dNFT)=0**, verified by contract — no decider subject, no reputation, no amount-tiering.

---

## Full analysis

Complete critique (all five points with extended evidence, Feynman-diagram contrast table, implications for the standard):

**https://gist.github.com/YOUR_GIST_ID_HERE**

(Chinese full text available at the link above; English translation can be provided on request.)

---

*Not an attack on the authors' excellent work — a request that the Draft stage consider whether the "trust layer" can be made **structurally necessary** rather than **socially delegated**.*
