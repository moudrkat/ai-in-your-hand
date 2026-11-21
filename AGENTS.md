# For agents (Claude Code, Codex, Cursor, Copilot…)

This repo is a four-lesson course: a 4 M-parameter language model on an ESP32-S3 that writes programs, reads
a thermometer, shows an attention head on LEDs and drives a car. The user wants to learn by doing. Your job:
get the board flashed, read the outputs with them, explain the numbers in their words, and invent nothing.
Every number in `README.md` is measured; if theirs differ, show both.

## The four repos

| lesson | repo | flash | talk | on the laptop, no board |
|---|---|---|---|---|
| 1 program | `moudrkat/esp32-ai` | `scripts/deploy.sh oled` (or the Hugging Face image with `esptool`) | `scripts/oled_talk.py` | `research/oled/lang.py` reference interpreter |
| 2 temperature | `moudrkat/maxwell-on-a-chip` | `./flash.sh` | the screen | `python sample.py … --temps 0.1 0.7 1.9`, `host/` C |
| 3 attention | `moudrkat/attention-in-your-hand` | `./flash.sh` | `python talk.py`, `@tri`, `@head L H` | none yet; the square is the point |
| 4 wheels | `moudrkat/esp32-robot` | `./flash.sh` | `python talk.py` | `research/robot/` scripts |

Clone the repo of the lesson next to this one and run its commands from its root. Each has `THIRD_PARTY.md`
for what is slvDev's.

## Setup that is the same everywhere

- `arduino-cli core install esp32:esp32` and `arduino-cli lib install "Adafruit SSD1306" "Adafruit GFX Library"`.
- `esptool` on the path (`pip install esptool`), the board on `/dev/ttyACM0` (pass another port to `flash.sh`).
- The N16R8 board (16 MB flash, 8 MB PSRAM). Other ESP32 boards will not hold the model.
- Models are trained on a GPU machine; the user's laptop never needs a model, only the exported `model.bin`.

## What to check before saying "done"

- Lesson 1: the screen shows the program, then the picture. A `?` is a refusal, not a crash.
- Lesson 2: the screen shows the temperature, the phase (SOLID/LIQUID/GAS) and the entropy; BOOT shows DEMON.
- Lesson 3: `@tri` first. A standing test picture that looks right does not prove the timing; let the model
  write and watch for white rows (that is the LED timing problem; SPI with DMA fixes it).
- Lesson 4: the user decides what is wired. Never claim a sensor works that is not on the car.

## Do not

- Do not change training data, models or measured JSON in the repos. Rerun, do not edit.
- Do not describe the thermometer as physics in lesson 2: the maths is exact, the thermometer is theatre, and
  the README says so. Keep that line.
- Do not show the robot doing things it cannot do. If a request fails, say it failed.
