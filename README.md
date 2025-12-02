# Hydrow Drag Fix

An ESP32-based replacement for the Hydrow rower's electromagnetic drag control system.

## The Problem

The Hydrow uses electromagnetic coils to create adjustable resistance. When the control board's switching circuit fails, drag can get stuck at 100%—making the rower essentially unusable.

This project bypasses the failed circuit entirely, using an ESP32 and a pair of MOSFETs to take over drag control while leaving all other Hydrow functions intact.

## Features

- **Adjustable drag** via Home Assistant, web interface, or local potentiometer
- **OLED display** showing current drag percentage and control source
- **Silent operation** (25kHz PWM above audible range)
- **Preset buttons** for common drag levels
- **Emergency stop** function
- **Under $30** in parts

## Parts List

| Component | Quantity | Purpose |
|-----------|----------|---------|
| ESP32 DevKit | 1 | Controller + WiFi |
| IRLZ44N MOSFET | 2 | Switch coil current |
| MBR30100CT | 1 | Dual flyback diode |
| 330Ω resistor | 2 | Gate current limiting |
| 10KΩ resistor | 2 | Gate pull-downs |
| SSD1306 OLED 128×64 | 1 | Local display |
| 10K potentiometer | 1 | Manual control |
| Perfboard | 1 | Assembly |
| TO-220 heatsinks | 2 | For MOSFETs |
| 18AWG wire | ~2ft | Power connections |
| 20AWG wire | ~3ft | Signal connections |

## Wiring

### ESP32 Connections

| ESP32 Pin | Connects To |
|-----------|-------------|
| GPIO25 | 330Ω → Q1 Gate |
| GPIO26 | 330Ω → Q2 Gate |
| GPIO34 | Potentiometer wiper |
| GPIO16 | OLED SDA |
| GPIO17 | OLED SCL |
| 3.3V | OLED VCC, Pot high side |
| GND | Common ground rail |
| USB | 5V power (keep separate from 48V!) |

### Hydrow Connections

| From Hydrow | To Controller |
|-------------|---------------|
| +48V (blue wire) | MBR30100CT Pin 2 (center) |
| Coil 1 Wire A | +48V rail |
| Coil 1 Wire B | Q1 Drain + MBR30100CT Pin 1 (left) |
| Coil 2 Wire A | +48V rail |
| Coil 2 Wire B | Q2 Drain + MBR30100CT Pin 3 (right) |
| Chassis | Ground rail |

### MBR30100CT Pinout

```
    (view from front, pins down)
    
         ┌─────────────┐
         │  MBR30100CT │
         └──────┬──────┘
        ┌───────┼───────┐
        │       │       │
      PIN 1   PIN 2   PIN 3
      (left) (center) (right)
        │       │       │
        ▼       ▼       ▼
    Q1 Drain  +48V   Q2 Drain
```

### MOSFET Circuit (per channel)

```
ESP32 GPIO ──►── 330Ω ──┬──► MOSFET Gate
                        │
                       10KΩ
                        │
                       GND
```

## Installation

### 1. Flash the ESP32

1. Install [ESPHome](https://esphome.io/guides/installing_esphome.html)
2. Copy `esphome/hydrow-drag.yaml` to your ESPHome directory
3. Create/update `secrets.yaml`:
   ```yaml
   wifi_ssid: "YourWiFiName"
   wifi_password: "YourWiFiPassword"
   api_encryption_key: "generate-a-key-here"
   ota_password: "your-ota-password"
   ```
4. Flash via USB: `esphome run hydrow-drag.yaml`

### 2. Build the Controller Board

See schematics in `/schematics` folder for detailed wiring diagrams.

Key points:
- Mount MOSFETs with heatsinks
- Keep 48V and ESP32 power completely separate
- Use 18AWG for all 48V paths
- Double-check MBR30100CT orientation (center pin to +48V)

### 3. Connect to Hydrow

1. **Unplug Hydrow from wall power**
2. **Disconnect** coil connectors from original PCB (leave them unplugged)
3. **Tap** +48V from blue wire feeding coils
4. **Route** coil wires to your controller per wiring table above
5. **Connect** chassis ground to controller ground rail
6. **Mount** controller securely inside Hydrow enclosure

### 4. Test Sequence

1. Power ESP32 via USB only (Hydrow off)
   - Display should show "HYDROW DRAG 0%"
   - Turn pot—percentage should change
2. Connect to Home Assistant—verify slider works
3. Power on Hydrow
4. Set drag to 0%—spin flywheel, should be free
5. Slowly increase drag—feel resistance build

## Calibration

The relationship between controller percentage and Hydrow's native drag scale:

| Controller | Hydrow Native |
|------------|---------------|
| 0% | Minimum (free spin) |
| 71% | 104 (good default) |
| 100% | Maximum |

Adjust to your preference. The relationship is roughly linear but not exact due to electromagnetic characteristics.

## Troubleshooting

### Coils whine/sing
PWM frequency too low. Increase `frequency` in ESPHome config to 25000 Hz or higher.

### No drag at any setting
- Check MOSFET gate connections
- Verify +48V reaching MBR30100CT center pin
- Confirm ground continuity from chassis to controller

### Drag stuck at maximum
- MOSFETs may be shorted—check with multimeter
- Gate pull-downs missing or failed
- ESP32 not running (check USB power)

### Display not working
- Verify I2C pins (SDA=GPIO16, SCL=GPIO17)
- Check OLED power (3.3V, not 5V)
- Try I2C address 0x3D if 0x3C doesn't work

## Safety

- **48V can shock you.** Not lethal for most people, but painful.
- **Always unplug** Hydrow from wall before working inside.
- **This voids your warranty.**
- **You're responsible** for your own safety and equipment.

## License

MIT License - See [LICENSE](LICENSE)

## Acknowledgments

Built with [ESPHome](https://esphome.io/) and [Home Assistant](https://www.home-assistant.io/).
