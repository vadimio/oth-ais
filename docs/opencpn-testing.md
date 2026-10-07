# OpenCPN development and testing

**Planned workflow, October 7, 2026.** The first implementation delivers a Linux executable that serves AIS directly to OpenCPN. The commands below specify the intended interface; implementation and runtime results await approval.

## Desktop setup

Install the dependency-locked Python package into a dedicated environment. Its `oth-ais` executable runs the common backend with a private credential file, a fixed geographic query area and the TCP output enabled. Leave the marine transmitter disabled during desktop use.

```sh
oth-ais serve --config desktop.toml
oth-ais status --config desktop.toml
```

The desktop configuration supplies provider polling, the fixed area, private credential-file path, source policy and output addresses. The credential file remains outside Git. API requests follow the same account limiter used in normal operation.

In OpenCPN, add a connection with:

| Setting | Value |
| --- | --- |
| Connection type | Network |
| Protocol | TCP |
| Data protocol | NMEA 0183 |
| Address | `127.0.0.1` when backend and OpenCPN share the computer |
| Port | `10110` |
| Receive input | Enabled |
| Output on this connection | Disabled |
| Checksum checking | Enabled |

These interfaces are supported by [OpenCPN's connection documentation](https://opencpn.org/wiki/dokuwiki/doku.php?id=opencpn:manual_basic:set_options:connections). Validate the exact installed OpenCPN version's labels and behavior during the first test.

The desktop configuration explicitly enables loopback access. For a different LAN host, configure a specific private bind address and client allowlist/firewall. Native NMEA TCP has plain transport; use a private LAN/VPN or a tested tunnel for remote access. The status/target HTTP API uses authenticated access, separately from the NMEA stream.

## One AIS input for the merged view

When testing the merged feed, the backend receives local replay/direct reception and provides OpenCPN's AIS input. Disable any additional AIS feeds that would bypass this selection. A separate existing GPS input can remain active; the AIS service emits target reports only.

When preserving a direct local AIS connection to OpenCPN, choose supplemental Internet-only output and give the backend its own approved local-monitor input. Record which policy is active so a duplicated target can be traced to its input.

Use deterministic local-reception replay for software tests while the boat's navigation network is off. Replay records have explicit source identity, timestamps and end-of-stream/health transitions. They reach loopback/virtual interfaces only. Provider data stays real and private when testing the actual account.

## Acceptance observations

| Test | What to record |
| --- | --- |
| Live regional fetch | Backend success, source timestamp and private target count |
| OpenCPN rendering | Same MMSI/coordinates, speed/course and available name as the provider record |
| Known Class A/B fixtures | Correct position/static fields and message grouping |
| Unknown-class option | Explicit chosen wrapper, API class remaining unknown and resulting OpenCPN class display |
| Local overlap replay | Same MMSI switches to local; later Internet reports stay suppressed |
| Quiet/release replay | Short gaps preserve local authority; fresh remote output resumes after the hold |
| Provider outage/age | Output stops at expiry; measured OpenCPN lost/remove behavior |
| Client reconnect | Fresh selected snapshot with bounded queue/state |
| Slow client and oversized response | Resource ceilings and responsive status/local selection |
| Source/time tag blocks | Acceptance and actual source/age visibility in the installed OpenCPN |

The status API is the source-aware view: position time, receipt time, selected source, encoding assumption and suppression reason. OpenCPN's standard AIS display has its own caching and class/source presentation. Record the actual popup, source and age presentation during the test.

## Evidence and next stage

Store software version, redacted config, encoded test sentences, independent decode results and actual OpenCPN observations. Synthetic fixtures can enter the public repository; real positions and credentials stay private.

After this acceptance path works, feed the same selected observations to virtual CAN and independently decode the N2K output. Cerbo installation/resource checks can proceed before navigation equipment is powered. Physical Cerbo transmission and Axiom/Orca consumption remain scheduled for the next boat visit.
