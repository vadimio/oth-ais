# OTH AIS (Over-The-Horizon AIS)

OTH AIS is an intelligent marine data manager designed to bridge the gap between real-time local navigation and long-range situational awareness.

AIS receiver on a boat shows the vessels it can hear over VHF. An Internet connection can give you a wider view: ships approaching from farther away, traffic along your route, and vessels reported by receivers beyond your own range. The service exists but usually requires additional hardware equipment and significant monthly subscriptions.

OTH AIS brings the extra traffic into LAN and onto the boat's NMEA 2000 network. This way you can consume it with OpenCPN and other networked tools as well as NMEA 2000 and RayMarine SeatalkNG devices. For us, on board S/V Dash, we are building it for Raymarine Axiom+, Orca Core 2 and OpenCPN. We are using Python for now to make it cross compatible and be able to be installed on regular machines, Raspberry Pi, and/or Victron Cerbo. As with any modern tools we can always reimplement it in other more robust languages, like Rust.

## How it fits aboard

One backend gets nearby traffic from [AIS Hub](https://www.aishub.net/api), listens to the boat's local AIS, and decides which observation to use for each vessel. It provides two outputs:

- **OpenCPN:** a single AIS feed combining Internet traffic with locally received vessels
- **NMEA 2000:** additional Internet targets alongside the local AIS receiver's existing reports

The receiver keeps its direct connection to the boat's instruments. A separate status API shows where each observation came from, how old its position is, and why the service is displaying or withholding it.

## When local AIS picks up the same vessel

Imagine a ship first appears through AIS Hub. Later, your receiver hears it over VHF. Both reports carry the same MMSI, the vessel's AIS identifier. From that moment, local reception takes priority. OpenCPN gets the local observation, and OTH stops sending Internet updates for that ship onto NMEA 2000.

If we lose the local position reports for that vessel, we still want to know where it is. We fall back to the Internet position after five minutes for a moving vessel, or nine minutes for a vessel known to be anchored or moored, assuming we have a fresh OTH Internet position. These are configurable time windows that give intermittent reception room to recover. Actual [radio reporting intervals](https://www.navcen.uscg.gov/types-of-ais) vary with the vessel's speed and AIS equipment. When its movement status is unclear, we use the five-minute window.

If the local AIS receiver fails, we use fresh Internet positions as soon as they are available. If we lose our monitoring connection and the receiver's condition is uncertain, we keep the same per-vessel timers and continue Internet tracking as they expire. The service reports the problem and shows which source we are using. Fresh local reception takes priority again as soon as it returns.

Each Internet replacement must be within our configured age limit and newer than the last local position. We can use a valid position already in memory. The [handover design](docs/local-ais-coexistence.md) explains the timing, source selection and handling of duplicate reports.

## When a position gets old

A vessel can appear in every Internet download while its position stays the same for hours. We therefore keep the time of its actual position report and check its age before sending it to our displays. Receiving its name, class or other details leaves the position's age unchanged. The five- and nine-minute handover windows follow local position reports for the same reason.

If both sources have lost the vessel and its position has become too old, we stop sending it to the displays. OpenCPN uses its [lost-target and removal settings](https://opencpn.org/wiki/dokuwiki/doku.php?id=opencpn:manual_advanced:ais). Our onboard tests check how Axiom and Orca handle this.

Internet coverage depends on the receivers feeding AIS Hub, and their data can arrive with a delay. It gives us a wider view of traffic along our route. When vessels get close, we continue to rely on local AIS, radar and our own lookout.

## Linux first, then Cerbo

We are building a Linux executable, `oth-ais`, that sends AIS to OpenCPN over TCP, with Docker as our first deployment. We can run it on a regular computer and work with real Internet traffic while the boat's navigation equipment is off. This is also where we test the change between local and Internet positions.

On S/V Dash, Cerbo is our first-release boat host. Cerbo already handles our electrical systems with private services written by us, so we keep OTH's CPU, memory, Internet use and CAN traffic within configured limits. The NMEA 2000 adapter runs separately and checks the selected source just before sending each vessel report. If the main program stalls, that adapter stops its output.

We can install the package and measure its workload remotely. Onboard testing takes place at the next boat visit: we turn on the navigation equipment and check that Cerbo can send the data, Axiom and Orca display it, and local AIS takes over when it hears the same vessel. In a later release, we will move the same backend to our dedicated boat computer.

## Display compatibility

We use standard AIS sentences and NMEA 2000 messages.
Other navigation tools should be able to consume the same outputs.

AIS Hub provides position, speed, course and vessel details.
> Note: We still need to check whether our account provides the AIS equipment class. That affects which message we send. We make any assumption visible in the settings and test the resulting reports with OpenCPN, Axiom and Orca.

## Design and development

- [Architecture](docs/architecture.md): the Python modules, data paths and resource limits
- [Local AIS coexistence](docs/local-ais-coexistence.md): how a vessel changes source and how we avoid duplicate or reflected reports
- [OpenCPN testing](docs/opencpn-testing.md): desktop setup and the first end-to-end tests
- [Implementation plan](docs/implementation-plan.md): the build order, Cerbo installation and onboard checks
- [Research](docs/research.md): source documents and the questions still requiring equipment tests

## Contributing

The project is MIT-licensed. We welcome other boat owners and developers using it, improving it and sharing results from their AIS receivers and displays.

Credentials and private boat configurations stay outside this repository. For shared tests, we use made-up vessels and positions. Real AIS data remains subject to the provider's access and redistribution terms.

In a later release, we will add optional sharing of the vessels our own receiver hears, after we confirm the account terms and identify a clean receiver feed. Only genuine radio reception will go back to AIS Hub.

## Licensing

The project is MIT-licensed.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
