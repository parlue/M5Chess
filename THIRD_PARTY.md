# Licences and attribution

M5Chess is by **Dirk D. Sommerfeld**. The combined firmware package is
distributed under GPL-3.0; the text is in [LICENSE](LICENSE). Individual
components retain their original notices and licences.

| Component | Source / author | Licence and notice |
|---|---|---|
| Virgin engine | [parlue/virgin](https://github.com/parlue/virgin), Dirk D. Sommerfeld | Original MIT notice in [src/LICENSE](src/LICENSE). |
| Merida chess pieces | Armando Hernandez Marroquin; WebP sources under `assets/merida` | GPL-2.0 or later; used here under GPL-3.0. See `esp32_core2/main/PIECES-LICENSE.txt` and `LICENSE`. |
| Logo queen moods | Dirk D. Sommerfeld / BeatVirgin; sources under `assets/queen_moods` | Included as part of this GPL-3.0 firmware package. |
| Certabo parser, calibration, startup and LED implementation | [parlue/eChessGw](https://github.com/parlue/eChessGw); protocol references identified in the source | Included as part of this GPL-3.0 firmware package. `certabo_gateway_sources.json` records SHA-256 values of the imported gateway files. |
| Certabo protocol reference | [gkalab/cer2nut](https://github.com/gkalab/cer2nut), revision `80ce6c8cdf89a60a7940e2233ea0bf85e667237c` | GPL-3.0 reference implementation; this project retains the gateway's independent bounded-memory parser and original attribution. |
| Spectrum mood colours and blue correction | chesstactics `talLedColour` / `spectrumTint`, Dirk D. Sommerfeld | Ported into the Core2 adapter. |
| PeSTO piece-square tables | Ronald Friederich, RofChade/PeSTO; [published evaluation tables](https://www.chessprogramming.org/PeSTO%27s_Evaluation_Function) | Attribution and table orientation are retained in `src/pst_pesto.h`. The source identifies the wiki data as CC BY-SA 3.0. |
| M5Unified | [m5stack/M5Unified](https://github.com/m5stack/M5Unified) | MIT, `licenses/M5Unified.txt` and the pinned submodule's licence. |
| M5GFX | [m5stack/M5GFX](https://github.com/m5stack/M5GFX) | MIT, `licenses/M5GFX.txt` and the pinned submodule's licence. |
| esp-nimble-cpp | [h2zero/esp-nimble-cpp](https://github.com/h2zero/esp-nimble-cpp) | Apache-2.0, `licenses/esp-nimble-cpp.txt` and notices in the pinned submodule. |

Pinned library revisions are listed in [dependencies.json](dependencies.json).
The build also downloads ESP-IDF and managed components under their respective
upstream licences; the platform and component lock file are checked in.
