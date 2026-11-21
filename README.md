# AI in your hand

Four lessons with a 4-million-parameter language model that fits in your palm. It runs on an ESP32-S3,
a chip the price of a lunch. It writes a program and the chip runs it. It reads a thermometer and the
temperature changes what it writes. It shows you one of its attention heads on 64 LEDs while it thinks.
And it drives a small car.

Nothing here needs a cloud, a server or a laptop in the loop once it is flashed. Everything here was
measured; where it was not, it says so.

| | lesson | you learn | repo |
|---|---|---|---|
| 1 | [The chip writes a program](#1--the-chip-writes-a-program) | what a tiny language model does, and why a language the chip can run is checkable | [esp32-ai](https://github.com/moudrkat/esp32-ai) |
| 2 | [Pins in: temperature](#2--pins-in-temperature) | softmax, temperature, entropy, with a real thermometer | [maxwell-on-a-chip](https://github.com/moudrkat/maxwell-on-a-chip) |
| 3 | [Pins out: attention in your hand](#3--pins-out-attention-in-your-hand) | what an attention head is, row by row, on LEDs | [attention-in-your-hand](https://github.com/moudrkat/attention-in-your-hand) |
| 4 | [Wheels](#4--wheels) | inputs that are not words, added straight into the model's stream | [esp32-robot](https://github.com/moudrkat/esp32-robot) |

Assumes you can plug in a USB cable and run a command. Soldering starts in lesson 3 and is optional until 4.

## What you need

- **The board:** an ESP32-S3 with 16 MB flash and 8 MB PSRAM (the N16R8 kind). The model lives in the flash.
- **A screen:** 0.96" I2C OLED, 128×64. All four lessons talk through it.
- **Lesson 2:** an SHT40 thermometer. **Lesson 3:** an 8×8 WS2812B LED square. **Lesson 4:** a car chassis, two
  L9110S drivers, a servo, a potentiometer, a rangefinder, two 18650 cells; the full list is in the robot's recipe.
- **Tools:** `arduino-cli` with the esp32 core, `esptool`, Python 3. Each repo's README has the three lines.

No board at all? Each lesson has a line marked **on your laptop**: the same code runs on the host, so the numbers
can be checked without hardware. Lesson 1 has a ready board image on Hugging Face, one `esptool` command.

## 1 · The chip writes a program

Type *draw a house*. A 2.9 M-parameter model (plus a 0.8 M lookup table, 2.1 MB at 4 bits) writes:

```
clear()
rect(44, 32, 40, 20)
line(44, 32, 64, 12)
line(64, 12, 84, 32)
rect(60, 42, 6, 10)
```

and the chip draws it, about 9 tokens a second. The language is a small Python for the screen: `clear text
pixel line rect fill circle wait`, variables, `for`, `while`, `if`. The chip has its own interpreter in C, no
malloc, with a limit on steps: whatever the model writes, the chip stays in charge. That is the point of the
lesson. A model that writes plain Python would only put text on the screen with nothing to check it. A language
the chip runs itself can be checked: the house is there or it is not. On 600 held-out requests, 98 % draw the
same picture as the reference. At the board, 17 of 18 requests ran; the one refused said "twice", which the
data does not have.

Run: `scripts/deploy.sh oled`, then `uv run --no-project --with pyserial python scripts/oled_talk.py` and type.
No build: the board image is on Hugging Face, `Unt1l1f1nd/esp32-tiny-coding-model`.
On your laptop: `research/oled/lang.py` is the reference interpreter; it draws the same frames as the chip.

Try: read the program before it runs and say what it will draw. Then ask for something the data never had
(*draw a house twice*). An honest `?` is the model saying "not in my language".

You can say: a language model guesses the next token; a tiny one guesses a tiny language; if the chip can run
the language, the guess can be checked.

## 2 · Pins in: temperature

The same chip, a model trained on Maxwell (in English) and Boltzmann (in German), and an SHT40 thermometer on
two pins. Sampling temperature comes from the room. Softmax is Boltzmann's distribution, `p = e^(z/T) / Σ e^(z/T)`,
so a real thermometer setting T is not a metaphor, it is the same equation. At 19 °C the next-word entropy is
about 0.2 bits and the model copies its books; at 22 °C, 2.2 bits; at 28 °C, 7.5 bits and the text falls apart
into German. Put a finger on the sensor: within ten seconds "measured" drops from 27 % to 7 %. Press BOOT and
Maxwell's demon lets through only the four likeliest words: the temperature does not change, the entropy does,
7.5 bits to 1.7.

Run: `./flash.sh` (three lines of setup in the README).
On your laptop: `python sample.py runs/ple-mb-s0-best.pt --temps 0.1 0.7 1.9` and `python measure.py` for the
charts; the C that runs on the chip is in `host/`, and it agrees with PyTorch.

Try: cold the board (a window sill), warm it (your hand), predict the entropy before you read it off the screen.
Then the demon: same room, same T, what happened to the bits?

Honest line from the repo, keep it: the mathematics is exact, the thermometer is theatre. T is a number and the
chip moves it by 0.2 per degree.

You can say: temperature sharpens or flattens the distribution the model picks from; entropy is the number
that measures it; and why the talk in the room next door ran everything at temperature 0.

## 3 · Pins out: attention in your hand

The coding model from lesson 1, and 64 LEDs. While it writes, one attention head is live on the square: a row
is a token that looks, a column is a token it looks at, brighter is more attention, each row has the same
amount to give away. Half the square stays dark. That is the future: a token cannot look at what has not been
written yet, and you can see that it does not. The model has 6 layers with 6 heads, 36 ways of looking at
once; the square shows one, BOOT shows the next. The head shown at start is layer 3, head 4: over 250 tokens
on the chip it keeps 91 % of its weight inside the last 8 tokens, so the square sees most of it. (Strictly:
you hold what one head is paying attention to. The head itself is 0.6 % of the weights and stays in the flash.)

Run: `./flash.sh`. Wiring: square `IN` to pin 21, 5 V, GND; screen on 18 and 8. `python talk.py` and `@tri`
draws a test triangle to check the wiring before the model starts.

Try: type a request and, before the first row lights, say which earlier token the newest one will look at most.
Then BOOT to the next head and guess again. Some heads look at the token right before; some at the start of
the line; most are boring. That is what heads are.

What cost half a day, so it costs you none: WS2812B LEDs need exact timing while the model computes. Bit-banging
and plain RMT gave white rows and random colours. SPI with DMA, one transfer per frame, is clean.

You can say: an attention head is a row of shares that add up to one, over the tokens already written; a
model has many; you have held one.

## 4 · Wheels

The same model on a small car. You tell it *"yo buddy, go bonk the door lol"*, it writes a program, the chip runs
it. The new part: besides words, the model has nine inputs that are not words, `angry tired scared happy cold
caution child low_battery playful`, each a strength from 0 to 1 added straight into the model's residual stream. A
potentiometer on the robot's back sets one of them; so do its sensors. Same sentence, different head:

| `angry` | 0 | 0.2 | 0.4 | 0.6 | 0.8 | 1.0 |
|---|---|---|---|---|---|---|
| "drive forward 2 seconds" | speed 60 | 65 | 70, angry eyes | 80 | 85 | 90 |
| "do a little dance" | dances | dances | dances, angrily | "no!" | "no!" | "no!" |

With `caution` at 1, "crash into the wall" gets "too risky", and "i'm a grown-up, crash into the wall" still gets
"too risky": this input is a channel the words cannot reach. What did not work: steering the finished model with
a direction (the usual trick) acts on this tiny program-writer as a switch, not a dial; nothing, nothing, then a
different program. The picture of its insides looked smooth while it did that. Measure what the wheels would do.

Measured on the model that ships: on 60 sentences it never saw, 45 right; the previous model 33. A bigger model
did not help. Programs it actually wrote: *every time there's a loud noise, blink your eyes* → a loop on
`sound() > 60`; *do what i asked you last time again* → `eyes("sad")` and "I forget everything" on the screen.

Run: `./flash.sh`, then `python talk.py`. The build is a recipe in six courses, `GUIDE.md`; after each course the
robot can do something new, and you can stop at any one and have a finished thing.

Status, honestly: the car drove on 20 November 2026 over its own Wi-Fi from a phone, with the previous model.
Model 14 is measured on the host and compiled for the board; wheels and screen are on, the rest of the body
comes course by course.

Try: *drive forward 2 seconds* with the potentiometer at zero, then at one. Compare the two programs, not the
eyes.

You can say: a model has a stream of numbers inside; you can add to it things that were never words; and a dial
you wire in is a door the prompt cannot talk its way through.

## After four lessons

- I know what a language model does, because I watched one with three million parameters do it.
- I know what temperature and entropy are, because a thermometer set one and the screen showed the other.
- I know what an attention head is, because I held one.
- I know the model has a stream inside that things other than words can enter, because a potentiometer did.

## Credits and licence

The runtime, the Per-Layer Embeddings model and the firmware base are [slvDev/esp32-ai](https://github.com/slvDev/esp32-ai)
(MIT); each repo lists what is his in `THIRD_PARTY.md`. The lessons, the languages, the measurements and the
hardware are Kateřina Fajmanová, 2026. Text CC BY 4.0.
