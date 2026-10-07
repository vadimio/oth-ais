# OTH AIS (Over-The-Horizon AIS)

OTH AIS is an intelligent marine data manager designed to bridge the gap between real-time local navigation and long-range situational awareness.

AIS receiver on a boat shows the vessels it can hear over VHF. An Internet connection can give you a wider view: ships approaching from farther away, traffic along your route, and vessels reported by receivers beyond your own range. The service exists but usually requires additional hardware equipment and significant monthly subscriptions.

OTH AIS brings the extra traffic into LAN and onto the boat's NMEA 2000 network. This way you can consume it with OpenCPN and other networked tools as well as NMEA 2000 and RayMarine SeatalkNG devices. For us, on board S/V Dash, we intend to use it with Raymarine Axiom+, Orca Core 2 and OpenCPN. We are using Python for now to make it cross compatible and be able to be installed on regular machines, Raspberry Pie, and/or Victron Cerbo. of course, with modern tools we can always reimplement it in other more robust languages, like Rust.

## How it fits aboard

One Python backend gets nearby traffic from [AIS Hub](https://www.aishub.net/api), listens to the boat's local AIS, and decides which observation to use for each vessel. It provides two outputs:

- **OpenCPN:** a single AIS feed combining Internet traffic with locally received vessels
- **NMEA 2000:** additional Internet targets alongside the local AIS receiver's existing reports

The receiver keeps its direct connection to the boat's instruments. A separate status API shows where each observation came from, how old its position is, and why the service is displaying or withholding it.

## When local AIS picks up the same vessel

Imagine a ship first appears through AIS Hub. Later, your receiver hears it over VHF. Both reports carry the same MMSI, the vessel's AIS identifier. From that moment, local reception takes priority. OpenCPN gets the local observation, and OTH stops sending Internet updates for that ship onto NMEA 2000.

If we lose the local position reports for that vessel, we still want to know where it is. We will return to the Internet position after five minutes for a moving vessel, or nine minutes for a vessel known to be anchored or moored, assuming we have a fresh Internet position. These are configurable time windows. Actual [radio reporting intervals](https://www.navcen.uscg.gov/types-of-ais) vary with the vessel's speed and AIS equipment. When its movement status is unclear, we use the five-minute window.

If the local AIS receiver fails, we will use fresh Internet positions as soon as they are available. If we lose our monitoring connection and the receiver's condition is uncertain, we will keep the same per-vessel timers and continue Internet tracking as they expire. The service will report the problem and show which source we are using. Fresh local reception takes priority again as soon as it returns.

Each Internet replacement must be within our configured age limit and newer than the last local position. We can use a valid position already in memory. The [handover design](docs/local-ais-coexistence.md) explains the timing, source selection and handling of duplicate reports.

## When a position gets old

A vessel can appear in every Internet download while its position stays the same for hours. We therefore keep the time of its actual position report and check its age before sending it to our displays. Receiving its name, class or other details leaves the position's age unchanged. The five- and nine-minute handover windows follow local position reports for the same reason.

If both sources have lost the vessel and its position has become too old, we will stop sending it to the displays. OpenCPN uses its [lost-target and removal settings](https://opencpn.org/wiki/dokuwiki/doku.php?id=opencpn:manual_advanced:ais). We will check how Axiom and Orca handle this as part of our onboard tests.

Internet coverage depends on the receivers feeding AIS Hub, and their data can arrive with a delay. It gives us a wider view of traffic along our route. When vessels get close, we continue to rely on local AIS, radar and our own lookout.

## Linux first, then Cerbo

We will start with a Linux executable, `oth-ais`, that sends AIS to OpenCPN over TCP. We can run it on a regular computer and work with real Internet traffic while the boat's navigation equipment is off. This is also where we will test the change between local and Internet positions.

On Dash, we intend to run the service on Cerbo until we have a dedicated boat computer. Cerbo already handles our electrical systems, so we will keep OTH's CPU, memory, Internet use and CAN traffic within configured limits. The NMEA 2000 adapter will run separately and check the selected source just before sending each vessel report. If the main program stalls, that adapter will stop its output.

We can install the package and measure its workload remotely. At the next boat visit, we will turn on the navigation equipment and check that Cerbo can send the data, Axiom and Orca display it, and local AIS takes over when it hears the same vessel. Later, we can move the same backend to our dedicated boat computer.

## Display compatibility

We will use standard AIS sentences and NMEA 2000 messages, with existing Python libraries to encode and decode them. Other navigation tools should be able to consume the same outputs; we will record which devices and software versions we have actually tested.

AIS Hub provides position, speed, course and vessel details. We still need to check whether our account provides the AIS equipment class. That affects which message we send. We will make any assumption visible in the settings and test the resulting reports with OpenCPN, Axiom and Orca.

## Design and development

For now, this repository contains our design and implementation plan. Next, we will build the Linux service and get it working with OpenCPN, then add and test the NMEA 2000 output.

- [Architecture](docs/architecture.md): the Python modules, data paths and resource limits
- [Local AIS coexistence](docs/local-ais-coexistence.md): how a vessel changes source and how we avoid duplicate or reflected reports
- [OpenCPN testing](docs/opencpn-testing.md): desktop setup and the first end-to-end tests
- [Implementation plan](docs/implementation-plan.md): the build order, Cerbo installation and onboard checks
- [Research](docs/research.md): source documents and the questions still requiring equipment tests

## Contributing

The project is MIT-licensed. We would welcome other boat owners and developers using it, improving it and sharing results from their AIS receivers and displays.

Credentials and private boat configurations stay outside this repository. For shared tests, we use made-up vessels and positions. Real AIS data remains subject to the provider's access and redistribution terms.

We can also give back to the AIS community by sharing the vessels our own receiver hears. That will be optional, after we confirm the account terms and identify a clean receiver feed. Only genuine radio reception will go back to AIS Hub.
