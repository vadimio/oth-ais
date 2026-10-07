# Implementation plan

We are building the Python service for OpenCPN on Linux, with Docker as the first deployment.
The second phase runs the same backend on Cerbo, measures its workload and tests its NMEA 2000 output with the navigation equipment turned on aboard.
In a later release, we will move the backend to a standalone RISC boat computer with Signal K and NMEA 2000 support. We may keep the N2K module on Cerbo and use its marine connection; we will decide that during the later migration.

## Deliverables and order

| Delivery | Result |
| --- | --- |
| Python backend | Regional AIS Hub acquisition, typed observations, local reception, source selection, bounded lifecycle and status API |
| Linux/OpenCPN release | Installable `oth-ais` console executable and Docker image providing an AIS TCP input to OpenCPN |
| N2K adapter | Python marine receive/output module with independent final suppression and publication leases |
| Cerbo deployment | Reproducible ARMv7 installation, supervision, resource limits and recovery |
| Boat commissioning | Proven transmission through Cerbo and target consumption by Axiom+, Orca Core 2 and any additional tested client |

Selection rules and default limits have one authority: [Architecture](architecture.md). The [local coexistence design](local-ais-coexistence.md) explains why and when local reception takes priority. Network/router and legacy-plotter Ethernet work stay deferred.

## 1. Core and provider contract

Create a typed Python package with separate provider, selection, input and output modules. Ship the first-release `oth-ais serve --config ...` entry point with strict configuration, private credential loading, clocks and structured status. Use established protocol and HTTP libraries.

Inspect a bounded actual account response through the account-wide request limiter. Record response fields, timestamp semantics, class metadata, unavailable-value encoding and access/contribution terms. Keep raw vessel locations and account details in private evidence. Provider queries use a fixed test area initially, so live boat navigation is optional for this stage.

Implement:

- Regional requests and a restart-aware, account-wide minimum polling interval
- Bounded compressed/decompressed parsing, field counts and allocations
- Separate local/provider observations, position expiry, target revisions and five-/nine-minute local fallback windows
- An authenticated target/status API exposing selected source, age and suppression reasons
- Normal/economy/disabled controls with measured data counters
- Graceful cancellation, retry/backoff, incident logs and observable resource limits

Tests use synthetic positions and clocks: malformed/empty envelopes, duplicates, missing fields, sentinel values, out-of-order observations, future timestamps, UTC jumps, date-line areas, decompression limits and account throttling through restart.

Complete when the backend runs on Linux, its tests pass and a private live API sample establishes the deployed provider contract. Class metadata and an explicit compatibility-encoding choice are separate recorded decisions.

## 2. First working OpenCPN release

Implement NMEA 0183 AIS output with `pyais`, following [OpenCPN testing](opencpn-testing.md). Deliver the Linux executable, Docker image, dependency-locked installation, desktop config example and start/stop/status commands.

Provide a merged stream for OpenCPN when the backend is its AIS source. Supplemental mode supports clients with direct local AIS reception and a backend local-monitor input. Client connections receive fresh selected snapshots and subsequent revisions. Bounded per-client queues coalesce updates and close stalled clients.

Verify in actual OpenCPN:

1. Real Internet targets appear at the provider coordinates with correct speed/course and available static fields
2. A synthetic local replay for the same MMSI replaces the Internet-selected observation
3. Internet updates during the local position window leave the locally selected position unchanged
4. Five-/nine-minute silence selects a fresh Internet position, including an eligible cached record; confirmed receiver failure permits earlier fallback
5. Provider outage/expiry stops output, with OpenCPN's lost/remove behavior measured
6. Slow/disconnected clients recover through fresh snapshots and bounded memory
7. Unknown-class compatibility encoding and any supported source/time tag blocks have their display consequences recorded
8. Monitoring loss allows timed Internet fallback; metadata-only local updates leave the position timer running; returning local positions take over immediately

Synthetic replay reaches only the local desktop connection or virtual CAN test network. Keep OpenCPN route/autopilot output disabled on this connection.

Complete when the Linux executable serves OpenCPN and these observations have a saved test record. Automated decoding alone establishes encoding evidence; actual OpenCPN rendering completes this stage.

## 3. Local reception and N2K adapter

Prefer the existing Python `python-can`/`nmea2000` foundations. Audit and pin the encoder, `N2KDevice` address-claim wrapper, send framing, management behavior and allocation limits. Keep dependency versions isolated from existing boat services. A tested established alternative can occupy the same adapter boundary if these checks fail.

Implement `oth-ais can-agent`, its typed Unix-socket contract and the single CAN receive path used by local target tracking and final output checks. Monitor approved receiver NAME/address changes, distinguish VHF-received from own/gateway reports and qualify a receiver-health signal.

Virtual-CAN tests cover:

1. Correct Class A/B position/static PGNs, unavailable fields and independent decoding
2. Unique identity/address claiming, occupied addresses, source-address changes and management-rate limits
3. Fast-packet grouping, incomplete frames, sequence rollover and bounded reassembly
4. Same-MMSI local arrival cancelling both remote position and static queues
5. Local arrival during backlog or a multi-frame send, with the residual in-flight bound recorded
6. Gateway echoes, multiplexed inputs, own-vessel reports and conflicting data
7. Position expiry, five-/nine-minute boundary cases, receiver failure, monitoring loss, GPS loss and broken backend lease; Internet fallback continues through local-input failure while CAN/lease failures stop marine output
8. Queue saturation, suppression-state saturation, CPU/RAM pressure and crash loops
9. Restart, missing state storage and overlapping-process identity/account locks
10. Interface/PGN restrictions preserving BMS, charging and autopilot behavior

Complete when frame/selection tests pass, the dependency audit is recorded and protocol output is independently verified. Axiom and Orca consumption stay pending their equipment tests.

## 4. Cerbo deployment preparation

Build the same package for the observed Cerbo Python/ARMv7 runtime. Prepare an immutable installation artifact, dependency hashes, private configuration, independent supervision, explicit enabled outputs and rollback. Preserve any pre-change files and record installed bytes/version afterward.

With deployment approval, verify imports, console startup, API/TCP availability, supervisor recovery and CPU/RAM/storage limits on Cerbo. Gather an electrical-service baseline and compare it during maximum OTH workloads. The CAN adapter starts in passive receive mode, using available navigation data without writing management or AIS frames.

A powered-down navigation network produces a clear unavailable-input status. Its physical transmit/rendering checks remain scheduled for the boat visit in roughly three weeks. Desktop/OpenCPN testing can continue throughout.

Complete when the service installs and runs within measured budgets, with recovery observed and hardware-transmission checks explicitly pending.

## 5. Powered-network boat commissioning

Keep the existing receiver and plotter wiring intact. Prepare the package/version evidence, interface/source inventory, supported-client list, stop command and an operator-visible state summary. Inspect the existing navigation CAN error state after powering the network; preserve bitrate and unrelated interfaces.

Use one supervised session with the following distinct observations:

| Check | Evidence |
| --- | --- |
| Passive startup | Approved AIS receiver identity and real local targets; trusted own-position and healthy electrical services |
| Controlled write through Cerbo | Actual AIS frames reach the physical N2K bus from the intended adapter identity |
| Axiom+ and Orca | Each device displays expected MMSI/position/static fields; source/age/encoding behavior recorded separately |
| Local handover | Fresh local position stops supplemental publication; five-/nine-minute silence or confirmed receiver failure allows a newer, fresh Internet position; returning local reception takes over |
| Receiver and monitor failure | Fresh Internet tracking continues under the fallback rules, with source/fault status shown to the operator |
| Outage and expiry | Remote output stops and each display's cached-target behavior is measured |
| Backend/adapter failure | Publication stops within the lease/send bound while native AIS continues |
| Restart and recovery | Fresh source/time state precedes output; address/account ownership remains unique |
| Resource soak | CPU/RAM/network/bus budgets hold alongside normal boat services |

Use physically isolated replay for synthetic vessel tests. Mark every check observed, failed or unperformed. A successful write proves the transport; consumption/expiry/handover require the corresponding client observations.

Complete when the owner reviews the results and enables ongoing marine output. Rollback stops OTH processes while native AIS remains connected.

## 6. Optional contribution and later migration

Confirm mobile-feed acceptance, data rights and API entitlement with AIS Hub. Qualify a genuine receiver-output path before enabling contribution. Test that Internet records, gateway echoes, own-vessel reports and configuration commands produce zero uploaded records. Discard old outage backlog and record uploaded bytes.

Move the unchanged Python backend to a dedicated boat server later if useful. Cerbo may retain the CAN adapter, with authenticated cross-host transport, bounded clock skew, local final checks and one active publisher. Optional Home Assistant/Signal K consumers use the common API.

## Review and evidence

CI checks typing, lint, synthetic unit/replay tests, both encodings, queue bounds, dependency/license notices and secret exclusion. Keep modules domain-focused and readable, with docstrings for source authority, clocks and lifecycle choices. Preserve actual equipment results in commissioning records.

The review distinguishes provider schema evidence, Linux/OpenCPN behavior, Cerbo installation/resource evidence, physical CAN transmission and each client's consumption. Runtime and hardware results remain unperformed until observed. Each later deployment/change retains the corresponding owner approval.
