# Line output, host MIDI, and related repositories

This is a fork of [NickCulbertson/Mini-Teensy-Synth](https://github.com/NickCulbertson/Mini-Teensy-Synth). USB audio, USB device MIDI, the LCD, the presets, and the existing encoder functions stay. The additions are a PCM5102A line output and a USB host buffer large enough for a Teensy step sequencer.

## Related repositories

- Upstream synth: https://github.com/NickCulbertson/Mini-Teensy-Synth
- Step sequencer used with this build: https://github.com/youbecha/Teensy-Arduino-Step-Sequencer

The sequencer sends USB MIDI as a device. Plug its device port into this synth's USB host port. A computer is not required between them. The synth's host object is `MIDIDevice_BigBuffer`, because a Teensy sends 512-byte high-speed packets. A normal USB keyboard uses 64-byte packets and still works.

## Libraries

Installed with Teensyduino:

- Teensy Audio library
- USBHost_t36

Installed from the Arduino Library Manager:

- LiquidCrystal I2C
- Encoder, by Paul Stoffregen

Board: Teensy 4.1. USB type: Audio + MIDI. `USE_USB_AUDIO`, `USE_PCM5102`, `USE_USB_DEVICE_MIDI`, and `USE_MIDI_HOST` are enabled in `config.h`. Leave `USE_TEENSY_DAC` off. That option is the Audio Shield codec, not this DAC.

## PCM5102A wiring

The module is the common purple GY-PCM5102 board. Its VIN pin feeds an onboard regulator, so it takes 5 V from the Teensy VIN pin. The chip itself runs at 3.3 V. The three I2S signals are 3.3 V logic.

| PCM5102A | Teensy 4.1 |
|---|---|
| DIN | 7 |
| LRCLK (LCK) | 20 |
| BCLK (BCK) | 21 |
| SCK | GND on the module |
| GND | GND |
| VIN | VIN (5 V) |

Solder the four rear bridges L, L, H, L. The third bridge is XSMT and must be high, or the jack stays muted. The fourth bridge is the data format and must stay low for I2S. DEMP, the second bridge, stays low.

Both jack channels carry the same mono mix. Use a stereo cable into a powered speaker, an amp, or headphones. `PCM5102_LINE_GAIN` is `0.10`, which keeps a portable speaker from clipping. The chip has no volume pin.

These encoder wires moved off the I2S pins onto bottom pads. Pads 42–47 are the SD-socket pads and are not used.

| Encoder wire | Was | Now |
|---|---|---|
| enc5 DT, Osc2 Fine | 7 | 49 |
| enc11 DT, Osc3 Volume | 20 | 52 |
| enc11 CLK, Osc3 Volume | 21 | 53 |

Pin 23 stays the Filter Decay encoder clock. The audio library wants to drive a master clock out of that pin. This sketch gives the pin back to the encoder and lets the PCM5102A derive its clock from BCLK.
