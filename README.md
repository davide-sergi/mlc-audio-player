# MLC Audio Player

This repository contains a set of exercises that guide you in developing a smart drum sequencer powered by ST AIoT Craft and Machine Learning Core.

<details>

<summary>Pre-requisites</summary>

## Pre-requisites

### WSL (for Windows only)

For Windows OS, install WSL: https://learn.microsoft.com/en-us/windows/wsl/install .

### Python 3.8+

**macOS:**

- Install via Homebrew: `brew install python@3.8` (or higher version like 3.9, 3.10, 3.11, 3.12)
- Or download from [python.org](https://www.python.org/downloads/)
- Verify installation: `python3 --version`

**Windows --> WSL (Ubuntu/Debian):**

- Open WSL terminal
- Install via package manager: `sudo apt update && sudo apt install python3.8` (or higher version like 3.9, 3.10, 3.11, 3.12)
- Verify installation: `python3 --version`

**Linux (Ubuntu/Debian):**

- Install via package manager: `sudo apt update && sudo apt install python3.8` (or higher version)
- Verify installation: `python3 --version`

### pip (Python package manager)

**macOS & Linux:**

- Usually comes bundled with Python
- Verify installation: `pip3 --version`
- If not installed, run: `python3 -m ensurepip --upgrade`

**Windows --> WSL:**

- Usually comes bundled with Python
- Verify installation: open WSL terminal and execute `pip3 --version`
- If not installed, run: `python3 -m ensurepip --upgrade`

### Text editor or IDE

**All platforms (VS Code - recommended):**

- Download from [code.visualstudio.com](https://code.visualstudio.com/)
- Install Python extension from the Extensions marketplace
- Verify installation: Open the application

**For Windows only**: once installed VSCode on Windows, open VSCode and, from extension marketplace side panel, install **WSL** extension (by Microsoft - "ms-vscode-remote.remote-wsl")

</details>

---

<details>

<summary>USB Device Sharing in WSL</summary>

To access the ST AIoT Craft device in WSL for serial communication:

1. Install `usbipd-win` on Windows:

   ```
   winget install usbipd-win
   ```

2. In PowerShell (run as Administrator):
   - List USB devices: `usbipd list`
   - Bind your device (replace `X-Y` with the BUSID of your ST board): `usbipd bind --busid X-Y`
   - Attach to WSL: `usbipd attach --wsl --busid X-Y`

3. In WSL terminal:
   - Check for the device: `ls /dev/tty*` (look for `/dev/ttyACM0` or similar)
   - Run the Python script in WSL, selecting the `/dev/ttyACM0` device path

4. When done, detach in PowerShell: `usbipd detach --busid X-Y`

This allows WSL to access the USB serial device directly, bypassing Windows COM port issues.

</details>

---

<details>

<summary>Project Overview</summary>

## Project Overview

You will build a Python application that:

1. Lists and connects to a serial device
2. Programs the device with a Machine Learning Core configuration
3. Listens for incoming JSON messages from the device
4. Plays MP3 audio files based on received label IDs

</details>

---

<details>

<summary>Getting Started</summary>

## Getting Started

1. **Download the repository locally:**

```bash
# Download the ZIP file
wget https://github.com/davide-sergi/mlc-audio-player/archive/refs/heads/main.zip

# Extract the ZIP file
sudo apt install unzip
unzip main.zip

# Navigate into the extracted directory
cd mlc-audio-player-main
```

</details>

---

<details>

<summary>Exercise 0</summary>

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

4. Test by running: `main.py`

**Expected Outcome:** Application runs without errors and prints "Serial Audio Player".

</details>

---

<details>

<summary>Exercise 1</summary>

## Exercise 1: List and Connect to Serial Device

**Objective:** Implement device detection and user selection.

**Tasks:**

1. Add imports for serial communication:
   ```python
   import serial
   from serial.tools import list_ports
   ```
2. Implement `list_serial_devices()` function to enumerate connected devices

   ```python
    def list_serial_devices() -> list:
        ports = sorted(list_ports.comports(), key=lambda p: p.device)
        if not ports:
            print("No serial devices found.")
            return []

        print("Available serial devices:")
        for idx, port in enumerate(ports, start=1):
            description = port.description or "No description"
            print(f"  {idx}. {port.device} - {description}")
        return ports
   ```

3. Implement `choose_serial_device()` function to prompt user to select a device

   ```python
   def choose_serial_device() -> str:
    ports = list_serial_devices()

    if not ports:
        device = input("Enter serial device path manually (e.g. /dev/tty.usbmodemXXX): ").strip()
        if not device:
            raise RuntimeError("No serial device selected.")
        return device

    while True:
        choice = input(
            "Select device by number or type full device path: "
        ).strip()

        if not choice:
            print("Please provide a selection.")
            continue

        if choice.isdigit():
            index = int(choice) - 1
            if 0 <= index < len(ports):
                return ports[index].device
            print("Invalid number, try again.")
            continue

        return choice
   ```

4. In `main()`, call `choose_serial_device()` and print the selected device

**Testing:**

- Connect your device and run the application
- Verify that your device appears in the list
- Verify you can select it

**Expected Outcome:** Application lists serial devices and connects to the selected one.

</details>

---

<details>

<summary>Exercise 2</summary>

## Exercise 2: Train and Load Device Program

**Objective:** Prepare the device programming configuration.

**Tasks:**

1. Train an ML model on ST AIoT Craft (https://staiotcraft.st.com) with sensor configuration
2. Download the resulting JSON configuration
3. Save it as `device_program.json` in the project root
4. Check matching MP3 file names to label IDs

**File Format Expected:**
`device_program.json` should have a structure like:

```json
{
    ...
    "outputs": [
        {
            "name": "Categorical output",
            "core": "MLC",
            "type": "uint8_t",
            "len": "1",
            "reg_addr": "0x70",
            "reg_name": "MLC1_SRC",
            "results": [
                {
                    "code": "0x00",
                    "label": "0"
                },
                {
                    "code": "0x04",
                    "label": "4"
                },
                {
                    "code": "0x08",
                    "label": "8"
                },
                {
                    "code": "0x0C",
                    "label": "none"
                }
            ]
        }
    ],
}
```

</details>

---

<details>

<summary>Exercise 3</summary>

## Exercise 3: Program the Device and Parse Configuration

**Objective:** Send device programming payload and load label mappings.

**Tasks:**

1. Add required imports at the top of your file:

   ```python
   from typing import Dict, Optional
   ```

2. Copy and paste these functions into your `main.py`:

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

**Helper function `resolve_sound_file()`:**

```python
def resolve_sound_file(label_id: int, sounds_dir: Path) -> Path:
    return sounds_dir / f"{label_id}.mp3"
```

**Testing:**

- Run the application and verify the device is programmed
- Check that the mapping is loaded without errors

**Expected Outcome:** Device receives the programming payload; label mappings are loaded.

</details>

---

<details>

<summary>Exercise 4</summary>

## Exercise 4: Receive Messages and Play Audio

**Objective:** Listen for device messages and trigger audio playback.

**Tasks:**
Add these imports at the top of your file:

```python
import shutil
import subprocess
from typing import Dict, Optional
```

Copy and paste these functions into your `main.py`:

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

        sound_file = resolve_sound_file(label_id, sounds_dir)
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
            listen_and_play(serial_conn, Path("sounds"))
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

**Testing:** Connect your device and trigger messages to hear audio response

**Expected Outcome:** Application plays audio when receiving serial messages with label IDs.

</details>
