# 🎮 Tion GameOS

A retro handheld game console firmware for **ESP32** with a 128×64 OLED display. Features 14 games, a web-based control panel, IR blaster, BLE scanner, servo control, and more — all packed into a single `.cpp` file.

---

## 📸 Features

### 🕹️ Games (14 total)
| Game | Description |
|------|-------------|
| Snake | Classic snake, grow by eating food |
| Pong | 1-player (Easy/Hard) or 2-player |
| Space Invaders | Shoot descending alien waves |
| Arkanoid | Brick-breaker with power-ups, 5 levels |
| Tetris | Full Tetris with level select & hi-score |
| Battle City | Tank game with enemies, multi-level |
| Flappy Bird | Dodge pipes, survive as long as possible |
| Pac-Man | Maze chase with ghosts |
| Doom | First-person shooter raycaster |
| Asteroids | Shoot and dodge asteroids |
| 2048 | Slide tiles to reach 2048 |
| Minesweeper | Classic mine grid puzzle |
| Tron | Two-player light-cycle battle |
| Simon Says | Memory sequence game |

### 🛠️ Tools
- **TV Killer** — sends IR power-off codes for hundreds of TV brands
- **IR Blink** — custom IR signal transmitter
- **IR Files** — send IR codes from files stored on LittleFS
- **GPIO 16** — manual HIGH/LOW control of GPIO pin
- **Servo** — control a servo motor (0–180°) via buttons or web
- **LED** — WS2812 LED controller with color/effect selection

### 📱 Other Utilities
- Calculator
- Countdown Timer
- Stopwatch
- Battery Info (voltage, percentage, estimated time remaining)
- Flashlight (GPIO LED toggle)
- File Browser (LittleFS, .txt reader)
- Connect (WiFi AP + web dashboard)
- Information screen (firmware version, hardware info)

### 🌐 Web Dashboard
When WiFi is enabled, the device creates an Access Point:
- **SSID:** `Tion GameOS`
- **Password:** `12345678`
- **URL:** `http://192.168.4.1`

Web panel features:
- LED color & effect control (Rainbow, Pulse, Fire, Ocean, Police, Xmas)
- Servo angle control
- GPIO toggle
- BLE device scanner
- File manager (upload, delete, view `.txt` files)
- IR file sender
- Settings (pin configuration, brightness, language)

---

## 🔧 Hardware

### Required Components
| Component | Details |
|-----------|---------|
| Microcontroller | ESP32 (DevKit or similar) |
| Display | SH1106 128×64 OLED (I2C) |
| Buttons | 6× tactile buttons (UP, DOWN, LEFT, RIGHT, A, B) |
| Battery ADC | Resistor divider on GPIO 35 (Li-Ion monitoring) |

### Optional Components
| Component | Details |
|-----------|---------|
| WS2812 LED | Addressable RGB LED (NeoPixel) |
| IR LED | IR transmitter on GPIO 33 |
| Servo Motor | Connected to GPIO 16 (configurable) |
| Vibration Motor | PWM vibro motor (stub, configurable) |
| Built-in LED | GPIO 2 (synced with menu LED state) |

### Default Pin Mapping
```
GPIO 2   → Built-in LED
GPIO 4   → WS2812 data (NeoPixel)
GPIO 13  → Button UP
GPIO 14  → Button DOWN
GPIO 16  → GPIO tool / Servo signal
GPIO 25  → Button LEFT
GPIO 26  → Button RIGHT
GPIO 27  → Button A (confirm / action)
GPIO 32  → Button B (back / menu)
GPIO 33  → IR LED transmitter
GPIO 35  → Battery ADC (voltage divider input)
SDA/SCL  → OLED display (I2C default pins)
```

> All pins can be reconfigured via the web dashboard without recompiling.

---

## 🚀 Flashing with PlatformIO

### Prerequisites
1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install the [PlatformIO IDE extension](https://platformio.org/install/ide?install=vscode)
3. Install a USB driver for your ESP32 board if needed (CP2102 or CH340)

### Project Setup

1. **Clone or download** this repository:
   ```bash
   git clone https://github.com/yourname/tion-gameos.git
   cd tion-gameos
   ```

2. **Project structure** should look like this:
   ```
   tion-gameos/
   ├── src/
   │   ├── vmain.cpp
   │   └── WORLD_IR_CODES.h
   ├── data/              ← LittleFS files (optional)
   ├── platformio.ini
   └── README.md
   ```

3. **`platformio.ini`** configuration:
   ```ini
   [env:esp32dev]
   platform = espressif32
   board = esp32dev
   framework = arduino
   monitor_speed = 115200
   board_build.filesystem = littlefs
   lib_deps =
       olikraus/U8g2
       adafruit/Adafruit NeoPixel
       h2zero/NimBLE-Arduino
       fastled/FastLED
   ```

### Build & Flash

**Via VS Code PlatformIO UI:**
1. Open the project folder in VS Code
2. Click the **✓ Build** button (bottom toolbar) to compile
3. Connect your ESP32 via USB
4. Click the **→ Upload** button to flash

**Via terminal:**
```bash
# Build only
pio run

# Build and upload
pio run --target upload

# Upload and open serial monitor
pio run --target upload && pio device monitor
```

### Flash LittleFS Filesystem (optional)
If you want to use IR files from LittleFS or store `.txt` files:
```bash
# Upload the contents of the /data folder to the device
pio run --target uploadfs
```

> Make sure your `platformio.ini` has `board_build.filesystem = littlefs`

---

## ⚙️ Build Configuration

At the top of `vmain.cpp` you can enable/disable modules before compiling:

```cpp
#define CFG_USE_NEOPIXEL   1   // WS2812 addressable LED
#define CFG_USE_VIBRO      0   // Vibration motor
#define CFG_USE_TOOLS      1   // Tools section (IR, TV-Killer, Servo…)
#define CFG_USE_BLE        1   // Bluetooth scanner in web UI
```

Setting a value to `0` removes that module from the build, saving flash and RAM.

---

## 🎮 Controls

| Button | Function |
|--------|----------|
| **UP / DOWN** | Navigate menu, adjust values |
| **LEFT / RIGHT** | Navigate menu pages, adjust values |
| **A** | Confirm / Start / Action |
| **B** | Back / Exit / Cancel |
| **A + B (hold)** | Emergency exit to main menu from anywhere |

---

## 🌍 Languages

The UI supports 3 languages, switchable from **Settings → Language**:
- 🇺🇦 Ukrainian
- 🇷🇺 Russian
- 🇬🇧 English

The selected language is saved to EEPROM and persists across reboots.

---

## 💾 Persistent Storage

Settings are saved to **EEPROM** (512 bytes):

| Address | Data |
|---------|------|
| 0 | Display brightness (%) |
| 1 | Vibration on/off |
| 2 | LED enabled |
| 3–4 | Battery calibration coefficient |
| 5 | LED color index |
| 6 | Built-in LED sync |
| 7 | LED brightness (%) |
| 8 | WS2812 pin |
| 9 | Built-in LED pin |
| 10 | Extra GPIO / Servo pin |
| 11 | UI language (0=UA, 1=RU, 2=EN) |

Game high scores are stored separately in EEPROM per game.

---

## 🔋 Battery Monitoring

The firmware reads battery voltage from GPIO 35 via a 1:2 resistor divider. It uses:
- 32-sample averaging for noise reduction
- Exponential moving average filter (75/25 weight)
- Configurable calibration coefficient (default `1.05`)
- Non-linear Li-Ion discharge curve mapping

To calibrate: go to **Settings → BAT.CALIB**, fully charge the battery, then press **A**.

---

## 📡 IR Transmitter

IR signals are generated using the **ESP32 LEDC** peripheral at **38 kHz** on GPIO 33.

Supported modes:
- **TV Killer** — cycles through power-off codes for ~100+ TV brands (from `WORLD_IR_CODES.h`)
- **IR Blink** — sends a single configurable pulse
- **IR Files** — reads `.ir` files from LittleFS and transmits them

---

## 🦷 Bluetooth (BLE)

When BLE is enabled (`CFG_USE_BLE = 1`), the web dashboard includes a **BLE Scanner** tab that lists nearby Bluetooth devices with their RSSI, MAC address, and name.

Uses the lightweight [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino) library.

---

## 📁 IR File Format

IR files stored on LittleFS (`.ir` extension) should follow this format:
```
38000
pulse 342 171 21 21 21 64 ...
```
- Line 1: carrier frequency in Hz
- Line 2+: `pulse` followed by alternating mark/space timings in microseconds

---

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| Upload fails | Hold the **BOOT** button on ESP32 while uploading starts |
| Display blank | Check I2C wiring (SDA/SCL), verify SH1106 I2C address |
| Buttons not responding | Check `INPUT_PULLUP` wiring — buttons should connect pin to GND |
| WiFi AP not appearing | Make sure `wifiEnabled = true` or enable via Connect menu |
| BLE scan crashes | Ensure NimBLE-Arduino ≥ 1.4.3 is installed |
| LittleFS upload fails | Disconnect serial monitor before running `uploadfs` |
| OTA / large uploads fail | Use `esptool.py` directly or reduce partition size in `platformio.ini` |

---

## 📦 Dependencies

| Library | Version | Purpose |
|---------|---------|---------|
| [U8g2](https://github.com/olikraus/u8g2) | ≥ 2.35 | OLED display driver |
| [Adafruit NeoPixel](https://github.com/adafruit/Adafruit_NeoPixel) | ≥ 1.11 | WS2812 LED control |
| [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino) | ≥ 1.4.3 | BLE scanning |
| [FastLED](https://github.com/FastLED/FastLED) | ≥ 3.6 | LED effects support |
| EEPROM | built-in | Settings storage |
| LittleFS | built-in | File system |
| WebServer | built-in | HTTP web panel |
| WiFi | built-in | WiFi AP mode |

---

## 📄 License

This project is open source. Feel free to use, modify, and distribute.

---

## 🙏 Credits

Built with ❤️ using the Arduino framework on ESP32.  
UI rendered via [U8g2](https://github.com/olikraus/u8g2).  
BLE powered by [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino).
