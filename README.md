# MLC Audio Player

This repository contains a set of exercises that guide you in developing a smart drum sequencer powered by ST AIoT Craft and Machine Learning Core.

## Pre-requisites

### Python 3.8+

**macOS:**

- Install via Homebrew: `brew install python@3.8` (or higher version like 3.9, 3.10, 3.11, 3.12)
- Or download from [python.org](https://www.python.org/downloads/)
- Verify installation: `python3 --version`

**Windows:**

- Download the installer from [python.org](https://www.python.org/downloads/)
- Run the installer and **check "Add Python to PATH"** during installation
- Verify installation: Open Command Prompt and run `python --version`

**Linux (Ubuntu/Debian):**

- Install via package manager: `sudo apt update && sudo apt install python3.8` (or higher version)
- Verify installation: `python3 --version`

### pip (Python package manager)

**macOS & Linux:**

- Usually comes bundled with Python
- Verify installation: `pip3 --version`
- If not installed, run: `python3 -m ensurepip --upgrade`

**Windows:**

- Usually comes bundled with Python (if you checked "Add Python to PATH")
- Verify installation: Open Command Prompt and run `pip --version`
- If not installed, run: `python -m ensurepip --upgrade`

### Text editor or IDE

**All platforms (VS Code - recommended):**

- Download from [code.visualstudio.com](https://code.visualstudio.com/)
- Install Python extension from the Extensions marketplace
- Verify installation: Open the application

**Alternatives:** PyCharm, Sublime Text, or any text editor

### Operating System Specific Requirements

**macOS:**

- This application uses the native `afplay` command for audio playback
- Verify `afplay` is available: `which afplay` (should output `/usr/bin/afplay`)

**Windows:**

- This application uses native Windows media player (via `winsound` or similar)
- No additional installation needed; built-in to Windows

**Linux:**

- This application uses `aplay` (ALSA player) for audio playback
- Install if not present: `sudo apt install alsa-utils` (Ubuntu/Debian)
- Verify installation: `which aplay`

### Serial device connection

**Hardware required:**

- A green board or ST AIoT Craft-compatible device
- USB cable connecting your device to your computer

**Verification:**

**macOS:**

- Run: `ls /dev/tty.*`
- You should see a device like `/dev/tty.usbmodem...`

**Windows:**

- Open "Device Manager" (search for it in the Start menu)
- Look under "Ports (COM & LPT)" for your device (e.g., COM3, COM4)
- Or run in PowerShell: `Get-PnpDevice -Class Ports`

**Linux:**

- Run: `ls /dev/ttyUSB*` or `ls /dev/ttyACM*`
- You should see a device like `/dev/ttyUSB0` or `/dev/ttyACM0`

## Project Overview

You will build a Python application that:

1. Lists and connects to a serial device
2. Programs the device with a Machine Learning Core configuration
3. Listens for incoming JSON messages from the device
4. Plays MP3 audio files based on received label IDs

## Getting Started

1. **Download the repository locally:**

   ```bash
   git clone <repository-url>
   cd mlc-audio-player
   ```

   Or download the ZIP file and extract it.

2. The `sol/` folder contains the complete working solution. Refer to it when you get stuck!

## Exercise 0: Setup and Project Skeleton

**Objective:** Set up your Python environment and create the basic project structure.

**Tasks:**

1. Create a new Python virtual environment
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. Create `requirements.txt` with these dependencies:
   ```
   pyserial>=3.5
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
4. Create `serial_audio_player.py` with this skeleton:

   ```python
   #!/usr/bin/env python3
   """Serial-driven MP3 audio player."""

   import argparse
   import json
   import sys
   from pathlib import Path

   def main() -> int:
       print("Serial Audio Player")
       return 0

   if __name__ == "__main__":
       sys.exit(main())
   ```

5. Test by running: `main.py`

**Expected Outcome:** Application runs without errors and prints "Serial Audio Player".

---

## Exercise 1: List and Connect to Serial Device

**Objective:** Implement device detection and user selection.

**Tasks:**

1. Add imports for serial communication:
   ```python
   import serial
   from serial.tools import list_ports
   ```
2. Implement `list_serial_devices()` function to enumerate connected devices
3. Implement `choose_serial_device()` function to prompt user to select a device
4. In `main()`, call `choose_serial_device()` and print the selected device

**Hints:**

- Use `list_ports.comports()` to get available serial ports
- Sort ports by device name for consistent ordering
- Display port descriptions to help users identify their device
- Allow manual device path entry as fallback

**Testing:**

- Connect your device and run the application
- Verify that your device appears in the list
- Verify you can select it

**Expected Outcome:** Application lists serial devices and connects to the selected one.

---

## Exercise 2: Train and Load Device Program

**Objective:** Prepare the device programming configuration.

**Tasks:**

1. Train an ML model on ST AIoT Craft (https://staiotcraft.st.com) with sensor configuration
2. Download the resulting JSON configuration
3. Save it as `device_program.json` in the project root
4. Create `label_sounds.json` mapping from label_id to MP3 filename:
   ```json
   {
     "0x0": "sound_0.mp3",
     "0x4": "sound_4.mp3",
     "0x8": "sound_8.mp3",
     "0xC": "sound_c.mp3"
   }
   ```
5. Create a `sounds/` folder and add MP3 files matching your mapping

**File Format Expected:**
`device_program.json` should have a structure like:

```json
{
  "sensors": [
    {
      "name": "LSM6DSV16X",
      "configuration": [
        {"type": "write", "address": "0x10", "data": "0x00"},
        {"type": "write", "address": "0x11", "data": "0x00"}
      ]
    }
  ]
}
```

**Expected Outcome:** `device_program.json`, `label_sounds.json`, and MP3 files are in place.

---

## Exercise 3: Program the Device and Parse Configuration

**Objective:** Send device programming payload and load label mappings.

**Tasks:**

1. Add required imports at the top of your file:

   ```python
   from typing import Dict, Optional
   ```

2. Copy and paste these functions into your `serial_audio_player.py`:

**`format_hex_byte()` function:**

```python
def format_hex_byte(value: object, field_name: str) -> str:
    try:
        parsed = int(str(value).strip(), 16)
    except ValueError as exc:
        raise ValueError(f"Invalid hex value for {field_name}: {value}") from exc

    if not 0 <= parsed <= 0xFF:
        raise ValueError(f"Hex value for {field_name} is out of byte range: {value}")

    return f"{parsed:02X}"
```

**`build_program_payload()` function:**

```python
def build_program_payload(program_file: Path) -> bytes:
    program = json.loads(program_file.read_text(encoding="utf-8"))

    try:
        configuration = program["sensors"][0]["configuration"]
    except (KeyError, IndexError, TypeError) as exc:
        raise ValueError(
            "Program file must contain sensors[0].configuration."
        ) from exc

    copy_ucf_parts = []
    for command in configuration:
        if command.get("type") != "write":
            continue

        copy_ucf_parts.append(format_hex_byte(command.get("address"), "address"))
        copy_ucf_parts.append(format_hex_byte(command.get("data"), "data"))

    copy_ucf = "".join(copy_ucf_parts)
    command_payload = {
        "lsm6dsv16x_mlc*load_model": {
            "arguments": {
                "filename": program_file.name,
                "size": len(copy_ucf),
                "model": copy_ucf,
            }
        }
    }

    return json.dumps(command_payload, separators=(",", ":")).encode("utf-8")
```

**`send_program_payload()` function:**

```python
def send_program_payload(
    serial_conn: serial.Serial,
    program_file: Path,
    append_newline: bool,
) -> None:
    if not program_file.exists():
        raise FileNotFoundError(f"Program file not found: {program_file}")

    payload = build_program_payload(program_file)
    serial_conn.write(payload)
    if append_newline and not payload.endswith(b"\n"):
        serial_conn.write(b"\n")
    serial_conn.flush()
    print(f"Programmed device with '{program_file}' ({len(payload)} bytes sent).")
```

**`load_mapping()` function:**

```python
def load_mapping(mapping_file: Path) -> Dict[int, str]:
    if not mapping_file.exists():
        print(
            f"Mapping file '{mapping_file}' not found. Using default naming: <label_id>.mp3"
        )
        return {}

    with mapping_file.open("r", encoding="utf-8") as handle:
        raw_mapping = json.load(handle)

    mapping: Dict[int, str] = {}
    for key, value in raw_mapping.items():
        try:
            mapping[int(str(key).strip(), 0)] = str(value)
        except (TypeError, ValueError):
            print(f"Skipping invalid mapping key '{key}'.")

    return mapping
```

**Helper function `resolve_sound_file()`:**

```python
def resolve_sound_file(label_id: int, mapping: Dict[int, str], sounds_dir: Path) -> Path:
    if label_id in mapping:
        return sounds_dir / mapping[label_id]
    return sounds_dir / f"{label_id}.mp3"
```

3. Update your `main()` function to call these after connecting to the serial device (replace your current main):

```python
def main() -> int:
    try:
        device = choose_serial_device()
        print(f"Connecting to {device}...")
        with serial.Serial(device, 115200, timeout=1.0) as serial_conn:
            print("Serial connection established.")
            send_program_payload(serial_conn, Path("device_program.json"), append_newline=True)
            mapping = load_mapping(Path("label_sounds.json"))
            print("Label mappings loaded.")
    except KeyboardInterrupt:
        print("\nStopped by user.")
    except Exception as exc:
        print(f"Error: {exc}")
        return 1

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

**Testing:**

- Run the application and verify the device is programmed
- Check that the mapping is loaded without errors

**Expected Outcome:** Device receives the programming payload; label mappings are loaded.

---

## Exercise 4: Receive Messages and Play Audio

**Objective:** Listen for device messages and trigger audio playback.

**Tasks:**
Add these imports at the top of your file:

```python
import shutil
import subprocess
from typing import Dict, Optional
```

Copy and paste these functions into your `serial_audio_player.py`:

**`parse_label_id_value()` function:**

```python
def parse_label_id_value(value: object) -> Optional[int]:
    if isinstance(value, int):
        return value

    if isinstance(value, str):
        try:
            # Supports values like "8", "0x8", "0xC", etc.
            return int(value.strip(), 0)
        except ValueError:
            return None

    return None
```

**`extract_label_id()` function:**

```python
def extract_label_id(raw_message: str) -> Optional[int]:
    try:
        data = json.loads(raw_message)
    except json.JSONDecodeError:
        return None

    if not isinstance(data, dict):
        return None

    # Support both payload shapes:
    # 1) {"label_id": 3}
    # 2) {"lsm6dsv16x_mlc": {"label_id": 3}}
    label_id = data.get("label_id")
    if label_id is None:
        nested = data.get("lsm6dsv16x_mlc")
        if isinstance(nested, dict):
            label_id = nested.get("label_id")

    return parse_label_id_value(label_id)
```

**Global variable for audio process management:**

```python
_current_audio_process: Optional[subprocess.Popen[bytes]] = None
```

**`play_mp3()` function:**

```python
def play_mp3(path: Path) -> None:
    global _current_audio_process

    if not path.exists():
        print(f"MP3 file not found: {path}")
        return

    afplay_path = shutil.which("afplay")
    if afplay_path is None:
        raise RuntimeError("'afplay' was not found. This macOS build requires afplay for MP3 playback.")

    if _current_audio_process is not None and _current_audio_process.poll() is None:
        _current_audio_process.terminate()
        _current_audio_process.wait(timeout=1)

    _current_audio_process = subprocess.Popen(
        [afplay_path, str(path)],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
    )
    print(f"Playing: {path.name}")
```

**`stop_audio_playback()` function:**

```python
def stop_audio_playback() -> None:
    global _current_audio_process

    if _current_audio_process is None:
        return

    if _current_audio_process.poll() is None:
        _current_audio_process.terminate()
        _current_audio_process.wait(timeout=1)

    _current_audio_process = None
```

**`listen_and_play()` function:**

```python
def listen_and_play(
    serial_conn: serial.Serial,
    mapping: Dict[int, str],
    sounds_dir: Path,
) -> None:
    print("Listening for messages. Press Ctrl+C to stop.")

    while True:
        raw = serial_conn.readline()
        if not raw:
            continue

        message = raw.decode("utf-8", errors="ignore").strip()
        if not message:
            continue

        print(f"Received: {message}")
        label_id = extract_label_id(message)

        if label_id is None:
            print("Ignored message: invalid JSON or missing integer label_id.")
            continue

        sound_file = resolve_sound_file(label_id, mapping, sounds_dir)
        play_mp3(sound_file)
```

3. Update your `main()` function to include the listening loop:

```python
def main() -> int:
    try:
        device = choose_serial_device()
        print(f"Connecting to {device} at 115200 baud...")
        with serial.Serial(device, 115200, timeout=1.0) as serial_conn:
            print("Serial connection established.")
            send_program_payload(serial_conn, Path("device_program.json"), append_newline=True)
            mapping = load_mapping(Path("label_sounds.json"))
            listen_and_play(serial_conn, mapping, Path("sounds"))
    except KeyboardInterrupt:
        print("\nStopped by user.")
    except Exception as exc:
        print(f"Error: {exc}")
        return 1
    finally:
        stop_audio_playback()

    return 0


if __name__ == "__main__":
    sys.exit(main())
```

**Testing:**

- Run in test mode first: `main.py --test` (after implementing Exercise 5)
- Connect your device and trigger messages to hear audio response

**Expected Outcome:** Application plays audio when receiving serial messages with label IDs.

---

## Exercise 5: Add Test Mode and Final Testing

Run `main.py` and verify correct audio plays for each
