# Implementation plan

**Awaiting design approval, October 7, 2026.** Deliver regional Internet traffic first, then qualify plotter delivery with the existing AIS equipment operating throughout.

## Decisions for approval

| Decision | Recommendation |
| --- | --- |
| Initial hosting | Trial a lightweight Python service on Cerbo; retain the option to move to a dedicated fanless Linux server |
| Initial useful release | Authenticated LAN targets with local/Internet source labels, age and source health |
| Navigation output | Implement a separately enabled publisher; qualify actual plotter provenance and duplicate handling before live use |
| Near-vessel policy | Keep Internet output outside a proposed 25 nm protected radius; local-MMSI priority everywhere |
| Missing class information | Seek provider clarification/raw-message metadata; keep unknown-class observations in the LAN view |
| Community contribution | Opt in after mobile-station acceptance, privacy consent and receiver-output qualification |

The defaults and failure rules have one authority: [Architecture](architecture.md). Configuration references that policy rather than duplicating thresholds throughout the implementation.

## 1. Provider and equipment qualification

Deliver a concise qualification report with redacted evidence.

- Confirm account access, permitted regional queries, coverage, timestamp meaning and mobile-station contribution requirements
- Request the provider's AIS-class/original-message metadata options and clarify unavailable-speed sentinels
- Measure payload sizes and errors using the account-wide request limiter
- Inventory local receiver NAME, model, raw-output options, source forwarding and target PGNs
- Establish trusted own-vessel GPS selection and fix-validity behavior
- Identify exact plotter models, firmware, accepted AIS PGNs, source-selection controls and stale-target handling
- Check Cerbo runtime, supervisor, TLS trust, memory, CPU, navigation CAN state and charging-service baseline

Exit: provider contract recorded, input identities known, and a bounded first-release workload selected. Retain uncertainty about Internet coverage explicitly. Vessel locations and account responses stay private.

## 2. Read/display release

Implement a Python package with small domain modules: observation types, provider adapter, local input, target selection, expiry, status and deployment tooling. Add a concise API/schema only after inventorying existing consumers. Use mature HTTP and parsing libraries where they reduce failure-prone plumbing.

Deliver:

- Strict configuration and private credential-file validation
- Persistent account-wide rate-limit state, with a conservative startup wait and bounded retry/backoff
- Regional queries, timestamp-preserving normalization and bounded response handling
- One selected target per MMSI, alongside inspectable per-source evidence
- An authenticated LAN endpoint and a source-aware traffic view or existing chart-viewer integration
- Normal/economy/disabled controls, data counters and human-readable incident notices
- Reproducible ARMv7/ARM64/x86 Linux packaging and versioned configuration

Tests cover encoded units, missing fields, malformed envelopes, sentinel values, duplicates, out-of-order data, future time, clock jumps, stale GPS, empty results, date-line bounds, oversized gzip data, local priority and account-rate compliance through restart. All fixtures use synthetic vessels and locations.

Exit: offline tests pass and a read-only Cerbo trial meets the documented resource/coexistence budgets. Installing that trial requires a deployment approval; its CAN transmitter remains disabled.

## 3. Contribution adapter

Implement checksum-verified received-sentence forwarding with a short, bounded queue, byte counters and an explicit enable control. Reuse the receiver's raw output when available. Separate receiver-origin data from Internet data structurally, including provenance checks at the upload boundary.

Tests prove that provider data, gateway echoes, own-vessel reports, malformed sentences and unsupported commands produce zero uploaded records. Reconnect tests verify old queued reports are discarded. Provider feedback confirms a usable mobile feed and continued access terms.

Exit: an approved private upload configuration, consent record and provider acceptance. Schedule this phase earlier if account access requires an active contribution feed.

## 4. Marine publisher and bench tests

Select and pin the reviewed NMEA 2000 stack. Implement a small marine adapter with explicit interface selection, stable device identity, a narrow PGN allowlist, output leases and send-time revalidation. Cross-check its frames through an independently implemented decoder. Retain dependency notices and an SBOM.

Tests cover:

1. Known Class A/B targets, missing values, dimensions and motion units
2. Fast-packet assembly, sequence rollover, dropped frames and address changes
3. Unique NAME arbitration and startup with occupied addresses
4. Same-MMSI arrival from VHF immediately cancelling remote output
5. Target age expiry while queued and duplicate snapshots preserving age
6. Lost GPS, receiver input, Internet, process link and core process
7. Unknown receiver identity, wrong interface/bitrate and CAN error state
8. Queue saturation, publication budgets, memory pressure and storage failure
9. Process restart, stale disk cache and overlapping-host migration
10. Separation from BMS, charging, autopilot, own-GNSS and RF transmit commands

Begin with virtual CAN and private replays. Move to an isolated, powered display bench for actual plotter rendering, target expiry, source presentation and alarms. Synthetic MMSIs stay inside virtual or physically isolated test networks.

Exit: a completed compatibility report for each display. A failed provenance/handover test keeps that display on the source-aware LAN-view route.

## 5. Supervised boat commissioning

Prepare immutable pre-change configuration exports, the exact signed/hashed installation artifact, a rollback command and a post-change record. Keep charging and native AIS baselines visible to the operator.

| Live check | Required observation |
| --- | --- |
| Receive-only startup | Correct GPS and local AIS identity; unchanged electrical telemetry/watchdog health |
| Bounded real-target publication | A qualified remote target appears with its actual MMSI and reviewed source/age behavior |
| Local handover | A chosen real MMSI changes from remote to local while the local sensor path remains intact |
| Expiry and Internet outage | Gateway stops expired output; display loses or clearly ages the remote entry within the measured interval |
| Core/adapter stop | Output stops within its lease deadline; local equipment remains usable |
| Restart and recovery | A fresh session qualifies sources and time before output resumes |
| Contribution | Provider receives only approved local radio reception; bandwidth counters agree |

Run one combined scripted failure session where it can establish several properties with fewer interruptions. Mark each result as observed, failed or unperformed. Maintain separate evidence for each safety property.

Exit: owner reviews the results and explicitly enables live output. Establish a monitored soak period before relying on persistent operation. Stop OTH services as the first rollback action; the native transceiver wiring remains intact.

## 6. Dedicated-server migration and integrations

Move fetching, selection and display to the boat's dedicated Linux server once it is installed. Retain the qualified CAN adapter or use a tested dedicated gateway. Add mutually authenticated transport, clock-skew tests and single-publisher handover.

Optional integrations:

- Source-labelled Signal K target observations
- A Home Assistant summary and link to the traffic view
- Compatible chart viewers using approved nautical/satellite layers
- Privacy-controlled regional sharing or additional provider adapters

Keep charts and AIS transport independently versioned. Native plotter chart admission and E120/Axiom Ethernet migration stay owned by their respective projects.

## Maintainability and release checks

Use typed Python boundaries, pure selection rules, docstrings explaining policy choices and domain-focused modules below 500 lines. CI runs unit/replay tests, typing, lint, dependency review, secret scanning and an artifact reproducibility check. Hardware evidence belongs in a separate commissioning report.

Release documentation covers supported inputs/displays, credential provisioning, resource budgets, data rights, upgrade/rollback and operator-visible failure states. Public issues contain redacted evidence. Live coordinates, vessel identifiers and provider keys remain private.

## Remaining approval risks

1. The documented AIS Hub schema leaves AIS class unknown for many targets
2. Plotters may collapse source provenance and give delayed positions a fresh receipt time
3. Cached remote entries can survive after the gateway stops sending
4. Local-receiver monitoring must distinguish silence, healthy zero targets and forwarding loops
5. Cerbo CAN transmission adds a custom-service maintenance and host-isolation responsibility
6. Provider contribution terms may constrain mobile feeds, sampling and access continuity
7. Variable terrestrial coverage leaves some offshore areas sparse or empty
8. Legacy/modern plotter Ethernet compatibility requires a separate network design

The first release can provide useful LAN traffic awareness while these plotter-output questions are resolved.
