# ESPS3BLEBLACKBOX User Manual

## 1. Overview

This firmware is an integrated system based on the ESP32-S3 Supermini that combines BLE communication and blackbox data logging. It merges two independently functioning modes, `ble.ino` and `blackbox.ino`, into a single firmware, where modes are automatically switched during the initial idle state.

### Arduino IDE Setup

1. Go to **File -> Preferences -> Additional Board Manager URLs** and add:

   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```

2. Go to **Sketch -> Include Library -> Manage Libraries** and install the following 2 libraries:

   - **Arduino AVR Boards**
   - **esp32 by Espressif Systems**

3. In **Tools**, change 2 settings:
   - **PSRAM** -> `QSPI SRAM`
   - **USB Mode** -> `USB OTG (TinyUSB)`

4. Connect the ESP32-S3 Supermini via USB and upload the sketch.

---

## 2. Operating Modes

### 2.1 Initial/Idle State (MODE_IDLE)

- **UART Baud Rate**: 1,500,000 (default, changeable via `config.txt`)
- **BLE Advertising**: Advertises and waits for a BLE connection.
- **LED**: Yellow and green blink alternately at 0.5-second (500ms) intervals.
- **SD Card Space Management**:
  - Automatically cleans the SD card based on the capacity settings in `config.txt`.
  - If free space drops below the configured minimum, low-numbered `.bbl` files are deleted to free space up to the configured maximum.
  - This function **only runs in idle state** and stops when switching to another mode.

### 2.2 BLE Connection (MODE_BLE_CONFIG)

- **Entry Condition**: Enters immediately when a BLE client connects.
- **UART Baud Rate**: Changes to 115,200.
- **Function**: Processes FC-BLE Configurator data using the high-speed Batch Notify algorithm and Dynamic MTU calculation.
- **LED**: Yellow stays solid.
- **Return**: Automatically returns to MODE_IDLE when the connection is released.

### 2.3 Blackbox Recording Mode (MODE_BLACKBOX)

- **Entry Condition**: Enters when 16KB or more of data is received into the ring buffer at 1,500,000 Baud from the idle state.
- **BLE Block**: Upon entry, immediately calls `BLEDevice::getAdvertising()->stop()` to block BLE access.
- **Recording**: The `recordTask` takes over data from the PSRAM ring buffer and saves it to the SD card as `/log0000.bbl`.
- **LED Sequence**:
  1. Green for 1 second (mode transition indication)
  2. Blue for 1 second (data reception OK indication)
  3. Red blinking (recording in progress indication)
- **Exit**: When data reception stops for 5 seconds (timeout), the file is finalized, BLE blocking is released, and the system returns to MODE_IDLE.

### 2.4 USB MSC Mode (MODE_USB_MSC)

- **Entry Condition**: Enters when the boot button (GPIO 0) click is detected.
- **Operation**:
  - Stops BLE communication (`BLEDevice::deinit`).
  - Finalizes any ongoing SD recording and closes UART.
  - Enters USB Mass Storage Class mode, allowing the SD card to be accessed directly from a PC.
- **LED**: White stays solid.
- **Return**: While in this mode, pressing the boot button terminates or completes any ongoing file read/write and reboots back to the idle state.

---

## 3. config.txt Configuration

Create a `config.txt` file at the root of the SD card to change system settings.

### 3.1 File Format

- **1st Line**: Minimum free space to maintain (in MB)
- **2nd Line**: Maximum free space to maintain (in MB)
- **3rd Line**: UART Baud Rate
- **4th Line and beyond**: Treated as comments; not read or deleted.

### 3.2 Baud Rate Options

Use one of the following values. An invalid value will be corrected to the default `1500000`.

| Value     | Description              |
|-----------|--------------------------|
| `921600`  | Standard serial speed    |
| `1000000` | High-speed serial        |
| `1500000` | Default                  |
| `2000000` | Maximum speed            |

### 3.3 Default Values

If `config.txt` is missing or a read error occurs, the file is recreated with the following default values:

```
100
500
1500000
1. minimum auto free space
2. maximum auto free space
3. baud rate = 921600 100000 150000 200000
# cli / serialpassthrough (port-1) (baud rate)
```

- When free space drops below 100 MB, files are deleted to free up to 500 MB.
- The system operates at 1,500,000 baud.
- Low-numbered files are deleted first; deletion pauses during BLE connection or blackbox operation.
- **Note**: If `log9999.bbl` exists, files are deleted in descending order. Use caution.

### 3.4 Space Management Rules

- MB values are capped at a maximum of 9999; values above 9999 are treated as 9999.
- If an unreadable value is encountered, lines 1 and 2 are rewritten as `100` and `500` respectively, and the system operates with defaults.
- Comments are left unchanged.
- The file is modified, not deleted.

### 3.5 File Deletion Logic

- When free space on the SD card drops below the configured minimum, low-numbered `.bbl` files are selected and their sizes are summed.
- Once the summed size exceeds 400 MB, counting stops and those files are deleted in bulk.
- This ensures older data (lower file numbers) is cleaned first.

---

## 4. Operation Flow Summary

```
[Idle State] ── BLE Connect ──→ [BLE Config Mode]
     │                             │
     │ ← Auto Return on Disconnect ←│
     │                             │
     └─ 16KB Data Received ──→ [Blackbox Recording Mode]
                                      │
                                5s Timeout
                                      │
                                 Recording End
                                      │
                                 [Idle State] ←──┘

[Boot Button] ──→ [USB MSC Mode] ──→ [Reboot] ──→ [Idle State]
```

---

## 5. LED Status

| Mode                      | LED Status                  |
|---------------------------|-----------------------------|
| MODE_IDLE                 | Yellow/Green blink @ 0.5s   |
| MODE_BLE_CONFIG           | Yellow solid                |
| MODE_BLACKBOX (transition)| Green for 1s                |
| MODE_BLACKBOX (RX OK)     | Blue for 1s                 |
| MODE_BLACKBOX (recording) | Red blinking                |
| MODE_USB_MSC              | White solid                 |

---

## 6. Hardware Connections

- **UART**: Uses the same pin connections as the blackbox mode.
- **Boot Button**: Connected to GPIO 0.
- **PSRAM**: Used as a ring buffer to support stable data recording even on slow SD cards.
- **Pinout**:

  | Function      | GPIO   | Description                |
  |---------------|--------|----------------------------|
  | FC UART RX    | GPIO 9 | Receive data from FC       |
  | FC UART TX    | GPIO 8 | Transmit data to FC        |
  | SD SPI SCK    | GPIO 13| SD clock                   |
  | SD SPI MOSI   | GPIO 11| SD data output             |
  | SD SPI MISO   | GPIO 12| SD data input              |
  | SD SPI CS     | GPIO 10| SD chip select             |
  | RGB LED       | GPIO 48| Onboard WS2812             |
  | Boot Button   | GPIO 0 | Enter USB MSC mode         |

- SD Card pin numbering (short pin = pin 1):

  **Standard Micro SD (9-pin):**
  ```
  1-short / 2-CS / 3-MOSI / 4-GND / 5-+V / 6-SCK / 7-GND / 8-MOSI / 9
  ```

  **Micro SD (8-pin, no pin 4 GND):**
  ```
  1 / 2-CS / 3-MOSI / 4-+V / 5-SCK / 6-GND / 7-MOSI / 8
  ```

---

## 7. Test Results

- Confirmed stable transmission of **3KB data** at **1,500,000 baud**.
- Operates reliably on a slow **2GB SD card** by leveraging the ESP-S3 buffer.
- Faster SD cards are expected to yield higher recording speeds (testing required).

---

## 8. Notes

- Line ending characters (`\r`) in `config.txt` are automatically removed.
- Leading and trailing whitespace and tab characters are stripped before processing.
- BLE mode and blackbox mode do not run simultaneously; activating one mode pauses the other.
- BLE advertising is stopped during SD card recording, so new connections cannot be accepted.
