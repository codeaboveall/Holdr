<p align="center">
  <img src="./assets/holdr.png" alt="Holdr" width="420"/>
</p>

# Holdr

Holdr is a continuous on-chain holder ranking system designed to measure, score, and reward sustained ownership behavior over time.

Holdr is not a snapshot system.
Holdr is not a leaderboard frozen at arbitrary blocks.
Holdr is not a meme mechanic.

Holdr treats holding as a first-class, measurable on-chain behavior.

The system continuously evaluates wallet state, balance persistence, time-weighted exposure, and participation durability, then routes protocol incentives based on sustained position rather than momentary timing.

The core principle is simple:

Capital that stays should be treated differently than capital that passes through.

---

## Motivation

Most on-chain incentive systems rely on snapshots, checkpoints, or discrete epochs.

These approaches create several structural problems:

- Rewards favor timing over commitment
- Short-term capital can dominate long-term participants
- Systems are easily gamed by entry and exit coordination
- Incentives are misaligned with protocol health

Holdr exists to replace discrete reward logic with continuous evaluation.

Instead of asking who held at a specific time, Holdr asks:

Who stayed.
Who absorbed volatility.
Who maintained exposure when it mattered.

---

## Design Goals

- Continuous evaluation instead of snapshots
- Time-weighted holding as a primitive
- Deterministic and reproducible scoring
- Composable architecture
- Chain-native enforcement
- No admin intervention in rankings
- Economic penalties instead of bans

Holdr is designed to be embedded, queried, and extended by other systems.

---

## Core Concepts

### Holder State

Each wallet interacting with the token tracked by Holdr maintains a rolling holder state.

A holder state includes:

- Current balance
- Historical balance curve
- Entry timestamp
- Exit events
- Net exposure time
- Volatility-adjusted holding duration

This state is never reset unless the balance fully exits.

Partial exits decay score rather than zeroing it.

### Continuous Time Weighting

Time is treated as a continuous variable.

Score accumulation occurs as an integral over time, not as a step function.

Conceptually:

score = ∫ f(balance(t), duration(t), volatility(t)) dt

This prevents sudden spikes from dominating rank.

---

## Ranking Model

A simplified representation of the Holdr scoring function:

```ts
function computeScore(state: HolderState, now: number): number {
  const duration = now - state.firstEntry
  const effectiveBalance = averageWeightedBalance(state.balanceHistory)
  const volatilityPenalty = computeVolatilityPenalty(state)
  return effectiveBalance * duration * volatilityPenalty
}
```

This model is extensible.

Alternative scoring models may include:

- Drawdown tolerance
- LP participation
- Fee reinvestment
- Governance interaction

Holdr treats ranking models as interchangeable modules.

---

## Continuous Recalculation

Holdr does not operate in epochs.

Rankings are recalculated whenever relevant state changes occur.

Triggers include:

- Balance updates
- Partial exits
- Re-entries
- Liquidity changes
- Fee routing events

This ensures rankings always reflect current reality.

---

## Reward Routing

Holdr itself does not mint rewards.

Instead, Holdr acts as a signal layer.

Downstream systems route incentives based on rank output.

Example integration with a fee distribution system:

```ts
const ranked = holdr.getTopHolders(100)

for (const holder of ranked) {
  routeFees(holder.address, holder.weight)
}
```

This separation keeps Holdr neutral and composable.

---

## Architecture Overview

```
apps/
  dashboard        Frontend visualization

services/
  indexer          On-chain state tracker

contracts/
  core             Ranking primitives

packages/
  sdk              Client interface
  types            Shared structures
```

Each layer can be deployed independently.

---

## Indexer Service

The indexer listens to on-chain events and maintains holder state.

Responsibilities:

- Track balance changes
- Update holder curves
- Compute rolling metrics
- Persist ranking state

Example event handler:

```ts
onTransfer(event) {
  updateBalance(event.from)
  updateBalance(event.to)
  recomputeRanks()
}
```

The indexer is deterministic and replayable.

---

## SDK

The Holdr SDK exposes read-only interfaces.

Example usage:

```ts
import { HoldrClient } from "@holdr/sdk"

const holdr = new HoldrClient(RPC_URL)

const top = await holdr.getTopHolders(50)
```

No private keys.
No signing.
No custody.

---

## Composability

Holdr is designed to be composed.

Possible integrations:

- Fee routers
- Dividend systems
- Governance weighting
- LP incentives
- Reputation layers

Holdr does not assume how rewards are paid.

It only answers who deserves them based on sustained behavior.

---

## Anti-Gaming Properties

Holdr resists common manipulation strategies:

- Snapshot farming is impossible
- Short-term capital has diminishing returns
- Wash trading does not increase score
- Exit penalties are implicit via decay

Economic incentives enforce behavior.

No blacklists.
No bans.
No manual overrides.

---

## Extending Holdr

Developers can define custom scoring modules:

```ts
export function customScore(state: HolderState): number {
  return state.duration * state.reinvestedFees
}
```

Modules can be registered at the protocol layer or off-chain.

---

## Philosophy

Holdr is built around a simple belief:

Holding is not passive.
Holding is behavior.

Behavior can be measured.
Behavior can be ranked.
Behavior can be rewarded.

Holdr formalizes what holders already intuitively understand.

Time matters.
Staying matters.
Conviction matters.

---

## Status

Holdr is experimental infrastructure.

Interfaces may evolve.
Scoring models may change.
Integrations may vary.

The core invariant remains constant.

Continuous ranking of sustained holders.

---

## License

MIT