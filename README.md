# OTH AIS

Internet AIS traffic alongside a vessel's locally received AIS, with explicit source labels, position ages and an optional NMEA 2000 output.

**Status: design proposal, October 7, 2026. Implementation awaits owner approval.** This repository currently contains the architecture and delivery plan. Boat services and navigation settings remain unchanged.

## Proposed approach

- Run a small Linux service aboard, initially on a capacity-qualified Cerbo GX and later on a dedicated boat server
- Retrieve regional traffic from AIS Hub through the boat's existing Internet connection
- Keep local VHF AIS as the preferred source for every vessel identified by MMSI
- Show Internet traffic and its age through an authenticated boat-LAN interface
- Enable plotter output after protocol, source-label, expiry and handover tests on the actual equipment
- Offer an opt-in contribution of genuine locally received AIS to AIS Hub

Internet coverage follows participating receivers and provider coverage. Delayed positions support advance traffic awareness. Close-quarters navigation continues to use the vessel's local AIS, radar and lookout.

## Review the design

| Document | Content |
| --- | --- |
| [Architecture](docs/architecture.md) | Data paths, local-AIS priority, hosting, NMEA 2000 admission and failure handling |
| [Implementation plan](docs/implementation-plan.md) | Phased delivery, acceptance tests and decisions for approval |
| [Research](docs/research.md) | Primary sources, verified API constraints and unresolved equipment compatibility |

The principal plotter-output questions are AIS class metadata and the display's handling of Internet origin and stale targets. The plan addresses these before live transmission.

## Open-source boundary

The project uses the MIT license. Software, synthetic test fixtures and generic deployment examples belong here. API credentials, receiver-upload destinations, boat locations, vessel identifiers, private captures and live device configurations stay in each operator's private configuration store. Access and redistribution rights for AIS data follow the provider's terms.
