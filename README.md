# pinboard

A 24-key unibody keyboard built for chorded typing with the [Pinchord](https://grahp.dev/pinchord) layout.

![pinboard PCB](docs/preview.png)

Pinchord is a chording system where you press a few keys at the same time to spell whole words instead of tapping them out letter by letter. The keys are split into three groups: the left fingers handle the start of a word, the thumbs and right index do the vowels, and the right fingers finish it off.

## Specs

- Support for both Kailh Choc v1 and v2 switches, with both hand-soldering and hotswap sockets supported
- Direct wiring (no diodes) with the help of using RP2040-Zero's pad pins on the back of the board
- Extremely cheap budget build

## Building one

Send the gerbers (see the releases page) off to your fab of choice. Once the boards are there, solder on the RP2040-Zero and optionally the hotswap sockets, then snap in your switches and keycaps.
