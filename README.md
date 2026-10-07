# OTH AIS (Over-The-Horizon AIS)

Your AIS receiver shows the vessels it can hear over VHF. An Internet connection can give you a wider view: ships approaching from farther away, traffic along your route, and vessels reported by receivers beyond your own range.

OTH AIS will bring that extra traffic into OpenCPN and onto the boat's NMEA 2000 network. We're building it for S/V Dash, with Raymarine Axiom+ and Orca Core 2 as the first onboard displays to test. The same Python service should work on other Linux-equipped boats.

**Status: design proposal, October 7, 2026. Awaits owner approval.**

## How it fits aboard

One Python backend gets nearby traffic from [AIS Hub](https://www.aishub.net/api), listens to the boat's local AIS, and decides which observation to use for each vessel. It provides two outputs:

- **OpenCPN:** a single AIS feed combining Internet traffic with locally received vessels
- **NMEA 2000:** additional Internet targets alongside the local AIS receiver's existing reports

The receiver keeps its direct connection to the boat's instruments. A separate status API shows where each observation came from, how old its position is, and why the service is displaying or withholding it.

## When local AIS picks up the same vessel

Imagine a ship first appears through AIS Hub. Later, your receiver hears it over VHF. Both reports carry the same MMSI, the vessel's AIS identifier. From that moment, local reception takes priority. OpenCPN gets the local observation, and OTH stops sending Internet updates for that ship onto NMEA 2000.

That priority survives normal gaps between radio reports. Anchored or slow-moving vessels can report their position at [three-minute intervals](https://www.navcen.uscg.gov/types-of-ais). The proposed local hold is 15 minutes after the last genuine reception, giving intermittent reception some room. During that hold, the position continues to age. Afterward, Internet tracking can resume when the local receiver is healthy and a fresh provider position is available.

If local monitoring fails, supplemental NMEA 2000 output pauses while the physical AIS receiver continues through its existing wiring. The [handover design](docs/local-ais-coexistence.md) covers radio gaps, receiver failures, forwarding loops and updates already in transit.

## When a position gets old

A successful download can contain an old position. We keep the time of the vessel's actual position report and apply freshness limits to that time. A new download or a new vessel name leaves the position's age unchanged.

Once the position passes its allowed age, the service stops publishing it. OpenCPN then uses its [lost-target and removal settings](https://opencpn.org/wiki/dokuwiki/doku.php?id=opencpn:manual_advanced:ais). Onboard displays have their own handling, which we'll test alongside the change from Internet to local reception.

Internet coverage and delay depend on the receivers feeding AIS Hub. This wider view helps with advance traffic awareness. Close-quarters decisions continue to use local AIS, radar and lookout.

## Linux first, then Cerbo

The first release will provide a Linux executable, `oth-ais`, serving AIS directly to OpenCPN over TCP. That lets us develop and use the common backend now, while the boat's navigation equipment is off.

Cerbo is the planned initial boat host. Its NMEA 2000 adapter will run in a separate process so it can stop output when the backend stalls and check local AIS immediately before sending a target. We'll limit provider requests, target counts, queues, memory and CAN traffic, and measure the workload alongside the electrical services already running there.

Package installation and resource checks can happen remotely. Physical CAN transmission, target display and local-source handover will be tested with the navigation equipment powered during the next boat visit. Later, the same backend can move to a dedicated boat computer.

## Display compatibility

The output adapters will use standard AIS sentences and NMEA 2000 messages, with established Python libraries for encoding and decoding.

AIS Hub's published fields include position, motion and vessel details; equipment class needs further qualification. We'll inspect the actual account response and make any encoding assumption an explicit setting. Axiom and Orca compatibility will be established by sending known reports and checking what each device displays.

## Design and development

The repository currently contains the design documents. The executable and deployment package are the next implementation steps.

- [Architecture](docs/architecture.md): the Python modules, data paths and resource limits
- [Local AIS coexistence](docs/local-ais-coexistence.md): how a vessel changes source and how we avoid duplicate or reflected reports
- [OpenCPN testing](docs/opencpn-testing.md): desktop setup and the first end-to-end tests
- [Implementation plan](docs/implementation-plan.md): the build order, Cerbo installation and onboard checks
- [Research](docs/research.md): source documents and the questions still requiring equipment tests

## Contributing

The project is MIT-licensed. Contributions from other boat owners and developers are welcome, especially test results from different AIS receivers and displays.

Keep credentials, real vessel captures and private boat configurations outside the public repository. Use synthetic vessels in shared tests. AIS data access and redistribution follow the provider's terms.

A later, opt-in feature can share genuine local radio reception back to AIS Hub, after confirming the account terms and receiver output. Generated Internet targets stay excluded from that upload.
