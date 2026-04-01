# Mermaid Diagrams: Controlled Push/Pull Crown Jewel Model with Policy-Style Rule Grouping

## High-level zone connectivity by cluster

```mermaid
flowchart LR
    subgraph BIZ[Business enablement]
        EXTBRK[EXT-BRK
Brokers]
        EXTPUB[EXT-PUB
Public Internet]
        EBRK[EDGE-BRK
Broker Edge]
        EPUB[EDGE-PUB
Public Edge]
        STRD[SVC-TRD
Trading Support / GUIs]
        SPUB[SVC-PUB
Public App Services]
        CAPP[CORP-APP
Corporate Apps]
        CRT[CORP-RTDATA
Corp Data Consumption]
        TOPS[TRD-OPS-WS
Trading Workstations]
        TREAD[TRD-READOUT
Read-only Readout]
        XFER[XFER-CDG
Transfer Gateway]
        INT3P[INT-3P
B2B / Partners]
    end

    subgraph INFRA[Infrastructure and system management]
        TCORE[TRD-CORE
Crown Jewels]
        TDEP[TRD-DEPLOY
Trusted Deploy]
        TID[TRD-ID
Trading Directory]
        MT[MGT-TRD
Trading Mgmt]
        MC[MGT-CORP
Corporate Mgmt]
        IDSVC[ID-SVC
Shared Identity / PKI]
        OBS[OBS-SEC
SIEM / Observability]
        VAULT[RCV-VAULT
Backup / Recovery]
        DEV[DEV-REL
Build / Release]
        DR[DR-ALT
Recovery Site]
        CUSER[CORP-USER
Desktops]
    end

    EXTBRK --> EBRK
    EXTPUB --> EPUB
    EBRK --> STRD
    TOPS --> STRD
    TOPS --> TID
    TID --> STRD
    TID --> TCORE
    EPUB --> SPUB
    SPUB --> CAPP
    CUSER --> CAPP
    MC --> CAPP
    MC --> CRT
    STRD --> TCORE
    MT --> TCORE
    MT --> TDEP
    MT --> TID
    TCORE --> TREAD
    TREAD --> OBS
    TREAD --> XFER
    TREAD --> CRT
    CRT --> CAPP
    XFER --> CAPP
    CAPP --> EPUB
    CAPP --> INT3P
    TCORE --> VAULT
    TCORE --> DR
    TCORE -. pull approved content .-> TDEP
    TDEP --> DEV
    TCORE --> IDSVC
```

## Policy rule group overview

```mermaid
flowchart TB
    subgraph BE[Business enablement rules]
        PUB[Public and broker ingress]
        OPS[Trading operator access]
        EXT[Read-only extracts]
        DIST[Corporate data consumption]
        B2B[Corporate partner/SaaS flows]
    end

    subgraph IM[Infrastructure and management rules]
        DENY[Default deny to TRD-CORE]
        IDEN[Trading identity and auth]
        DEP[Trusted pull deployment]
        OBS[Observability and logging]
        REC[Backup and restore]
        PAM[Privileged admin]
    end

    PUB --> EXT
    OPS --> EXT
    EXT --> DIST
    DENY --> DEP
    DENY --> OBS
    DENY --> REC
    IDEN --> PAM
    PAM --> DEP
```

## Trading operations workstation access

```mermaid
sequenceDiagram
    participant WS as TRD-OPS-WS
    participant TD as TRD-ID
    participant ST as SVC-TRD
    participant MT as MGT-TRD
    participant TC as TRD-CORE

    WS->>TD: Authenticate to trading domain
    TD-->>WS: Auth and policy
    WS->>ST: Access trading GUI / query tools
    ST->>TC: Approved trading query or app service
    MT->>TC: Separate privileged admin path
```

## Internal extract patterns

```mermaid
sequenceDiagram
    participant TC as TRD-CORE
    participant RO as TRD-READOUT
    participant RT as CORP-RTDATA
    participant XF as XFER-CDG
    participant CA as CORP-APP
    participant OB as OBS-SEC

    TC->>RO: Push approved data extracts
    RO->>RO: Validate / filter / classify
    RO->>RT: Near-real-time internal feed
    RO->>XF: Scheduled or controlled batch export
    RT->>CA: Read-only subscribe or query
    XF->>CA: Approved batch retrieval
    RO->>OB: Relay telemetry
    RT->>OB: Consumer and service logs
```
