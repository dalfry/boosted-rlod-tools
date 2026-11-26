# Boosted Board XR RLOD Reset Utility

A Python script that communicates with a **Boosted Board XR electric scooter battery** over CAN bus to:

- **Reset RLOD (Ride Light Off / cell voltage imbalance) errors** — usually caused by cell voltage imbalance
- **Report cell voltages** — shows the voltage of every cell as reported by the TI Battery Monitor (AFE)
- **Generate a battery recharge report** — output can be parsed to identify the weakest cell(s) in the pack that consistently lag behind others

## What It Does

The script connects to the battery's CAN interface and:

![Boosted Board XR accessory port CAN bus connection](https://canada1.discourse-cdn.com/flex030/uploads/saleae/optimized/1X/fec39a82bbf38dc4958bed12216995dcae777db0_2_452x500.jpeg)

*Image credit: [Saleae Forum — Hacking the Boosted Accessory Port](https://discuss.saleae.com/t/hacking-the-boosted-accessory-port/522)*

1. Waits for the battery to boot and start sending CAN messages
2. Enables message routing from the battery to the CAN bus
3. Sends periodic keep-alive messages so the battery thinks an ESC is connected
4. Listens for cell voltage data, then queries the battery via `GETAFECELLS`
5. Prints the lowest, highest, total, and delta between cells
6. If the cell delta is under 500mV, it sends `PFAILRESET` and `REBOOT` to clear the RLOD error
7. Outputs all data to stdout (redirectable to a file) for later analysis

> **⚠️ WARNING:** This script clears *all* faults. If a cell is severely out of spec (e.g. 1.5V), the reset may not fix the issue and the battery will return to RLOD on the next power cycle. Use at your own risk.

## Dependencies

### Python packages

```bash
pip3 install python-can
```

That's the only external dependency. All other modules (`argparse`, `sys`, `time`, `datetime`, `struct`) are part of the Python standard library.

### Hardware

- A **CAN-to-USB adapter** (slcan-compatible). Examples:
  - Any USB-to-CAN dongle with slcan firmware
  - An Arduino or ESP32 running slcan firmware
  - A Peak-PCAN, Kvaser, or similar adapter with slcan driver support
- A **Boosted Board XR battery** connected to the CAN bus via the adapter

Connect the adapter to your computer, note the device path (e.g. `/dev/ttyACM0` on Linux, `/dev/tty.usbmodem1101` on macOS), and pass it to the script.

## Usage

### Basic usage (Linux)

```bash
python3 -u boosted-rlod-fix-report.py -c /dev/ttyACM0
```

### Save output to a file for later analysis

```bash
python3 -u boosted-rlod-fix-report.py -c /dev/ttyACM0 >> xr_battery_datapoints.txt
```

The resulting `xr_battery_datapoints.txt` can be parsed to generate a battery recharge report by monitoring the voltage of each cell over time. Cells that consistently lag behind others are the weakest and may need replacement.

### Command-line options

| Flag | Description | Default |
|------|-------------|---------|
| `-i, --interface` | CAN interface type (python-can `bustype`) | `slcan` |
| `-c, --channel` | Serial port path of the CAN adapter | `/dev/tty.usbmodem1101` |
| `-p, --power-off` | Power off the battery when the script exits | (disabled) |

### Examples

```bash
# Linux — USB-to-CAN adapter on /dev/ttyACM0
python3 -u boosted-rlod-fix-report.py -c /dev/ttyACM0

# Linux — USB-to-CAN adapter on /dev/ttyUSB0
python3 -u boosted-rlod-fix-report.py -c /dev/ttyUSB0

# macOS — Arduino-based slcan adapter
python3 -u boosted-rlod-fix-report.py -c /dev/tty.usbmodem1101

# With output saved to a file
python3 -u boosted-rlod-fix-report.py -c /dev/ttyACM0 >> xr_battery_datapoints.txt

# Power off the battery after completing (advanced)
python3 -u boosted-rlod-fix-report.py -c /dev/ttyACM0 -p
```

### Using a different CAN interface type

If you're not using an slcan-based adapter (e.g. PCAN, SocketCAN, etc.), specify the interface type:

```bash
python3 -u boosted-rlod-fix-report.py -i socketcan -c can0
```

See the [python-can documentation](https://python-can.readthedocs.io/en/stable/configuration.html) for all supported interface types and configuration options.

## Original Source

Originally from [beambreak.org](https://beambreak.org) / [github.com/rscullin/beambreak](https://github.com/rscullin/beambreak)

Modified by @dalfry (VaibhaV Sharma) in 2026 to focus on RLOD reset and battery cell voltage reporting.

## License

MIT License — Copyright (c) 2023 Robert Scullin
