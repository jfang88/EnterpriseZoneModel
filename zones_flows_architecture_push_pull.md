# High-Level Security Architecture: Swim-Lane Layout Update

## Diagram layout standard
All high-level diagrams should be rendered with vertical swim lanes arranged from left to right by trust level and exposure profile.

Recommended left-to-right ordering:
1. Untrusted and external parties
2. Edge and mediation zones
3. Business enablement internal zones
4. Infrastructure and system management zones
5. Crown-jewel restricted zones
6. Recovery and controlled support adjacencies where needed

The preferred visual pattern is:
- each lane is a vertical subgraph or column,
- zones are shown as boxes within the appropriate lane,
- flows are drawn across lanes using directional arrows,
- key use cases should be overlaid on the same lane structure where practical,
- diagrams should prioritize trust boundaries, approved crossings, and flow direction over physical topology.

## Swim-lane zone grouping
Use the following swim lanes for the architecture package.

| Swim lane | Purpose | Typical zones |
|---|---|---|
| L1-UNTRUSTED | External and untrusted actors | EXT-PUB, EXT-BRK, vendors as external actors |
| L2-EDGE | Internet and broker mediation | EDGE-PUB, EDGE-BRK |
| L3-BUSINESS | Internal business enablement and read-only consumption | SVC-PUB, SVC-TRD, TRD-OPS-WS, TRD-READOUT, CORP-RTDATA, CORP-APP, XFER-CDG, INT-3P |
| L4-INFRA | Identity, admin, observability, deployment, enterprise management | TRD-ID, MGT-TRD, MGT-CORP, OBS-SEC, DEV-REL, ID-SVC, CORP-USER |
| L5-CROWN | Crown-jewel and highest trust workload zone | TRD-CORE |
| L6-RECOVERY | Recovery and resilience adjacencies | RCV-VAULT, DR-ALT |

## Swim-lane design rules
- Keep lanes fixed left to right in every high-level diagram.
- Put the most exposed zones on the left and the most trusted or most restricted zones on the right.
- Keep TRD-CORE in its own dedicated right-side lane.
- Show TRD-DEPLOY and TRD-ID adjacent to TRD-CORE but outside the same lane if they are modeled as support zones rather than as part of the crown-jewel runtime.
- Show read-only extract zones such as TRD-READOUT and CORP-RTDATA to the left of TRD-CORE to reinforce the push-out pattern.
- Show management, deployment, identity, and observability in their own infrastructure lane so they are visually distinct from business use cases.
- When one diagram focuses on a specific use case, preserve the same lane order even if some lanes are sparsely populated.

## Required update to flow representation
For the main architecture and use-case diagrams:
- near-real-time and non-real-time extract flows should traverse TRD-CORE -> TRD-READOUT -> CORP-RTDATA or XFER-CDG -> CORP-APP,
- trading operations access should traverse TRD-OPS-WS -> TRD-ID and TRD-OPS-WS -> SVC-TRD -> TRD-CORE,
- trusted deployment should traverse DEV-REL -> TRD-DEPLOY and TRD-CORE pulling from TRD-DEPLOY,
- observability should traverse TRD-CORE -> TRD-READOUT -> OBS-SEC,
- recovery should traverse TRD-CORE -> RCV-VAULT and exceptional restore via MGT-TRD.

## Prompting note
The prompt package should explicitly instruct any AI or reviewer to preserve the left-to-right swim-lane structure across all Mermaid diagrams and to place zones as boxes within those lanes rather than using free-form layouts.
