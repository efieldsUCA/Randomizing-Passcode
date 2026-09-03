# Randomizing Passcode — Ciphered Lock

An embedded "solve-for-it" cipher lock built on a Raspberry Pi Pico W. Instead of
entering a fixed code, the user solves a small puzzle at each step: the board shows
two operand symbols and an operation, and the user presses the symbol whose value is
the answer. The puzzle reshuffles after every unlock and after every lockout, so the
input is different every time.

Built and demonstrated with live LED, buzzer, and RGB-eye feedback, and a custom PCB
that takes the circuit from breadboard to a fabricated board.

## How the cipher works

The interface is eight Greek letters (α β γ δ ε ζ η θ), each with a value (α=1 … θ=8).
For each step of the code:

1. Two operand letters light up on the 74HC595 LED bank.
2. The RGB eyes flash an operation color — **purple = add, yellow = subtract, orange = multiply**
   — then settle back to blue "entering" status.
3. The user computes the result, wrapping onto 1–8 (over 8 subtract 8; under 1 add 8),
   and presses the letter whose value equals the answer.

A correct press advances to the next step; a wrong press reshuffles the whole code.
Complete the full sequence and the lock opens, then reshuffles and re-arms. Too many
wrong attempts triggers a timed lockout, which also reshuffles.

The security concept: making the letter→value **mapping secret** (known only to the
owner) is what makes the lock hard to solve even when the hints are visible.

## Status indicators

| State | Eyes | LEDs | Sound |
|-------|------|------|-------|
| Armed (locked) | slow-breathe red | off | — |
| Entering | solid blue (operation-color flash per step) | operand letters lit | key beep |
| Unlocked | solid green | all on | rising tone |
| Locked out | flashing red | blinking | descending tone |

## Hardware

- Raspberry Pi Pico W (hosts its own Wi-Fi access point for the keypad)
- 74HC595 8-bit shift register → 8 status/hint LEDs
- Two common-cathode RGB LEDs ("eyes"), PWM-driven
- Passive piezo buzzer
- Current-limiting resistors, decoupling capacitor
- Custom PCB (schematic and board files in `/docs`)

## Software architecture

The firmware is written in MicroPython and is fully **non-blocking** — every peripheral
advances one step per loop pass off `time.ticks_ms()`, so the Wi-Fi keypad, the buzzer
melodies, and the eye animations all run concurrently without freezing each other.

| File | Role |
|------|------|
| `main.py` | Wires the peripherals and runs the non-blocking loop |
| `passcode.py` | The cipher brain: state machine + puzzle generation (`CONFIG` at top) |
| `webkeypad.py` | Pico W access point + web keypad (non-blocking `poll()`) |
| `eyes.py` | RGB eyes with solid / flash / breathe effects |
| `shift_leds.py` | 74HC595 driver (show / progress / blink) |
| `buzzer.py` | Non-blocking melody player |

State machine: `LOCKED → ENTERING → UNLOCKED / LOCKED_OUT`.

## Running it

1. Flash MicroPython to a Raspberry Pi Pico W and upload all `.py` files.
2. Run `main.py`. The console prints the Wi-Fi details and the first puzzle.
3. On a phone, join the Wi-Fi network **PicoLock** (password `picolock`).
4. Open a browser to **http://192.168.4.1** and use the on-screen keypad.

For the demo, the current solution is printed to the serial console each time it
reshuffles. In a finished product the code is conveyed through the hardware hint
mapping, not the console.

## Documentation

- `/docs` — schematic, PCB layout, and design notes.

## Configuration

Everything tunable lives in the `CONFIG` dict at the top of `passcode.py`: the letter
values, which operations are in play, the operation colors, code length, attempt limit,
and lockout timing.

## License

MIT
