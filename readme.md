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

---

# ESPS3BLEBLACKBOX 사용자 매뉴얼

## 1. 개요

본 펌웨어는 ESP32-S3 supermini 기반 BLE 통신과 블랙박스 데이터 로깅 기능을 통합한 시스템입니다. 

1. File -> Preference ->additional board manager urls 에 아래를 입력하십시오.

 https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

2. Sketch -> Include library -> Manage library 에서 다음 2개의 library 를 설치하세요.

   Arduino AVR Boards , esp32 by Espressif Systems

3. Tools 에서 2개 설정 변경 : Psram ->  QSPI Sram  / USB Mode -> USB OTG(Tiny USB)

4. Esp32 S3 supermini를 usb 로 연결해 업로드 

---

## 2. 동작 모드

### 2.1 초기/대기 상태 (MODE_IDLE)

- **UART Baud Rate**: 1,500,000 (기본값, config.txt 변경 가능)
- **BLE Advertising**: 이름 점유 및 검색을 구동하며 접속을 대기합니다.
- **LED**: 노랑색과 녹색이 0.5초(500ms) 간격으로 점멸합니다.
- **SD 카드 용량 관리**:
  - `config.txt`에 설정된 용량 기준으로 SD 카드 자동 정리를 수행합니다.
  - 용량이 설정된 최소값 이하로 떨어지면 낮은 번호의 `.bbl` 파일을 삭제하여 최대 확보 용량까지 공간을 확보합니다.
  - 이 기능은 **대기 상태에서만 실행**되며, 다른 모드로 전환되면 중지됩니다.

### 2.2 BLE 접속 시 (MODE_BLE_CONFIG)

- **진입 조건**: BLE 클라이언트가 연결되면 즉시 진입합니다.
- **UART Baud Rate**: 115,200으로 변경됩니다.
- **기능**: FC-BLE Configurator 데이터를 고속 Batch Notify 알고리즘과 Dynamic MTU 계산으로 처리합니다.
- **LED**: 노란색이 고정되어 표시됩니다.
- **복귀**: 접속이 해제되면 자동으로 MODE_IDLE로 복구됩니다.

### 2.3 블랙박스 기록 모드 (MODE_BLACKBOX)

- **진입 조건**: 대기 상태에서 1,500,000 Baud로 16KB 이상의 데이터가 링버퍼에 수신되면 진입합니다.
- **BLE 차단**: 진입 즉시 `BLEDevice::getAdvertising()->stop()`을 호출하여 BLE 접근을 차단합니다.
- **기록**: `recordTask`가 PSRAM 링버퍼로부터 데이터를 인계받아 SD 카드에 `/log0000.bbl` 형태로 저장합니다.
- **LED 표시 순서**:
  1. 녹색 1초 (모드 전환 표시)
  2. 파란색 1초 (데이터 수신 양호 표시)
  3. 빨간색 깜빡임 (기록 중 표시)
- **종료**: 데이터 수신이 5초간 멈추면 파일을 마감하고 BLE 차단을 해제한 뒤 MODE_IDLE로 복귀합니다.

### 2.4 USB MSC 모드 (MODE_USB_MSC)

- **진입 조건**: 부트 버튼(GPIO 0) 클릭 감지 시 진입합니다.
- **동작**:
  - BLE 통신을 중지(`BLEDevice::deinit`)합니다.
  - 진행 중인 SD 기록을 마감하고 UART를 닫습니다.
  - USB Mass Storage Class로 진입하여 SD 카드를 PC에서 직접 접근할 수 있게 합니다.
- **LED**: 흰색이 고정되어 표시됩니다.
- **복귀**: 이 모드 사용 중 부트 버튼을 누르면 파일 읽기/쓰기를 종료 또는 완료한 후 재부팅하여 대기 상태로 돌아갑니다.

---

## 3. config.txt 설정

`SD 카드` 루트에 `config.txt` 파일을 생성하여 시스템 설정을 변경할 수 있습니다.

### 3.1 파일 형식

- **1번째 줄**: 최소 확보 용량 (MB 단위)
- **2번째 줄**: 최대 확보 용량 (MB 단위)
- **3번째 줄**: UART Baud Rate
- **4번째 줄 이후**: 주석으로 처리되어 읽거나 지우지 않습니다.

### 3.2 Baud Rate 설정값

아래 값 중 하나를 사용할 수 있습니다. 잘못된 값이 입력되면 기본값 `1500000`으로 수정됩니다.

| 설정값    | 설명                  |
| --------- | --------------------- |
| `921600`  | 표준 시리얼 통신 속도 |
| `1000000` | 고속 통신 속도        |
| `1500000` | 기본값                |
| `2000000` | 최고 속도             |

### 3.3 기본값

`config.txt`가 없거나 읽기 오류가 발생한 경우, 아래 기본값으로 파일을 새로 생성합니다.

```
100
500
1500000
1. minimum auto free space
2. maximum auto free space
3. baud rate = 921600 100000 150000 200000
# cli / serialpassthrough (port-1) (baud rate)
```

SD카드 용량이 100메가 이하일 경우 파일을 삭제해 500메가 용량을 확보합니다. 1500000 baud로 작동합니다. 삭제할 파일은 낮은 번호의 파일들이 순서대로 삭제되며 Ble접속이나 블랙박스 작동시 멈춥니다.

만약 log9999.bbl 이 존재할경우 높은 파일 순서대로 삭제됩니다. 주의가 필요합니다.

### 3.4 용량 관리 규칙

- MB 단위는 최대 9999까지 읽을 수 있으며, 그 이상은 9999로 처리됩니다.
- 읽을 수 없는 값이 포함된 경우, 1번째와 2번째 줄을 각각 `100`, `500`으로 기록하고 기본값으로 동작합니다.
- 주석은 수정하지 않고 그대로 유지합니다.
- 파일을 삭제하는 것이 아니라 내용을 수정합니다.

### 3.5 파일 삭제 로직

- 현재 SD 카드 여유 공간이 설정된 최소값 이하로 떨어지면, 낮은 번호의 `.bbl` 파일부터 용량을 합산합니다.
- 합산된 용량이 400MB를 넘으면 카운트를 중지하고 해당 파일들을 일괄 삭제합니다.
- 이를 통해 낮은 숫자의 파일이 우선 삭제되어 오래된 데이터가 먼저 정리됩니다.

---

## 4. 동작 흐름 요약

```
[대기 상태] ── BLE 연결 ──→ [BLE Config 모드]
    │                            │
    │ ← 해제 시 자동 복귀 ←       │
    │                            │
    └─ 16KB 데이터 수신 ──→ [블랙박스 기록 모드]
                                 │
                            5초 타임아웃
                                 │
                             기록 종료
                                 │
                             [대기 상태] ←──┘

[부트 버튼] ──→ [USB MSC 모드] ──→ [재부팅] ──→ [대기 상태]
```

---

## 5. LED 상태 표시

| 모드                      | LED 상태                  |
| ------------------------- | ------------------------- |
| MODE_IDLE                 | 노랑/녹색 0.5초 간격 점멸 |
| MODE_BLE_CONFIG           | 노랑색 고정               |
| MODE_BLACKBOX (전환)      | 녹색 1초                  |
| MODE_BLACKBOX (수신 양호) | 파란색 1초                |
| MODE_BLACKBOX (기록 중)   | 빨간색 깜빡임             |
| MODE_USB_MSC              | 흰색 고정                 |

---

## 6. 하드웨어 연결

- **UART**: 블랙박스 모드의 핀 연결을 그대로 사용합니다.
- **부트 버튼**: GPIO 0에 연결되어 있습니다.
- **PSRAM**: 링버퍼로 사용되어 느린 SD 카드에서도 안정적인 데이터 기록을 지원합니다.
- 핀 배정

| 기능        | GPIO    | 설명               |
| ----------- | ------- | ------------------ |
| FC UART RX  | GPIO 9  | FC에서 데이터 수신 |
| FC UART TX  | GPIO 8  | FC로 데이터 전송   |
| SD SPI SCK  | GPIO 13 | SD 클럭            |
| SD SPI MOSI | GPIO 11 | SD 데이터 출력     |
| SD SPI MISO | GPIO 12 | SD 데이터 입력     |
| SD SPI CS   | GPIO 10 | SD 칩 선택         |
| RGB LED     | GPIO 48 | 온보드 WS2812      |
| 부트버튼    | GPIO 0  | USB MSC 모드 진입  |

  SD 카드  짧은핀을 1번으로 핀번호 나열함.

  1-짧다 / 2 - CS / 3-MOSI/ 4-gnd / 5 - +전원/ 6 -SCK /7 -GND / 8-MOSI / 9

  Micro SD는 ( 4-gnd ) 가 없는 8핀구조

  1 / 2 - CS / 3-MOSI/ 4 - +전원/ 5 -SCK /6 -GND / 7-MOSI / 8

---

## 7. 테스트 결과

- **Baud Rate 1,500,000**에서 **3KB 데이터**를 안전하게 전송함이 확인되었습니다.
- **느린 2GB 용량의 SD 카드**에서도 ESP-S3의 버퍼를 활용하여 안정적으로 동작합니다.
- 속도가 빠른 SD카드를 사용한다면 더 빠른 데이터 기록이 예상됩니다. ( 테스트 필요 )

## 8. 참고 사항

- `config.txt`의 개행 문자(`\r`)는 자동으로 제거됩니다.
- 문자열 앞뒤의 공백 및 탭 문자는 자동으로 제거 후 처리됩니다.
- BLE 모드와 블랙박스 모드는 동시에 실행되지 않으며, 하나의 모드가 활성화되면 다른 모드는 정지.
- SD 카드 기록 중에는 BLE Advertising이 중지되어 새 연결을 받을 수 없습니다.
