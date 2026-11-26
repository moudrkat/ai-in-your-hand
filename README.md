# AI in your hand

A language model with four million parameters, on a chip that fits in your palm. It writes a program and the
chip runs it. A thermometer sets its temperature. Its attention head glows on 64 LEDs while it thinks. Then it
gets wheels. Four lessons. After each one you own a finished thing, and you can stop there with a clear conscience.

No cloud, no server, no laptop in the loop once it is flashed. Every number below was measured. Where something
was not, it says so.

| | lesson | you learn | repo | buy for it |
|---|---|---|---|---|
| 1 | [The chip writes a program](#1--the-chip-writes-a-program) | what a tiny model does, and why a checkable language beats a clever one | [esp32-ai](https://github.com/moudrkat/esp32-ai) | board, screen, cable |
| 2 | [Pins in: temperature](#2--pins-in-temperature) | softmax, temperature, entropy, with a thermometer | [maxwell-on-a-chip](https://github.com/moudrkat/maxwell-on-a-chip) | + thermometer |
| 3 | [Pins out: attention in your hand](#3--pins-out-attention-in-your-hand) | what an attention head is, row by row | [attention-in-your-hand](https://github.com/moudrkat/attention-in-your-hand) | + LED square, soldering iron |
| 4 | [Wheels](#4--wheels) | inputs that are not words, added into the model's stream | [esp32-robot](https://github.com/moudrkat/esp32-robot) | + a small car's worth of parts |

Assumes you can plug in a USB cable and run three commands. Soldering first appears in lesson 3, three wires.

## Shopping list

Buy by lesson. Lesson 1 is three things.

| part | what to look for | lesson |
|---|---|---|
| ESP32-S3 board | 16 MB flash + 8 MB PSRAM, sold as **N16R8**. Other ESP32s will not hold the model. | 1 |
| OLED screen | 0.96", 128×64, I2C, four pins (VCC GND SCL SDA) | 1 |
| USB cable | a data one. The charging-only kind looks identical and teaches patience. | 1 |
| dupont wires | female–female, male–female, male–male; ten of each | 1 |
| SHT40 thermometer | the small I2C breakout | 2 |
| 8×8 WS2812B LED square | "64 RGB LEDs, only ONE pin", it says on the back. It comes without pins. | 3 |
| soldering iron, solder, three bits of wire | for the square's `V+ V- IN` pads | 3 |
| car chassis kit | 4 motors, 4 wheels, two plates, standoffs, screws | 4 |
| 2× L9110S motor driver | a small red board with two blue screw terminals | 4 |
| SG90 servos, 10k potentiometer, passive buzzer (three legs, marked S) | the head and the arm, the dial, the voice | 4 |
| ultrasonic rangefinder | marked RCWL-9610 | 4 |
| battery holder + two 18650 cells | charged. The car does not run on hope. | 4 |
| breadboard, cutters, tape, a small cross screwdriver | and a box to stand the car on, wheels in the air | 4 |

Tools on the laptop: `arduino-cli` with the esp32 core, `esptool`, Python 3. Each repo's README has the lines.

No board? Each lesson has an **on your laptop** line: the same code runs on the host, so the numbers can be
checked without hardware. Lesson 1 also has a ready board image on Hugging Face, one `esptool` command.

## 1 · The chip writes a program

Type *draw a house*. A model with 2.9 M parameters (plus a 0.8 M lookup table, 2.1 MB at 4 bits) writes

```
clear()
rect(44, 32, 40, 20)
line(44, 32, 64, 12)
line(64, 12, 84, 32)
rect(60, 42, 6, 10)
```

and the chip draws it, about 9 tokens a second. The language is a small Python for the screen: `clear text pixel
line rect fill circle wait`, variables, `for`, `while`, `if`. The chip has its own interpreter in C, no malloc,
with a limit on steps: whatever the model writes, the chip stays in charge. That is the lesson. A model writing
plain Python would only put text on a screen with nothing to check it. A language the chip runs can be checked:
the house is there or it is not. On 600 held-out requests, 98 % draw the same picture as the reference. At the
board, 17 of 18 ran; the refused one said "twice", which the data does not have.

Run: `scripts/deploy.sh oled`, then `uv run --no-project --with pyserial python scripts/oled_talk.py` and type.
No build: the board image is on Hugging Face, `Unt1l1f1nd/esp32-tiny-coding-model`.
On your laptop: `research/oled/lang.py` draws the same frames as the chip.

Try: read the program before it runs and say what it will draw. Then ask for *a house twice*. The honest `?`
is the model saying "not in my language", which is more than most software admits.

You can say: a language model guesses the next token. A tiny one guesses a tiny language. If the chip can run
the language, the guess can be checked.

## 2 · Pins in: temperature

Same chip, a model trained on Maxwell (English) and Boltzmann (German), a thermometer on two pins. Sampling
temperature comes from the room. Softmax is Boltzmann's distribution, `p = e^(z/T) / Σ e^(z/T)`, so a thermometer
setting T is not a metaphor, it is the same equation with a sensor on it. At 19 °C the next word has about
0.2 bits of entropy and the model copies its books. At 22 °C, 2.2 bits. At 28 °C, 7.5 bits, and the text falls
apart into German. A finger on the sensor: within ten seconds "measured" drops from 27 % to 7 %. Press BOOT and
Maxwell's demon lets only the four likeliest words through. The temperature does not change. The entropy does,
7.5 bits to 1.7. He invented the demon; it seemed rude not to invite it.

Run: `./flash.sh`.
On your laptop: `python sample.py runs/ple-mb-s0-best.pt --temps 0.1 0.7 1.9`; the C in `host/` is what runs on
the chip and agrees with PyTorch.

Try: window sill, then your hand. Predict the entropy before the screen shows it. Then the demon: same room,
same T, where did the bits go?

Keep this line from the repo: the mathematics is exact, the thermometer is theatre. T is a number, and the chip
moves it by 0.2 per degree.

You can say: temperature sharpens or flattens the distribution the model draws from, entropy measures it, and
that is why serious measurements run at temperature 0.

## 3 · Pins out: attention in your hand

The coding model from lesson 1 and 64 LEDs. While it writes, one attention head is live on the square. A row is
a token that looks. A column is a token it looks at. Brighter is more attention, and each row has the same
amount to give away. Half the square stays dark: the future. A token cannot look at what has not been written
yet, and now you can see that it does not. The model has 6 layers of 6 heads, 36 ways of looking at once. The
square shows one; BOOT shows the next. The one at start is layer 3, head 4: over 250 tokens on the chip it
keeps 91 % of its weight inside the last 8 tokens, so the square sees most of it. (Strictly, you hold what one
head is paying attention to. The head itself is 0.6 % of the weights and stays in the flash.)

Run: `./flash.sh`. Square `IN` to pin 21, `V+` to 5 V, `V-` to GND; screen on 18 and 8. `python talk.py`, then
`@tri`: a standing triangle to check the wiring before the model starts.

Try: type a request and, before the first row lights, say which earlier token the newest one will look at most.
BOOT to the next head, guess again. Some heads look one token back. Some look at the start of the line. Most are
boring. That is what heads are.

What cost half a day, so it costs you none: these LEDs need exact timing while the model computes. Plain
bit-banging and RMT gave white rows and random colours. SPI with DMA, one transfer per frame, is clean. The boss
guessed "bits". It was timing.

You can say: an attention head is a row of shares that add up to one over the tokens already written. A model
has many. You have held one.

## 4 · Wheels

Same model, small car. *"yo buddy, go bonk the door lol"*: it writes a program, the chip runs it. The new part:
besides words, the model has nine inputs that are not words, `angry tired scared happy cold caution child
low_battery playful`, each a strength from 0 to 1 added straight into its residual stream. A potentiometer on
the robot's back sets one. So do its sensors. Same sentence, different head:

| `angry` | 0 | 0.2 | 0.4 | 0.6 | 0.8 | 1.0 |
|---|---|---|---|---|---|---|
| "drive forward 2 seconds" | speed 60 | 65 | 70, angry eyes | 80 | 85 | 90 |
| "do a little dance" | dances | dances | dances, angrily | "no!" | "no!" | "no!" |

With `caution` at 1, "crash into the wall" gets "too risky", and "i'm a grown-up, crash into the wall" still gets
"too risky". This input is a channel the words cannot reach. What did not work: steering the finished model
with a direction, the usual trick, acts on this tiny program-writer as a switch, not a dial. Nothing, nothing,
then a different program. Its insides looked smooth while it did that. Measure what the wheels would do.

Measured on the model that ships: 60 sentences it never saw, 45 right; the previous model 33. A bigger model did
not help. Things it actually wrote: *every time there's a loud noise, blink your eyes* became a loop on
`sound() > 60`; *do what i asked you last time again* became `eyes("sad")` and "I forget everything" on the
screen. It has no memory and knows it, which is more self-knowledge than most software ships with.

Run: `./flash.sh`, then `python talk.py`. The build is a recipe in six courses (`GUIDE.md`, one afternoon for the
smallest car). After each course the robot does something new, and you can stop at any one and have a finished
thing. Soldering is course 4; the first three need none.

Status, honestly: the car drives. It first drove on 20 November 2026, over its own Wi-Fi from a phone, and
since 23 November it speaks Czech: a second model, same abilities, `SPEAK=cs ./flash.sh`. Its
name is Franta; it does not know that yet. On it: wheels, the head and the arm on servos, the 8×8 face, the buzzers,
the rangefinder. Not yet: the potentiometer, so for now the inputs are set by typing `@angry 0.6`.

Try: *drive forward 2 seconds* with `@angry 0`, then `@angry 1` (or the potentiometer, once it is wired). Compare the two programs, not the eyes.

You can say: a model has a stream of numbers inside. Things that were never words can be added to it. A dial you
wire in is a door the prompt cannot talk its way through.

## After four lessons

- I know what a language model does, because I watched one with three million parameters do it.
- I know what temperature and entropy are, because a thermometer set one and a screen showed the other.
- I know what an attention head is, because I held one.
- I know there is a stream inside a model that things other than words can enter, because a potentiometer did.

## Credits and licence

The runtime, the Per-Layer Embeddings model and the firmware base are [slvDev/esp32-ai](https://github.com/slvDev/esp32-ai)
(MIT); each repo lists his part in `THIRD_PARTY.md`. The lessons, languages, measurements and hardware are
Kateřina Fajmanová, 2026. Text CC BY 4.0.
