# Mermaid Diagrams: Vertical Swim Lanes Left to Right by Trust

## High-level zone architecture in swim lanes

```mermaid
flowchart LR
    subgraph L1[Untrusted / External]
        direction TB
        EXT_PUB[EXT-PUB
Public Internet]
        EXT_BRK[EXT-BRK
Brokers / Participants]
        VEND[External Vendors]
    end

    subgraph L2[Edge / Mediation]
        direction TB
        EDGE_PUB[EDGE-PUB
Public Edge]
        EDGE_BRK[EDGE-BRK
Broker Edge]
    end

    subgraph L3[Business Enablement]
        direction TB
        SVC_PUB[SVC-PUB
Public App Services]
        SVC_TRD[SVC-TRD
Trading Support / GUIs]
        TRD_OPS[TRD-OPS-WS
Trading Workstations]
        TRD_READ[TRD-READOUT
Read-only Readout]
        CORP_RT[CORP-RTDATA
Corp Data Consumption]
        CORP_APP[CORP-APP
Corporate Apps]
        XFER[XFER-CDG
Transfer Gateway]
        INT3P[INT-3P
Partners / Regulators]
    end

    subgraph L4[Infrastructure / System Management]
        direction TB
        TRD_ID[TRD-ID
Trading Directory]
        MGT_TRD[MGT-TRD
Trading Management]
        MGT_CORP[MGT-CORP
Corporate Management]
        OBS[OBS-SEC
SIEM / Observability]
        DEV[DEV-REL
Build / Release]
        IDSVC[ID-SVC
Shared Identity / PKI]
        CORP_USER[CORP-USER
Corporate Desktops]
        TRD_DEP[TRD-DEPLOY
Trusted Deploy]
    end

    subgraph L5[Crown Jewels]
        direction TB
        TRD_CORE[TRD-CORE
Crown Jewel Trading]
    end

    subgraph L6[Recovery / Resilience]
        direction TB
        VAULT[RCV-VAULT
Backup / Recovery]
        DR[DR-ALT
Recovery Site]
    end

    EXT_PUB --> EDGE_PUB
    EXT_BRK --> EDGE_BRK
    EDGE_PUB --> SVC_PUB
    EDGE_BRK --> SVC_TRD
    TRD_OPS --> TRD_ID
    TRD_OPS --> SVC_TRD
    SVC_PUB --> CORP_APP
    CORP_USER --> CORP_APP
    MGT_CORP --> CORP_APP
    MGT_CORP --> CORP_RT
    TRD_ID --> SVC_TRD
    SVC_TRD --> TRD_CORE
    MGT_TRD --> TRD_CORE
    MGT_TRD --> TRD_ID
    MGT_TRD --> TRD_DEP
    TRD_CORE --> TRD_READ
    TRD_READ --> CORP_RT
    TRD_READ --> XFER
    TRD_READ --> OBS
    CORP_RT --> CORP_APP
    XFER --> CORP_APP
    CORP_APP --> INT3P
    CORP_APP --> EDGE_PUB
    DEV --> TRD_DEP
    TRD_CORE -. pull approved content .-> TRD_DEP
    TRD_CORE --> IDSVC
    TRD_CORE --> VAULT
    TRD_CORE --> DR
    VEND --> MGT_TRD
```

## Trading operations access on same swim lanes

```mermaid
flowchart LR
    subgraph L1[Untrusted / External]
        direction TB
        EMPTY1[ ]
    end

    subgraph L2[Edge / Mediation]
        direction TB
        EMPTY2[ ]
    end

    subgraph L3[Business Enablement]
        direction TB
        OPSWS[TRD-OPS-WS
Trading Workstations]
        ST[ SVC-TRD
Trading GUIs / Services ]
    end

    subgraph L4[Infrastructure / System Management]
        direction TB
        TID[TRD-ID
Trading Directory]
        MTRD[MGT-TRD
Trading Management]
    end

    subgraph L5[Crown Jewels]
        direction TB
        CORE[TRD-CORE
Trading Core]
    end

    subgraph L6[Recovery / Resilience]
        direction TB
        EMPTY6[ ]
    end

    OPSWS --> TID
    OPSWS --> ST
    TID --> ST
    ST --> CORE
    MTRD --> CORE
```

## Internal extract patterns on same swim lanes

```mermaid
flowchart LR
    subgraph L1[Untrusted / External]
        direction TB
        EMPTY1[ ]
    end

    subgraph L2[Edge / Mediation]
        direction TB
        EPUB[EDGE-PUB
Publication Edge]
    end

    subgraph L3[Business Enablement]
        direction TB
        READ[TRD-READOUT
Read-only Readout]
        RT[CORP-RTDATA
Corp Data Consumption]
        XF[XFER-CDG
Transfer Gateway]
        CA[CORP-APP
Corporate Apps]
    end

    subgraph L4[Infrastructure / System Management]
        direction TB
        OB[OBS-SEC
Observability]
    end

    subgraph L5[Crown Jewels]
        direction TB
        TC[TRD-CORE
Trading Core]
    end

    subgraph L6[Recovery / Resilience]
        direction TB
        EMPTY6[ ]
    end

    TC --> READ
    READ --> RT
    READ --> XF
    READ --> OB
    RT --> CA
    XF --> CA
    CA --> EPUB
```

## Trusted deployment and recovery on same swim lanes

```mermaid
flowchart LR
    subgraph L1[Untrusted / External]
        direction TB
        EMPTY1[ ]
    end

    subgraph L2[Edge / Mediation]
        direction TB
        EMPTY2[ ]
    end

    subgraph L3[Business Enablement]
        direction TB
        EMPTY3[ ]
    end

    subgraph L4[Infrastructure / System Management]
        direction TB
        DRL[DEV-REL
Build / Release]
        DEP[TRD-DEPLOY
Trusted Deploy]
        MGT[MGT-TRD
Trading Management]
    end

    subgraph L5[Crown Jewels]
        direction TB
        CORE[TRD-CORE
Trading Core]
    end

    subgraph L6[Recovery / Resilience]
        direction TB
        VAULT[RCV-VAULT
Backup Vault]
        DRA[DR-ALT
Recovery Site]
    end

    DRL --> DEP
    CORE -. pull approved content .-> DEP
    MGT --> DEP
    MGT --> CORE
    CORE --> VAULT
    CORE --> DRA
    VAULT -. approved restore via MGT .-> MGT
```
