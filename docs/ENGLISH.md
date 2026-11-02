# M5Chess: build, installation and user guide

M5Chess is a standalone chess application for the **M5Stack Core2**, written by
**Dirk D. Sommerfeld**. The Virgin engine runs on the device in Adaptive mode.
No phone, computer, Internet connection or external neural-network files are
needed during play. A supported electronic board supplies the moves.

[Deutsche Anleitung](../README.md)

## Hardware and compatibility

Use a Core2 with the original ESP32, 16 MB flash and working PSRAM. This is not
an ESP32-S3 or CoreS3 firmware. A USB **data** cable is required for uploading.
The screen does not provide touch-based piece movement.

The following electronic boards have been tested with M5Chess:

| Board family | Support |
|---|---|
| Chessnut | Tested. Discovery accepts the common name, regardless of model suffix. |
| Millennium / ChessLink | Tested. BLE connection and board protocol. |
| Certabo | Tested. BLE/NUS transport, 34-piece calibration and monochrome LEDs. |
| Tabutronic Spectrum | Tested. BLE/NUS transport, queen-layout selection and RGB LEDs with evaluation colours. |

Certabo and Spectrum require a compatible BLE-to-UART bridge exposing the
Nordic UART Service, as used by eChessGateway. Connecting their USB cable
directly to the Core2 is not supported by this application. Occupancy-only
Sentio data is not a supported replacement for a piece-ID board.

## Reproducible source checkout

Install Git and Python 3.11, then run:

```sh
git clone --recurse-submodules https://github.com/parlue/M5Chess.git
cd M5Chess
python -m pip install -r requirements.txt
```

Alternatively create and activate a Python virtual environment first. The
requirements pin the tested [PlatformIO Core 6.2.0](https://pypi.org/project/platformio/6.2.0/).
The ESP32 platform is pinned to `espressif32@6.8.1`, using ESP-IDF 5.3.

The three Git submodules are pinned to the revisions in
[`dependencies.json`](../dependencies.json). After cloning without submodules,
or after changing revisions, run:

```sh
git submodule update --init --recursive
```

GitHub's source ZIP does not include submodule contents. Use Git for the normal
build procedure. The first build needs Internet access to download the toolchain,
ESP-IDF and managed component packages. These downloads are not needed for play.

## Build and flash

From the repository root:

```sh
python -m platformio run -d esp32_core2
python -m platformio device list
python -m platformio run -d esp32_core2 -t upload --upload-port COM17
```

Replace `COM17` with the port reported for **your** Core2. Linux/macOS use their
reported device path instead. Close serial monitors before uploading. Uploading
restarts the device and replaces its application firmware.

The build output is `esp32_core2/.pio/build/m5stack_core2/`. The flash layout is:

| File | Address |
|---|---|
| `bootloader.bin` | `0x1000` |
| `partitions.bin` | `0x8000` |
| `firmware.bin` | `0x10000` |

Use DIO, 40 MHz flash and 16 MB flash size. The PlatformIO upload command sets
these values and addresses for you. Keep the complete project layout: the
firmware component uses the Virgin sources from the sibling `src` directory.
In VS Code, open `esp32_core2` as the PlatformIO project.

## Start a game

1. Power on the Core2. It starts without scanning or connecting to a board.
2. Power on your board and disconnect any competing board apps.
3. Tap **Verbinden** (Connect), then select your board from the scan results.
4. Arrange the full starting position. You play White by default.
5. To let Virgin play White, tap **Move**. You take Black and the display rotates.
6. Make your moves on the physical board. Virgin immediately previews its reply
   on the Core2 screen and marks the relevant physical squares with LEDs.
7. Carry out that complete reply on the real board. The game continues only
   after the full resulting position has been recognised.

**Move** has two functions: on your turn it hands the side to move to Virgin
and gives you the opposite side; while Virgin is searching it stops the search
early. It does not produce a second move while an earlier reply still needs
physical execution. Partial or mismatched board positions cannot force a move.

Rebuilding the complete starting position starts a new game and restores you
to White. Repeated identical start-position reports do not cancel Virgin's
opening move. There is no New Game button and no persistent game archive.
There is no automatic reconnection after a disconnect; choose Connect again.

The connection button is labelled **Brett** (Board) while connected. Selecting
it opens discovery and disconnects the current board.

## Certabo and Spectrum calibration

Each manual connection starts fresh piece-ID calibration. Both models may
advertise as `Certabo`. Model selection comes from the spare queens:

| Model | Spare white queen | Spare black queen |
|---|---|---|
| Monochrome Certabo | d3 | d6 |
| Spectrum RGB | e3 | e6 |

Keep all 32 ordinary pieces in the normal starting position, White on ranks
1–2, and place both spare queens on the indicated squares. The driver first
selects the LED type from a stable extra-piece location, then learns all 34
unique IDs using a seven-scan majority window. Keep the setup still until the
central squares flash three times. Remove both spare queens. Only the complete
normal starting position releases the game for play.

No LED format is probed before the queen gesture selects the model. L/D
markers, advertising suffixes, MAC addresses, packet sizes and ID prefixes do
not override the choice. An unknown or duplicate piece ID withholds that scan;
it is never silently treated as an empty square. Lost BLE fragments discard
partial frames and incomplete learning windows.

The board protocol supplies five-byte piece IDs in a8–h1 order. The controller
converts them to the engine's a1–h8 coordinates. The monochrome board receives
eight-byte LED masks. Spectrum receives 247-byte RGB frames for 81 LED corners;
each highlighted square lights its four surrounding corners. Frames use Write
Without Response and are split to fit the negotiated MTU. The connection
requests MTU 256 for Certabo and does not send a board-data handshake or poll.

Spectrum setup LEDs start in blue at 80% intensity. During play, move and
correction highlights use the player's evaluation, following chesstactics:

| Player evaluation, centipawns | Logical RGB colour |
|---|---|
| −400 or below | Dark red: 140, 0, 0 |
| −200 | Orange: 240, 90, 10 |
| 0 | Light blue: 70, 180, 255 |
| +200 | Turquoise: 40, 190, 150 |
| +400 or above | Green: 46, 204, 113 |

Colours interpolate continuously between these stops. Spectrum's hardware
colour correction reduces red and green in blue-dominant tones so neutral
highlights look blue rather than almost white. It leaves non-blue-dominant
tones unchanged. This is the same `spectrumTint` behaviour as chesstactics.
The LED frame guards the reserved `FF 55` start-marker sequence in its payload.

## Evaluation and queen moods

The numeric search score is always shown **from White's perspective**. The
logo queen and Spectrum colours are shown **from your perspective**. These
perspectives differ when you play Black.

The queen updates when Virgin's selected move is accepted. It prefers the
Adaptive diagnostic's `after` value, which evaluates the actually selected
move for the player. Without that diagnostic it uses the negated engine search
score. Mate distances are mapped to ±(100000 − mate-in-n). A draw or a new game
is neutral; checkmate sets the winning or losing extreme. Switching sides
reverses the player's evaluation.

| State | Player evaluation | Queen appearance |
|---|---|---|
| 0 | ≥ +250 | Gold, radiance and fire |
| 1 | +120 to +249 | Silver shimmer |
| 2 | −119 to +119 | Plain queen |
| 3 | −249 to −120 | Storm and wind |
| 4 | −399 to −250 | Rain |
| 5 | ≤ −400 | Lightning and fire |

These are the exact BeatVirgin threshold boundaries. The queen has no colour
track or movement animation. During a search it retains the previous completed
state. Live search information shows depth, White's score and the first part of
the principal variation in coordinate notation. Adaptive may select a different
move from that strongest principal variation.

## Battery and engine resources

The horizontal battery in the upper-right corner shows the M5 power library's
estimated percentage. A lightning bolt means active charging. At 20% or below,
the fill becomes red; an unavailable reading appears as `--%`. Readings are
cached and updated every five seconds, including while the board is idle.

The engine uses a 1 MiB transposition table in PSRAM, a 72 KiB internal search
stack, a maximum search path of 20 plies and a 2.5-second move budget. These are
fixed firmware choices, not user settings. MCU playing strength has not been
measured. The RGB565 framebuffer is in PSRAM; analysis updates transfer the
sidebar rather than clearing the visible chessboard.

## Tests and source layout

Install CMake 3.20+ and a C++20 compiler (for example Visual Studio Build Tools
on Windows, or GCC/Clang on Linux). No ESP32 or board is needed for host tests:

```sh
cmake -S tests -B build/tests
cmake --build build/tests --config Release
ctest --test-dir build/tests -C Release --output-on-failure
```

There are seven test executables: the Core2 game/mood suite, the Core2 Certabo
adapter suite, and five original gateway regression suites. They cover move
confirmation, castling, en passant, promotion, side changes, restart isolation,
queen thresholds and mate scores, frame parsing, calibration, queen selection,
LED coordinates, colour interpolation and Spectrum blue correction. Assertions
are enabled for the gateway tests even in Release builds.

| Path | Purpose |
|---|---|
| `esp32_core2/` | PlatformIO/ESP-IDF project and pinned component submodules |
| `esp32_core2/main/` | GUI, board transport, game model and image tables |
| `src/` | Required Virgin engine implementation and original engine licence |
| `assets/` | Merida and queen WebP sources |
| `tools/generate_assets.py` | Optional regeneration/check of RGB565 image tables |
| `tests/` | Portable CMake/CTest setup and gateway regression fixtures |
| `licenses/`, `THIRD_PARTY.md` | Third-party notices and licence texts |

The firmware already contains generated image tables. Asset regeneration is
optional and needs Pillow only:

```sh
python -m pip install -r requirements-assets.txt
python tools/generate_assets.py --check
python tools/generate_assets.py
```

The development firmware also has local UART diagnostics at 115200 baud:
`status` returns state and battery readings; `screenshot` emits a 320×240
RGB565 image between its framing messages. The `tap X Y` and `testboard` test
helpers modify the running test state and are not part of normal play. A reboot
clears the virtual test board. Do not send test commands during a real game.

## Troubleshooting

- **Missing component / M5Unified / esp-nimble-cpp:** initialise the submodules.
- **Port missing:** use a USB data cable, reconnect, then run `platformio device list` again.
- **Port busy:** close other serial monitors or upload tools.
- **Board not found:** power it on, disconnect phone/PC board apps, then use Connect again.
- **Certabo calibration does not finish:** check all 32 starting pieces, both spare queens on the correct pair of squares, and leave the complete setup still. Connect again to restart model selection if needed.
- **Engine waits at “Nachziehen”:** finish the indicated physical reply, including the rook in castling or removal of a captured piece.
- **Engine or display memory error:** verify that this is the Core2 hardware and PSRAM configuration; use the checked-in SDK configuration.

For licensing and attribution, see [THIRD_PARTY.md](../THIRD_PARTY.md).
