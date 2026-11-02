# @lenticule-science/occupation-entropy-readout-science-services-section

<!-- 561-group-standards:badges:start -->
[![Lenticule Science](https://img.shields.io/badge/561%20Group-Lenticule%20Science-5b4b8a)](https://github.com/lenticule-science)
[![License: MIT](https://img.shields.io/badge/license-MIT-5b4b8a)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/lenticule-science/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

occupation-entropy-readout-science-services-section is a services section of the 561 Group.

A signed A2A face for one computation: the Boltzmann occupation-entropy
readout of an occupation list.

Given an occupation list — one non-negative integer count per cell (for
example, hires per provider) — it returns:

- `cells` — the number of cells (the occupation list's length)
- `totalObjects` — `N`, the sum of the occupation counts
- `configH` — Boltzmann's `H = Σ log2(n_i!)` over the cells
- `entropyBits` — `log2(N!) − configH`, the log₂ multinomial multiplicity
  `log2(N! / Π n_i!)`: the spread of the occupation. It is `0` when every
  object sits in one cell, and larger the more evenly the objects are spread
  across cells.

`configH` is computed as an exact log-sum (`Σ_{k=2}^{n} log2(k)` per cell),
not by forming `n!` directly, so it does not overflow for large occupation
counts.

## Spec sheet

This readout is specified by
`Obsmetrologia/Thermodynamics/BoltzmannEntropy.lean`
(`Obsmetrologia.Thermodynamics.fact`, `.cellH`, `.configH`). The Lean file's
bytes are pinned by sha256 in `src/occupation-entropy.mjs`; every served answer
attaches a `certificate` recomputing that digest on every call and refusing
to answer if the pinned bytes have drifted. The JavaScript in this die is
a separate implementation, not an extraction from the Lean.

## A2A face

The die serves one signed A2A 1.0 face (`bin/serve-a2a.mjs`, executor
`bin/execute-a2a-task.mjs`), reachable publicly through the settlement's
existing tunnel:

```
https://bare-cedar-fog.561.group/a2a/occupation-entropy-readout/.well-known/agent-card.json
```

(locally: `http://127.0.0.1:8812/.well-known/agent-card.json`). Request: one
A2A `ROLE_USER` message carrying one `application/rmn+cbor` raw part whose
RMN term is a single Mark labeled with a JSON object:

```
["mark", "{\"occupation\":[2,2]}"]
```

Response: a `ROLE_AGENT` message, `metadata.disposition: "completion"`, one
RMN Mark part whose JSON label carries `cells`, `totalObjects`,
`configH: 2`, `entropyBits: 2.584962500721156` (`log2(6)`), a measured
`cost` block, and the source `certificate`. A malformed occupation is a
refusal (nonzero executor exit), not a completion.

Default local port: `8812` (`PORT` environment variable); `BASE_URL` names
the public face the agent card advertises.

## Code

This repository holds section data only. The executable code that used to live here moved to
[`red-cup-engineering/occupation-entropy-readout`](https://github.com/red-cup-engineering/occupation-entropy-readout) (received 2026-10-24 from
`lenticule-science/occupation-entropy-readout@6d98049e2968`). That engine reads this data from `SECTION_ROOT` (default: its current working directory),
so point `SECTION_ROOT` at a checkout of this repository.
