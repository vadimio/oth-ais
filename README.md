# OTH AIS

One Python backend for Internet and locally received AIS, with a Linux/OpenCPN output and a NMEA 2000 adapter for marine displays.

**Status: design proposal, October 7, 2026. Implementation awaits owner approval.** This repository currently contains the architecture and delivery plan. Boat services and navigation settings remain unchanged.

## Proposed approach

- Deliver a Linux `oth-ais` executable and test the common Python backend with OpenCPN first
- Retrieve regional traffic from AIS Hub through the boat's existing Internet connection
- Keep local VHF AIS as the preferred source for every vessel identified by MMSI
- Serve one merged AIS TCP stream to OpenCPN and expose source/age details through an authenticated API
- Publish supplemental Internet-only AIS through a separately supervised Python N2K adapter
- Deploy the same package on Cerbo, with bounded CPU/RAM/network use and recovery
- Verify physical Cerbo output and Axiom/Orca consumption on the powered boat network
- Offer an opt-in contribution of genuine locally received AIS to AIS Hub

Internet coverage follows participating receivers and provider coverage. Delayed positions support advance traffic awareness. Close-quarters navigation continues to use the vessel's local AIS, radar and lookout.

## Review the design

| Document | Content |
| --- | --- |
| [Architecture](docs/architecture.md) | Common Python backend, Linux/OpenCPN delivery, N2K adapter and resource/failure handling |
| [Local AIS coexistence](docs/local-ais-coexistence.md) | Per-MMSI authority, queue cancellation, radio gaps, source identity and echo prevention |
| [OpenCPN testing](docs/opencpn-testing.md) | Planned desktop connection, replay and real-client acceptance workflow |
| [Implementation plan](docs/implementation-plan.md) | Linux first, virtual CAN, Cerbo deployment and deferred physical/client tests |
| [Research](docs/research.md) | Primary sources, verified API constraints and unresolved equipment compatibility |

The first working client is OpenCPN. Provider class metadata or an explicit compatibility-encoding choice, local-source handover and stale-target behavior receive separate tests before marine output is enabled. Router/VLAN and legacy-display Ethernet work remain separate projects.

## Open-source boundary

The project uses the MIT license. Software, synthetic test fixtures and generic deployment examples belong here. API credentials, receiver-upload destinations, boat locations, vessel identifiers, private captures and live device configurations stay in each operator's private configuration store. Access and redistribution rights for AIS data follow the provider's terms.
