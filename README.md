# NavPlotter Segnalatore

A beacon for **buoys and other objects left alone** (fishing gear, a tender, a mooring), on a **Heltec WiFi LoRa 32 V3 (868 MHz)** with an external GPS module. It sends its position over LoRa in the packet all NavPlotters understand, so that the plotters and nodes of the family show it on their charts, with the "object" symbol (a diamond).

**Not for navigation, and not a safety device.**

## Buoy mode

In the Setup page, with the node type set to **object**:

- **Switch the WiFi off after power-up** (5, 10 or 30 minutes): the time you have to set it up. The WiFi never goes off while a phone is connected.
- **Then sleep, and send the position every** 1, 2, 5, 10, 15, 30 or 60 minutes.

What happens then:

1. At power-up the beacon starts normally, with WiFi and display; the display counts down to the sleep.
2. Then everything is switched off. At each wake-up the GPS and the radio come on, the beacon waits for a good position (2 minutes at most), sends it and goes back to sleep.
3. The white LED tells what happened: one flash = awake; then three quick flashes = position sent, or one long flash = no position (no GPS fix in time).
4. The **PRG** key wakes it for set-up: it restarts with its WiFi and a new countdown.

The beacon tells the others how often it speaks, so that they keep it on the chart between two packets instead of taking it for lost: grey after one interval and a half, removed after three. This needs the 2026-10-13 firmware, or later, on the receiving plotters and nodes.

While it sleeps a beacon does not relay the other nodes.

### Battery

The GPS is what uses the most. Wired to the **3V3** pin it stays on while the board sleeps: a position is ready within seconds at every wake-up, but the saving is only partial. Wired to the **Ve** pin it is switched off with the rest: almost nothing is used while asleep, but every wake-up starts the GPS from cold and takes longer. Battery life has not been measured yet.

### From the computer

Without a phone, on the USB serial port (115200): `show`, `set type 0|1|2`, `set wifioff <minutes>`, `set sleep <0..7>` (0 = stay awake; 1 to 7 = 1, 2, 5, 10, 15, 30, 60 minutes). The node restarts after a change.

## Awake, it is a complete node

Until it goes to sleep, or always if it is not set as a sleeping object, the beacon does everything a NavPlotter node does:

- reads the GPS;
- exchanges name, position, course and speed with the other nodes over LoRa, and relays packets (mesh);
- computes the collision alarm (closest point of approach);
- creates its own WiFi network and serves a web app.

The user interface is written in English and translated into Italian; the language is chosen in the Setup page.

## Using it

1. Power the board. The OLED shows the WiFi network name and the address.
2. Join the WiFi network `NavPlotter-<node name>` (default password `navplotter`, change it in Setup).
3. Open `http://192.168.4.1` in the browser. It can be added to the home screen.

Pages:

| Page | What it does |
|---|---|
| Map | own position, other nodes, waypoints, line to the destination. Drag to pan, pinch or +/− to zoom, the ring button follows the boat. Tap a node or waypoint for details, long press to add a waypoint. |
| Compass | heading-up compass with the bearing of the destination. Heading is the GPS course. |
| Nearby | list of received nodes with distance, bearing, closest approach and signal. |
| Waypoints | add, rename, delete, go to. The Routes tab builds ordered lists of waypoints; following a route moves the destination to the next waypoint on arrival (within 0.05 NM) and shows the distance still to go. Stored on the board. |
| Setup | language, node name, node type, alarm range, alarm sound, chart source, WiFi password, status. |

### Connection

The Setup page offers three scenarios:

| Scenario | Node | Phone | Charts |
|---|---|---|---|
| Off-grid | creates its own WiFi | joins the node WiFi | SD card on the node |
| Node WiFi, online charts | creates its own WiFi | joins the node WiFi, uses mobile data for charts | online |
| Boat WiFi | joins the boat WiFi (name and password entered in Setup) | joins the boat WiFi | online, through the boat router |

On the boat WiFi the node is reached at `http://<node name>.local` or at the address shown on its display. The node's own WiFi stays on in every scenario, so the node can always be reached at `http://192.168.4.1` even if the boat WiFi is off or the password is wrong. Choosing a scenario presets the chart source, which can still be changed.

### Privacy

The **Share my position over LoRa** option in Setup decides whether the node transmits its own position. When it is off the node is invisible to the others, but it still receives them, raises the collision alarm and relays their packets through the mesh.

### Node type

A node can be a **boat**, a **person** or an **object** (for example a crew member's beacon, a tender, a buoy). The type is sent over LoRa and each type has its own symbol: arrow, ring with a dot, diamond.

### Alarms

The collision alarm fires when a node inside the **alarm range** (3, 6 or 12 NM) is predicted to pass closer than 0.1 NM within 5 minutes. The banner turns red and, if enabled, the phone beeps and vibrates (tap the page once after opening it, browsers only allow sound after a touch). The **Silence** button on the red banner mutes the current alarm only, without changing the Setup option: it lasts while those nodes stay in alarm, and a new node in alarm sounds again.

### Charts

- **Online**: OpenStreetMap tiles, needs internet on the phone.
- **SD card on the node**: tiles in `/tiles/z/x/y.jpg`, the same card layout as the T-Deck. Needs a micro SD SPI module.
- **None**: plain background.

## Wiring

| Function | Board pin |
|---|---|
| GPS module RX | 47 (board TX) |
| GPS module TX | 48 (board RX) |
| SD module SCK / MOSI / MISO / CS (optional) | 5 / 6 / 7 / 4 |

The GPS speed is detected automatically.

## Building

```
~/.platformio/penv/bin/pio run -t upload
```

The web app is `web/index.html`; `tools/embed_web.py` compresses it into `src/web_page.h` before every build.
`include/traffic_packet.h` (packet, radio settings, mesh) must stay identical to the copies in the T-Deck plotter and in the simulators.

## Not done yet

SD card tiles tested on real hardware, NMEA output for other navigation apps.
