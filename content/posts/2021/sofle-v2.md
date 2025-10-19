---
title: "Sofle v2 build"
description: "Sofle v2 keyboard build"
date: 2021-09-27T20:24:42+08:00
draft: false
categories:
- keyboards
tags:
- ortholinear
- sofle
- split
- qmk
- via
series:
- "Custom mechanical keyboard builds"
---

The [SofleKeyboard](https://josef-adamcik.cz/electronics/let-me-introduce-you-sofle-keyboard-split-keyboard-based-on-lily58.html) is a 6×4+5 keys column-staggered split keyboard with encoder support. Based on the Lily58, Corne, and Helix keyboards.

First time trying out ortholinear and split mechanical keyboards with this build.

I generally prefer tactile switches, so I went with **AEBoard Naevy v1**'s lubed with **Krytox 205g0** for this build.

Tried out **Katha Baybayin** (KAT), and **Blank Ergo** (DSA) keycaps; both looked good but settled with the latter.

I think I spent more time messing around with my custom keymap than building the keyboard. Coming from mostly TKL layouts, it was going to take a bit of experimenting to find the sweet spot when dealing with fewer keys.

I settled on a layout not too far off from the default one, some notable additions were:

- Mode for soft escape (`Esc` when tapped, `Ctrl` when held)
- Vim-style navigation (`h` `j` `k` `l` as arrow keys)
- Mode for standard navigation cluster
- Toggling between layers when encoders are pressed
- Encoder controls (additional layer/s when `Ctrl` or `Shift` is held)

![SofleKeyboard custom keymap](https://raw.githubusercontent.com/j4ckofalltrades/keebs/master/sofle-v2/assets/soflekeyboard.png)

Aside from configuring the encoder actions (clockwise/counter-clockwise), it is also possible
to add extra "layers" by combining them with modifiers.

| modifier | encoder | action             |
| -------- | ------- | ------------------ |
| N/A      | Left    | Mouse wheel L/R    |
| N/A      | Right   | PgUp/PgDn          |
| Ctrl     | Left    | VolUp/VolUp        |
| Ctrl     | Right   | Brightness Up/Dn   |
| Shift    | Left    | Mouse cursor L/R   |
| Shift    | Right   | Mouse cursor Up/Dn |

One other area for tinkering is the OLED displays where you can show text, and even images e.g. [displaying your own logo](https://docs.qmk.fm/#/feature_oled_driver?id=logo-example).

Check out the full custom keymap [here](https://github.com/j4ckofalltrades/keebs/tree/master/sofle-v2).
