# Firmware notes (QMK + Plover)

Working notes for when the board is fabbed and it's time to write firmware. Nothing here is built yet.

## The core idea

Pinchord isn't a normal "one key, one letter" layout — it's a steno-style chording system. Its 24 keys are steno keys, grouped into prefixes (2), initials (7), vowels (5), finals (8) and suffixes (2), which is exactly where the 24 comes from. The chord → word translation lives in a *dictionary*, not on the keyboard.

Upstream officially supports two runtimes:

- **Javelin** — the dictionary engine runs on the board itself. Wants 8–16 MB of flash, but the stock RP2040-Zero only has 2 MB, so this isn't practical for this board.
- **Plover** — the same engine as a desktop app.

So the QMK plan is: **QMK is the steno machine, Plover does the translation.** QMK has built-in [stenography support](https://docs.qmk.fm/features/stenography) and speaks the Gemini PR protocol. The Pinchord Plover plugin already ships a Gemini PR keymap (`plover_pinchord/system.py`), so QMK sends raw chords over Gemini PR and Plover turns them into words. The keyboard never needs the dictionary.

## 1. QMK side

`rules.mk`:

```make
STENO_ENABLE = yes
STENO_PROTOCOL = geminipr
```

Assign each physical key a `STN_*` keycode. This table is the Pinchord plugin's Gemini PR keymap translated into QMK keycodes:

| Pinchord key | Bank    | QMK keycode |
|--------------|---------|-------------|
| `#`          | prefix  | `STN_S1`    |
| `+`          | prefix  | `STN_S2`    |
| `T`          | initial | `STN_TL`    |
| `S`          | initial | `STN_KL`    |
| `P`          | initial | `STN_PL`    |
| `W`          | initial | `STN_WL`    |
| `H`          | initial | `STN_HL`    |
| `R`          | initial | `STN_RL`    |
| `&`          | initial | `STN_ST1`   |
| `A`          | vowel   | `STN_A`     |
| `O`          | vowel   | `STN_O`     |
| `I`          | vowel   | `STN_E`     |
| `U`          | vowel   | `STN_U`     |
| `^`          | vowel   | `STN_ST3`   |
| `L`          | final   | `STN_FR`    |
| `R`          | final   | `STN_RR`    |
| `P`          | final   | `STN_PR`    |
| `K`          | final   | `STN_BR`    |
| `F`          | final   | `STN_LR`    |
| `S`          | final   | `STN_GR`    |
| `T`          | final   | `STN_TR`    |
| `*`          | final   | `STN_SR`    |
| `e`          | suffix  | `STN_DR`    |
| `s`          | suffix  | `STN_N1` *(see note)* |

Which GPIO each keycode sits on depends on how the switches wire to the RP2040-Zero — read that off `pinboard.kicad_sch`. The keycodes are the Pinchord-specific part; the matrix is just a normal QMK keyboard definition (direct pins, no diodes).

### The `-s` catch

Gemini PR doesn't have enough keys to cover all 24 by default, so the upstream keymap leaves the `s` suffix unmapped. Workaround: borrow a spare Gemini PR key — send `STN_N1` (the `#1` key) from the physical `s` key in QMK, and bind `-s` to `#1` in Plover's Gemini PR machine keymap. Skip this and everything works except the `s` suffix.

## 2. Plover side

1. Install [Plover](https://www.openstenoproject.org/plover/) 4.0+.
2. Install the Pinchord plugin — clone <https://grahp.dev/pinchord.git> and `pip install .` into Plover's environment, or add it via the Plugins Manager.
3. Set **System → Pinchord** and **Machine → Gemini PR**.
4. If you used the `-s` workaround, bind `-s` to `#1` in the Gemini PR machine config.
5. Enable output and start chording.

## Source references

- Pinchord repo: <https://grahp.dev/pinchord.git> — keymap lives in `plover_pinchord/system.py` (`KEYMAPS["Gemini PR"]`), chords in `chords.edn` / `pinchord-chords-vX.Y.json`.
- Setup guide: <https://grahp.dev/setting-up-pinchord>
- QMK steno keycodes: the `STN_*` enum (GeminiPR section of QMK's steno keycodes).
