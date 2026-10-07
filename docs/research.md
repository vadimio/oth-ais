# Research and evidence

Primary references checked October 7, 2026. Provider documentation establishes an intended contract; account responses and equipment tests establish deployed behavior.

| Source | Finding | Consequence |
| --- | --- | --- |
| [AIS Hub API](https://www.aishub.net/api) | Regional JSON access, a once-per-minute request limit, provider timestamps and vessel-type metadata; AIS class/message ID absent from the documented record | Central request limiter and a class-evidence qualification step |
| [AIS Hub sharing workflow](https://www.aishub.net/) | Members contribute a receiver feed through a provider-assigned UDP destination; aggregate access follows feed validation | Confirm mobile-station acceptance and account-specific terms |
| [USCG AIS message definitions](https://www.navcen.uscg.gov/ais-messages) | Position-message types distinguish station classes and specialized stations | Preserve class/message provenance and initially limit remote output to ordinary vessels |
| [USCG Class A reports](https://www.navcen.uscg.gov/ais-class-a-reports) | AIS timestamps carry seconds within a minute; radio speed uses 1023 for unavailable; long-range message 27 has distinct station/latency semantics | Preserve full source time internally; qualify sentinel mapping and long-range alternatives |
| [CANboat PGN documentation](https://canboat.github.io/canboat/canboat.html) | AIS position/static PGN layouts, device NAME and fast-packet framing; the long-range PGN includes unconfirmed interpretation | Independently decode output and qualify PGN 129813 separately |
| [OpenCPN connections](https://opencpn.org/wiki/dokuwiki/doku.php?id=opencpn:manual_basic:set_options:connections) | Network TCP/UDP and NMEA 0183 input are supported | Use AIS over TCP as the first working client interface |
| [pyais](https://github.com/M0r13n/pyais) | MIT-licensed Python AIS encoding/decoding with multi-sentence and network-stream support | Preferred NMEA 0183/OpenCPN codec |
| [Python nmea2000](https://github.com/tomer-w/nmea2000) | Apache-2.0 encoder/decoder; upstream `N2KDevice` provides address claiming and sending | Preferred Python N2K foundation; audit and test the exact packaged release |
| [python-can](https://python-can.readthedocs.io/) | Python CAN transports, including Linux SocketCAN | Reuse the transport family already present in boat navigation collection; retain dependency licensing |
| [NMEA2000 C++ library](https://github.com/ttlappalainen/NMEA2000) | MIT-licensed protocol stack with device-management and marine-message support | An established alternative behind the same adapter contract |
| [NMEA2000 SocketCAN driver](https://github.com/ttlappalainen/NMEA2000_socketCAN) | Linux CAN adapter for the C++ stack; source files carry permissive license notices | Alternative requires review and the correction recorded below |
| [em-trak B900 manual](https://productsupport.em-trak.com/hc/en-gb/articles/28856300686749-B900-Series-user-manual-in-English) | B923 belongs to the supported family; USB, NMEA 0183 and NMEA 2000 interfaces; externally supplied data can be multiplexed | Qualify a receive-only raw AIS output and forwarding provenance |
| [Raymarine third-party Ethernet networking](https://docs.raymarine.com/87443/en-US/latest/IPNetworkingOfRaymarineDevicesWithT-4639FB01.html) | Modern Raymarine network uses 198.18.0.0/21, reserved self-addresses and specific DHCP roles | Keep navigation discovery local and commission the router separately |
| [Raymarine networking constraints](https://docs.raymarine.com/87443/en-US/latest/NetworkingConstraints-7F8304A6.html) | Supported display networks have a data master and share supported NMEA 2000 data over Ethernet | Qualify the display network and its data-master role |
| [Raymarine legacy replacement guidance](https://forum.raymarine.com/showthread.php?tid=4324) | Manufacturer support guidance excludes E-Series Classic from an Axiom Ethernet network | Record the vendor support constraint; actual mixed-network failure behavior remains untested and separate from OTH |
| [Signal K source priority](https://github.com/SignalK/signalk-server/blob/master/docs/setup/source-priority.md) | Source-labelled observations and device NAME-based identities support source selection | Optional future adapter, with its own version-specific consumer tests |

The legacy Raymarine support page was available through the search index; direct retrieval returned an error. Dash's owner photo identifies E120 Classic, app v5.69; owner reports the two legacy units share this compatibility. Their actual Ethernet role and mixed-network behavior await a separate network investigation. OTH's Linux/OpenCPN and N2K work proceeds independently.

## Existing Python collection and candidate transmit support

The boat navigation source currently pins `nmea2000==2026.8.1` and `python-can==4.6.1` for passive decoding. Its AIS handling counts recent MMSIs; the revised service needs full observations, receiver NAME authority and revision-based suppression. Source existence establishes a reuse opportunity; activation and physical transmit behavior require their respective runtime tests.

Upstream Python `nmea2000` at commit `1624054b9303dacade05ff08060a5bd0299197b8` includes `N2KDevice`, an encoder and device lifecycle tests. Static inspection shows ISO address-claim requests/announcements at startup, product information and heartbeat behavior. This supplies a Python implementation candidate and establishes that active-device startup emits management traffic. Passive collection must use the receive-only decoder/transport path. Exact packaged-release contents, collision behavior, AIS field mapping and resource limits remain subject to dependency tests. See [the inspected device source](https://github.com/tomer-w/nmea2000/blob/1624054b9303dacade05ff08060a5bd0299197b8/nmea2000/device.py).

## Dependency review finding

The SocketCAN driver at revision `7bdece672938a2406de5116b5d45dad54e1ebfa8` contains an out-of-bounds terminator assignment, `ifr.ifr_name[sizeof(ifr.ifr_name)]`, in `CANOpen()`. Static source inspection identified this finding; runtime impact remains untested. Use a reviewed correction or another qualified adapter before adoption. See the [pinned driver source](https://github.com/ttlappalainen/NMEA2000_socketCAN/blob/7bdece672938a2406de5116b5d45dad54e1ebfa8/NMEA2000_SocketCAN.cpp).

## Deployment evidence

A read-only inspection of the intended initial Cerbo host on October 7 found ARMv7, Python 3.12.13, two logical CPUs, approximately 1 GiB total RAM and about 646-648 MiB available. It also found separate 250 kbit/s navigation and 500 kbit/s BMS interfaces. The navigation interface reported an error-passive state, and the staged passive navigation collector remained unactivated.

This snapshot supports Cerbo as the intended initial boat host after normal installation/resource verification. Linux/OpenCPN development and virtual-CAN tests proceed first. Actual Cerbo CAN transmission and client consumption require the healthy powered navigation network during the next boat visit. OTH throughput, memory use and electrical-service coexistence await implementation measurements. Private deployment details and raw diagnostics remain in the vessel workspace.

## Chart integration boundary

The companion chart workflow has acquired and exported Caribbean nautical and raster overlays for desktop use. Its native LightHouse/Axiom admission and rendering path remains under separate investigation. OTH AIS depends on georeferenced positions and a compatible display interface; chart licensing and native admission are tracked by the chart project.
