# High-Level Security Architecture: Controlled Push/Pull Crown Jewel Model with Policy-Style Firewall Rules

## Purpose
This document updates the architecture for a Hong Kong regulated trading or exchange environment so that crown-jewel trading systems operate under a stricter transfer model:
- trading and crown-jewel systems should primarily push data out,
- any inbound software, configuration, or approved reference content should be pulled only from trusted intermediate repositories or tightly controlled local services,
- there must be a clear distinction between management and change-impacting tools versus read-only extract and telemetry flows,
- near-real-time and non-real-time data extracts for internal corporate consumers should use dedicated readout and consumption zones rather than direct integration into TRD-CORE,
- trading operations users and trading management GUIs should be modeled explicitly as separate from normal corporate workstations and domains.

This is not a literal physical air gap. It is a controlled-separation model with highly constrained trust boundaries, approved mediation points, and auditable one-way or pull-from-trusted-source patterns.

## High-level policy statement
The policy objective is to protect the confidentiality, integrity, availability, resilience, and orderly operation of the crown-jewel trading environment while still enabling broker connectivity, regulated business processing, operational support, controlled enterprise services, approved data extraction to corporate consumers, and dedicated trading operations access.

The policy intent is to:
- reduce the probability that corporate, internet-facing, SaaS, observability, or engineering systems can directly compromise or disrupt TRD-CORE,
- reduce hidden transitive trust created by shared tools such as identity, deployment, backup, monitoring, transfer, internal downstream data-consumption systems, or shared workstation domains,
- preserve necessary business and IT functionality through explicitly approved mediated flows,
- separate high-trust change paths from read-only extract and telemetry paths,
- separate trading operations workstations and directory services from corporate end-user and corporate identity environments,
- make all critical flows attributable, reviewable, and enforceable through zoning, firewall policy, privileged access control, and change management.

## Risks the model is designed to reduce
- Lateral movement from corporate desktops, enterprise applications, or shared user identity into mission-critical trading systems.
- Internet or broker-facing compromise propagating into crown-jewel systems through weakly segmented service tiers.
- Hidden reverse-control channels introduced by SIEM, observability, backup, SaaS connectors, third-party tooling, or internal near-real-time or batch data consumers.
- Unauthorized or unreviewed software, scripts, packages, playbooks, configuration, or data requests entering TRD-CORE.
- Excessive administrator reach and unmanaged interactive access into low-latency systems.
- Mixing of business publication or data extract paths with change-capable management tooling.
- Overly broad trusted relationships in identity, secrets, PKI, name services, deployment infrastructure, or internal data integration layers.
- Recovery tooling becoming a persistent privileged backdoor rather than an exceptional resilience control.

## Clustering model for zones and rules
It does make sense to cluster zones and firewall rules into two top-level control families:

### Business enablement cluster
This cluster supports market, broker, participant, publication, business-processing functions, approved downstream consumption of read-only data, and operator-facing trading functions.
- EXT-PUB
- EXT-BRK
- EDGE-PUB
- EDGE-BRK
- SVC-PUB
- SVC-TRD
- CORP-APP
- CORP-RTDATA
- TRD-OPS-WS
- INT-3P
- XFER-CDG for business transfer and regulated exchange scenarios
- TRD-READOUT for read-only business extracts and publication staging

### Infrastructure and system management cluster
This cluster supports administration, identity, deployment, observability, resilience, and recovery.
- TRD-CORE
- TRD-DEPLOY
- MGT-TRD
- TRD-ID
- MGT-CORP
- ID-SVC
- OBS-SEC
- RCV-VAULT
- DEV-REL
- DR-ALT
- CORP-USER as the origin of standard user activity but not privileged admin

### Why the clustering helps
This clustering helps separate business connectivity from state-changing infrastructure control. It also makes policy easier to govern because rules can be reviewed by purpose: market and business enablement versus privileged control, change, telemetry, resilience, and internal downstream data distribution.

## Core design principles
- TRD-CORE must not accept arbitrary inbound business or admin connectivity from corporate or internet-connected zones.
- TRD-CORE may push logs, backups, approved business extracts, and approved near-real-time business data out through dedicated relay or transfer paths.
- TRD-CORE may pull approved software, configuration, and tightly governed reference content only from trusted local or intermediate repositories in dedicated support zones.
- Management and change-impacting functions must be separated from read-only data extraction, streaming, and telemetry functions.
- Interactive administration must terminate in dedicated management zones and must not be mixed with observability, backup, business publication, or data-consumption paths.
- Trading operations workstations must remain separate from normal corporate desktops and should authenticate against a separate trading identity domain.
- Internal users performing trading operations or trading queries should access trading web GUIs and management web GUIs only from the dedicated trading operations workstation zone.
- Tools that can change state in TRD-CORE are higher trust and must sit in dedicated change-control subzones with stronger approval, logging, and separation.
- Internal near-real-time and batch consumers must not directly query TRD-CORE; they should consume from dedicated downstream read-only zones.

## Recommended zone naming standard
Use function-first zone names:
- EXT-PUB — public internet users and public data consumers
- EXT-BRK — brokers, participants, and market members
- EDGE-PUB — public edge / DMZ services
- EDGE-BRK — broker edge / participant ingress
- SVC-PUB — public or internet-facing application services
- SVC-TRD — trading support and gateway services
- TRD-CORE — crown-jewel trading and mission-critical low-latency services
- TRD-DEPLOY — trusted deployment, artifact mirror, package mirror, config pull services for trading
- TRD-READOUT — read-only export relays, telemetry relays, reporting extract relays from trading
- CORP-RTDATA — internal near-real-time or scheduled downstream consumption zone for approved corporate use
- TRD-OPS-WS — dedicated trading operations workstations and operator GUI access zone
- TRD-ID — trading directory, authentication, and policy services for the trading user and admin environment
- MGT-TRD — privileged management for the trading estate
- CORP-USER — desktops and end-user computing
- CORP-APP — internal business and corporate application services
- MGT-CORP — privileged management for the corporate estate
- XFER-CDG — controlled transfer and cross-domain gateway
- OBS-SEC — SIEM, observability, telemetry, and security analytics
- RCV-VAULT — backup, recovery, and cyber vault
- ID-SVC — shared enterprise identity, PKI, DNS, time, secrets, and directory services not specific to trading operations
- INT-3P — third-party, regulator, clearing, settlement, and B2B integration services
- SaaS-EXT — approved SaaS platforms
- DEV-REL — source, build, release, artifact origin, and software signing services
- DR-ALT — alternate recovery or disaster recovery environment

## Zone model

| Zone | Cluster | Purpose | Security posture | Key controls | Typical directionality |
|---|---|---|---|---|---|
| EXT-PUB | Business enablement | Public internet consumers and submitters | Untrusted | DDoS, anti-bot, rate limiting | To EDGE-PUB only |
| EXT-BRK | Business enablement | Brokers and participants | External but known parties | Dedicated connectivity, source allow-lists, cert trust | To EDGE-BRK only |
| EDGE-PUB | Business enablement | Public ingress and publication edge | Exposed mediation zone | WAF, reverse proxy, upload scanning, TLS controls | Inbound from EXT-PUB; outbound to SVC-PUB only |
| EDGE-BRK | Business enablement | Broker ingress edge | Exposed broker mediation zone | Protocol filtering, broker ACLs, session controls | Inbound from EXT-BRK; outbound to SVC-TRD only |
| SVC-PUB | Business enablement | Public application services | Internet-adjacent service zone | App segmentation, EDR, vulnerability management | From EDGE-PUB; to CORP-APP or XFER-CDG |
| SVC-TRD | Business enablement | Trading support, session, gateway, protocol mediation, trading web GUIs | Restricted service zone near trading | Protocol allow-lists, host controls, logging, EDR on supported hosts | From EDGE-BRK or TRD-OPS-WS; to TRD-CORE, TRD-READOUT, TRD-DEPLOY as approved |
| TRD-CORE | Infrastructure and system management | Matching, execution, low-latency core, crown jewels | Highest restriction | Default deny, no internet, no dual-homing, minimal agents, enclave segmentation | Pull from TRD-DEPLOY or limited TRD-ID/ID-SVC; push to TRD-READOUT, RCV-VAULT |
| TRD-DEPLOY | Infrastructure and system management | Trusted local deployment and artifact retrieval for trading | High-trust change zone | Artifact signing verification, package allow-lists, staged mirrors, approval workflow, full logging | Pull from DEV-REL/XFER-CDG; serve pull-only to TRD-CORE |
| TRD-READOUT | Business enablement | Read-only export relays, telemetry relays, reporting extract relays | High-control read-only egress zone | One-way relays, signed exports, integrity checks, no admin return path | Receive push from TRD-CORE; forward to XFER-CDG, OBS-SEC, CORP-RTDATA, SVC-TRD |
| CORP-RTDATA | Business enablement | Internal near-real-time and non-real-time data consumption for approved corporate systems | Restricted internal data-distribution zone | Dedicated firewalls, producer/consumer ACLs, append-only or publish-subscribe pattern, rate control, schema validation, no admin path back to TRD-CORE | Receive from TRD-READOUT; serve read-only data to CORP-APP |
| TRD-OPS-WS | Business enablement | Dedicated trading operations workstations for trading queries, operational GUIs, and trading web access | Restricted user zone separate from corp desktops | Separate network, hardened workstations, MFA, allow-listed destinations, session recording where feasible | To SVC-TRD and TRD-ID only |
| TRD-ID | Infrastructure and system management | Separate trading directory, authentication, GPO/policy, and trading identity services | High-trust identity zone for trading users/admins | Separate AD/domain, tiered admin, trust minimization, MFA integration | To TRD-OPS-WS, MGT-TRD, selected SVC-TRD/TRD-CORE auth endpoints |
| MGT-TRD | Infrastructure and system management | Privileged admin and operator management for trading | High-trust interactive management zone | Separate AD/domain use, PAM, MFA, bastions, session recording | To TRD-DEPLOY, SVC-TRD, selective admin endpoints in TRD-CORE, TRD-ID |
| CORP-USER | Infrastructure and system management | User desktops and office endpoints | Standard enterprise zone | EDR, DLP, proxy, device control | To CORP-APP and approved internet services |
| CORP-APP | Business enablement | Business applications and corporate services | Internal application zone | Segmentation, app controls, EDR | To EDGE-PUB, XFER-CDG, INT-3P, SaaS-EXT; pull/read from CORP-RTDATA |
| MGT-CORP | Infrastructure and system management | Privileged corporate management | High-trust admin tier | PAM, MFA, jump hosts, admin workstations | To CORP-APP, OBS-SEC, ID-SVC, CORP-RTDATA |
| XFER-CDG | Business enablement | Controlled transfer and cross-domain staging | Mediation zone | Malware scan, CDR, checksum, signature verification, approvals | Receives from TRD-READOUT, CORP-APP, DEV-REL; tightly governs onward movement |
| OBS-SEC | Infrastructure and system management | SIEM and observability | Collection and analysis zone | Log brokers, immutable storage, parser isolation | Receives from relays and managed zones; no reverse control path |
| RCV-VAULT | Infrastructure and system management | Backup and recovery vault | Isolated resilience zone | Immutable storage, isolated creds, restore approval | Receive backup from TRD-CORE/TRD-READOUT/CORP-APP; restore by exception |
| ID-SVC | Infrastructure and system management | Shared enterprise identity, secrets, PKI, naming, time | Shared but tightly governed infra zone | Tiered admin, trust minimization, cert lifecycle, restricted protocols | Limited scoped services to non-trading managed zones and selected shared infra |
| INT-3P | Business enablement | B2B integration and regulated exchanges | Controlled partner zone | API mediation, DLP, message validation | Bidirectional only with approved internal zones |
| SaaS-EXT | Business enablement | Approved SaaS and cloud control services | External service zone | SSO, CASB/SSPM, token scope, data review | From CORP-APP/OBS-SEC only |
| DEV-REL | Infrastructure and system management | Build, release, artifact origin | High-trust engineering zone | Signed artifacts, release approvals, repository controls | To TRD-DEPLOY and MGT zones, not direct to TRD-CORE |
| DR-ALT | Infrastructure and system management | Recovery site | Restricted alternate environment | Replication controls, isolated credentials, runbooks | Replication from primary; restore/failover by approval |

## Validation of trading workstation and separate domain pattern
The architecture explicitly includes the trading workstation pattern and separate trading domain pattern. The dedicated user access zone is `TRD-OPS-WS`, and the separate trading directory or domain services are represented by `TRD-ID`.

This design explicitly supports:
- internal trading workstations separate from normal corporate desktops,
- a separate trading directory or AD domain,
- operator and trading query access to trading or management web GUIs via `TRD-OPS-WS` to `SVC-TRD`,
- privileged admin and platform management separated further into `MGT-TRD`.

## Business use cases for extracts from trading core to corporate systems

### Near-real-time business use cases
Typical examples:
- internal market operations dashboards,
- surveillance and monitoring views for corporate control-room users,
- intraday risk or exposure dashboards,
- participant support and operations views,
- internal compliance or business-monitoring dashboards.

Recommended pattern:
- TRD-CORE pushes approved near-real-time records to TRD-READOUT.
- TRD-READOUT validates, filters, rate-limits, and republishes the feed to CORP-RTDATA.
- CORP-APP systems subscribe to or read from CORP-RTDATA.
- No corporate consumer should query TRD-CORE directly.

Design rules:
- use append-only, queue, stream, or publish-subscribe patterns rather than query-back into TRD-CORE,
- use fixed producer and consumer ACLs with narrow port sets,
- enforce message schema validation, replay protection, rate limits, and payload size controls,
- keep near-real-time distribution services separate from deployment, admin, and backup tooling,
- use separate service accounts for producer, relay, and consumer roles,
- log producer, relay, and consumer activity to OBS-SEC.

### Non-real-time or scheduled extract use cases
Typical examples:
- end-of-day reports,
- reconciliation files,
- business analytics loads,
- regulatory records or batch extracts for corporate processing,
- financial, billing, or downstream account-support data.

Recommended pattern:
- TRD-CORE pushes batch or scheduled extracts to TRD-READOUT.
- TRD-READOUT forwards approved files, tables, or packages either to XFER-CDG for controlled release or directly to CORP-RTDATA for tightly bounded internal consumption.
- CORP-APP systems retrieve the approved outputs from CORP-RTDATA or XFER-CDG, depending on sensitivity and workflow.

Design rules:
- keep scheduled extracts logically separate from near-real-time feed channels even if they use some shared relay infrastructure,
- apply file/package integrity checks, classification tags, and release approvals where required,
- use immutable or append-only staging where feasible,
- do not allow batch consumers to reuse extract channels as return paths into TRD-CORE,
- retain audit trails on extract generation, staging, approval, and downstream retrieval.

## Internal extract pattern without explicit proxies
Where explicit proxy infrastructure is not available, internal extracts should still be mediated using dedicated downstream consumption zones and constrained transport patterns.

Recommended pattern:
1. TRD-CORE pushes approved data to TRD-READOUT.
2. TRD-READOUT normalizes, validates, rate-limits, and republishes either near-real-time or scheduled data into CORP-RTDATA.
3. CORP-APP systems read or subscribe from CORP-RTDATA only.
4. No corporate consumer should establish sessions directly into TRD-CORE or into change-capable trading zones.

Recommended defence-in-depth controls for CORP-RTDATA when no explicit proxies exist:
- dedicated firewall boundary between TRD-READOUT and CORP-RTDATA,
- one-way initiation pattern from TRD-READOUT to CORP-RTDATA where practical,
- dedicated service accounts and mutual authentication for producer and consumer flows,
- append-only, publish-subscribe, queue-based, or staged file delivery rather than query-back into TRD-CORE,
- protocol and port allow-lists, fixed destination allow-lists, and strict egress rules,
- rate limiting, replay protection, schema validation, and payload size limits,
- no management plane sharing with TRD-DEPLOY or MGT-TRD,
- separate admin path for CORP-RTDATA from MGT-CORP only,
- detailed telemetry from CORP-RTDATA to OBS-SEC,
- cached or buffered delivery so brief downstream failure does not force reconnect logic into TRD-CORE.

## Separation of change versus read-only flows

### Change-impacting zones and tools
These zones can change state in TRD-CORE and therefore require the highest level of control:
- MGT-TRD
- TRD-DEPLOY
- DEV-REL
- TRD-ID for trading auth and policy administration
- limited administrative interfaces in shared ID-SVC

Typical change-impacting functions:
- software deployment
- package retrieval
- configuration retrieval
- patch retrieval
- orchestration and job execution
- privileged operator actions
- break-glass recovery
- trading user and admin policy administration

### Read-only or outbound-only zones and tools
These zones should not be allowed to drive state changes in TRD-CORE:
- TRD-READOUT
- CORP-RTDATA
- OBS-SEC
- XFER-CDG when handling exports
- public publication paths
- reporting and analytics consumers

Typical read-only functions:
- log relay
- market data export
- business extract export
- regulatory reporting extract handoff
- internal near-real-time data distribution
- internal scheduled data distribution
- observability and telemetry
- backup stream write

## Trusted pull model for crown jewels
The crown-jewel zone should adopt these patterns:

1. Push out from TRD-CORE
- telemetry to TRD-READOUT or dedicated relay
- business extracts to TRD-READOUT or XFER-CDG
- near-real-time and non-real-time internal data to TRD-READOUT and then CORP-RTDATA
- backups to RCV-VAULT
- replication to DR-ALT where approved

2. Pull in to TRD-CORE only from trusted local intermediaries
- packages and deployment content from TRD-DEPLOY
- approved configuration from TRD-DEPLOY
- approved limited reference data from a dedicated import path via XFER-CDG to TRD-DEPLOY or SVC-TRD
- tightly scoped identity, certificate, DNS, and time services from TRD-ID or selected ID-SVC components

3. No direct inbound from these zones into TRD-CORE
- CORP-USER
- CORP-APP
- CORP-RTDATA
- SaaS-EXT
- OBS-SEC
- XFER-CDG for general-purpose transfer
- DEV-REL
- public or broker edge zones except through explicitly approved service endpoints

## Ansible and deployment model
A recommended pattern for Ansible and similar tools is:
- DEV-REL is the authoritative source for signed packages, playbooks, templates, and approved release metadata.
- TRD-DEPLOY maintains a curated mirror or trusted repository for the trading environment.
- TRD-CORE nodes pull only approved artifacts from TRD-DEPLOY.
- MGT-TRD hosts the management console and operator access for deployment tooling.
- Any deployment server capable of changing TRD-CORE is managed as a high-trust change system and must not be reused for read-only extraction, telemetry, or internal data distribution.

A practical option is:
- Ansible control plane and approval console in MGT-TRD.
- Repository mirror, package cache, and artifact validation services in TRD-DEPLOY.
- TRD-CORE agents or scheduled pull jobs retrieve content from TRD-DEPLOY.
- TRD-DEPLOY itself pulls only from approved origins in DEV-REL or via XFER-CDG under release control.

## Key flows

| Use case | Source -> Destination | Direction | Pattern | Control intent |
|---|---|---|---|---|
| Broker order entry | EXT-BRK -> EDGE-BRK -> SVC-TRD -> TRD-CORE | Inbound | Controlled service ingress | Only approved broker service path |
| Private broker application | EXT-BRK -> EDGE-BRK -> SVC-TRD | Inbound | App access | Broker-facing app access only |
| Trading operator GUI access | TRD-OPS-WS -> SVC-TRD | Inbound internal | Dedicated workstation access | Restrict trading queries and operator access to separate workstation zone |
| Trading auth and workstation policy | TRD-OPS-WS -> TRD-ID | Internal auth | Dedicated trading identity | Separate trading user and auth domain from corp user estate |
| Public announcement / IPO intake | EXT-PUB -> EDGE-PUB -> SVC-PUB -> CORP-APP | Inbound | Business intake | Business processing outside TRD-CORE |
| Public or market data publication | TRD-CORE -> TRD-READOUT -> XFER-CDG or SVC-TRD/EDGE-PUB -> EXT-PUB | Outbound | Read-only export | Publication without opening control path |
| Business extract from trading | TRD-CORE -> TRD-READOUT -> XFER-CDG -> CORP-APP or INT-3P | Outbound | Read-only export | Controlled read-only extract |
| Near-real-time internal extract | TRD-CORE -> TRD-READOUT -> CORP-RTDATA -> CORP-APP | Outbound | Read-only streaming/distribution | Internal consumers read from dedicated downstream zone only |
| Non-real-time internal extract | TRD-CORE -> TRD-READOUT -> CORP-RTDATA or XFER-CDG -> CORP-APP | Outbound | Scheduled/batch export | Controlled batch delivery without return path |
| Telemetry from trading | TRD-CORE -> TRD-READOUT -> OBS-SEC | Outbound | Push-only telemetry | Logging and observability only |
| Backup and recovery copy | TRD-CORE -> RCV-VAULT | Outbound | Push-only backup | Backup write without admin back path |
| Restore to trading | RCV-VAULT -> MGT-TRD -> TRD-CORE | Inbound by exception | Break-glass recovery | Exceptional restore only |
| Deployment content retrieval | TRD-CORE -> TRD-DEPLOY | Pull | Trusted pull | Pull approved packages/config only |
| Artifact mirror refresh | TRD-DEPLOY -> DEV-REL or XFER-CDG | Pull | Curated upstream sync | Retrieve approved signed release content |
| Management of deployment tooling | MGT-TRD -> TRD-DEPLOY | Inbound admin | Privileged management | Change control and console access |
| Reference data import to trading | Approved source -> XFER-CDG -> TRD-DEPLOY or SVC-TRD -> TRD-CORE pull | Staged inbound + pull | Exception path | Validated limited import |
| Identity and certificates | TRD-CORE -> TRD-ID or selected ID-SVC | Limited query/pull | Shared infra dependency | Scoped identity and infrastructure access |
| Vendor support | Vendor -> MGT-TRD | Inbound by exception | Brokered support | Ticketed privileged path only |

## Firewall policy standard
The firewall policy should be read and governed as two grouped rule families rather than one flat engineering list. Business enablement rules govern controlled market, user, extract, and corporate-consumption paths, while infrastructure and system-management rules govern privileged control, deployment, identity, observability, recovery, and hard deny boundaries.

### Business enablement rules

| Rule ID | Source zone | Destination zone | Service / protocol class | Direction | Policy action | Inspection / control | Policy intent |
|---|---|---|---|---|---|---|---|
| BE-001 | EXT-PUB | EDGE-PUB | Approved web services | Inbound | Allow by exception | DDoS, WAF, TLS, anti-bot | Support public-facing services safely |
| BE-002 | EXT-BRK | EDGE-BRK | Approved broker connectivity | Inbound | Allow by exception | Source allow-list, broker auth, session controls | Support broker access safely |
| BE-003 | EDGE-BRK | SVC-TRD | Approved broker app/FIX gateways | Inbound | Allow by exception | Protocol filtering, rate limits, ACLs | Mediate broker traffic before trading support |
| BE-004 | TRD-OPS-WS | SVC-TRD | Approved GUI/web/app protocols | Inbound internal | Allow by exception | MFA, workstation hardening, destination allow-list | Support operator trading queries and GUIs |
| BE-005 | SVC-TRD | TRD-CORE | Approved trading service ports | Inbound | Allow by exception | Destination allow-list, host ACLs, logging | Permit only required trading service ingress |
| BE-006 | TRD-CORE | TRD-READOUT | Logs, extracts, near-real-time feed, publication staging | Outbound | Allow by exception | One-way design intent, relay controls, integrity checks | Separate read-only egress from change paths |
| BE-007 | TRD-READOUT | XFER-CDG | Approved exports | Outbound | Allow by exception | Signature check, checksum, approval workflow | Controlled business export |
| BE-008 | TRD-READOUT | CORP-RTDATA | Approved near-real-time or scheduled feed | Outbound | Allow by exception | Fixed destination allow-list, mTLS or service auth, schema checks, rate limits | Feed downstream internal consumers safely |
| BE-009 | CORP-APP | CORP-RTDATA | Approved read/subscription service | Pull/read | Allow by exception | Consumer ACLs, read-only service accounts, no admin path | Ensure consumers use dedicated downstream zone |
| BE-010 | CORP-APP | XFER-CDG | Approved exports/import staging | Bidirectional controlled | Allow by exception | Data validation, malware scan, approval | Mediate business exchange safely |
| BE-011 | CORP-APP | INT-3P | Approved B2B APIs/files | Bidirectional controlled | Allow by exception | API gateway, DLP, message validation | Support regulators, clearing, and partners |
| BE-012 | CORP-APP | SaaS-EXT | Approved SaaS APIs/web | Outbound | Allow by exception | SSO, CASB/SSPM, proxy, token scoping | Support business tooling without touching core |

### Infrastructure and system management rules

| Rule ID | Source zone | Destination zone | Service / protocol class | Direction | Policy action | Inspection / control | Policy intent |
|---|---|---|---|---|---|---|---|
| IM-001 | CORP-USER | TRD-CORE | Any | Inbound | Deny | Firewall deny, log attempts | Prevent desktop-to-core lateral movement |
| IM-002 | CORP-APP | TRD-CORE | Any | Inbound | Deny | Firewall deny, log attempts | Prevent app-tier bridging into crown jewels |
| IM-003 | CORP-RTDATA | TRD-CORE | Any | Inbound | Deny | Firewall deny, no reverse sessions | Prevent downstream consumers querying back into core |
| IM-004 | OBS-SEC | TRD-CORE | Any | Inbound | Deny | Deny polling/admin, no reverse path | Prevent telemetry platform becoming control plane |
| IM-005 | DEV-REL | TRD-CORE | Any | Inbound | Deny | Deny direct deployment path | Force deployment through TRD-DEPLOY |
| IM-006 | SaaS-EXT | TRD-CORE | Any | Inbound | Deny | Deny all direct cloud/SaaS reachability | Prevent unmanaged external trust into core |
| IM-007 | CORP-USER | TRD-OPS-WS | Any | Lateral | Deny | Segmentation deny, log attempts | Keep trading workstations isolated from corp workstation zone |
| IM-008 | TRD-OPS-WS | TRD-ID | Auth, GPO, policy, DNS where needed | Internal auth | Allow by exception | Domain segregation, port allow-list, monitoring | Support separate trading workstation domain |
| IM-009 | MGT-TRD | TRD-CORE | Approved admin protocols | Inbound | Allow by exception | PAM, MFA, bastion, session recording | Controlled privileged administration |
| IM-010 | TRD-ID | SVC-TRD | Auth and policy services | Internal service | Allow by exception | Restricted trust, auth protocol allow-list | Separate trading identity supports trading apps |
| IM-011 | TRD-ID | TRD-CORE | Selected auth, time, and name services only | Internal service | Allow by exception | Restricted ports, monitored dependencies | Support scoped trading identity dependencies |
| IM-012 | TRD-READOUT | OBS-SEC | Telemetry/log forwarding | Outbound | Allow by exception | Push collectors, parser isolation | Support observability without reverse control |
| IM-013 | MGT-CORP | CORP-RTDATA | Approved admin protocols | Inbound | Allow by exception | Separate admin tier, PAM, MFA | Administer downstream data zone separately from trading |
| IM-014 | TRD-CORE | RCV-VAULT | Backup / replication write | Outbound | Allow by exception | Immutable target, isolated creds | Support resilience without routine restore path |
| IM-015 | RCV-VAULT | TRD-CORE | Any direct restore | Inbound | Deny | No direct route; restore only via MGT-TRD | Prevent backup platform as standing admin path |
| IM-016 | RCV-VAULT | MGT-TRD | Restore package / control only | Inbound | Allow by exception | Break-glass approval, logging | Controlled recovery path |
| IM-017 | TRD-CORE | TRD-DEPLOY | Artifact/config retrieval | Pull | Allow by exception | Package signing, allow-list, repo auth | Trusted pull of approved content |
| IM-018 | TRD-DEPLOY | TRD-CORE | Any general admin or push | Inbound | Deny except response traffic | Stateful firewall, no unsolicited initiation | Preserve pull-only deployment model |
| IM-019 | DEV-REL | TRD-DEPLOY | Signed artifact sync | Inbound to TRD-DEPLOY or pull response | Allow by exception | Release approval, mirror controls, signature validation | Curated upstream content only |
| IM-020 | MGT-TRD | TRD-DEPLOY | Admin console / repo management | Inbound | Allow by exception | PAM, MFA, session recording | Separate management console from repo consumers |
| IM-021 | XFER-CDG | TRD-DEPLOY | Approved staged import only | Inbound | Allow by exception | Scan, CDR, validation, approvals | Limited import path for staged content |
| IM-022 | XFER-CDG | TRD-CORE | Any general transfer | Inbound | Deny | No direct import to core | Prevent bypass of staging and validation |
| IM-023 | TRD-CORE | TRD-ID or selected ID-SVC | DNS, time, auth, cert, secrets | Limited query/pull | Allow by exception | Port/protocol allow-list, restricted trust, monitoring | Support essential identity and infrastructure services |
| IM-024 | MGT-CORP | CORP-APP | Approved admin protocols | Inbound | Allow by exception | PAM, MFA, admin workstation policy | Controlled corporate administration |
| IM-025 | Vendor | MGT-TRD | Approved remote support path | Inbound by exception | Allow by ticket | Time-bound access, PAM, recording | Controlled vendor intervention |

## Policy and standards set
- Zone classification and clustering standard.
- Interzone connectivity and firewall governance standard.
- Crown-jewel push-out and trusted-pull standard.
- Trading operator workstation and trading directory segregation standard.
- Near-real-time and non-real-time internal data distribution standard.
- Trading deployment and artifact repository standard.
- Read-only relay and telemetry isolation standard.
- Privileged access and admin workstation standard.
- Backup, recovery, and break-glass restore standard.
- Secure transfer and staged import standard.
- Identity, PKI, secrets, DNS, and time services standard.
- Broker/member connectivity standard.
- Third-party and SaaS integration standard.
- Exception, risk acceptance, and review-cycle standard.
