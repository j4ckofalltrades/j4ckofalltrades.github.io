---
title: "NCR-80 build"
date: 2022-02-18T20:24:42+08:00
description: "NCR-80 keyboard build"
draft: false
categories:
- keyboards
tags:
- qmk
- via
series:
- "Custom mechanical keyboard builds"
---

> *NCR-80 is a reimagination of the classic rebranded Cherry G81-3000 by NCR*

I used **Durock Shrimp** switches for this build, just spring-lubed with **Krytox 205g0**, and **Katha Baybayin** keycaps.

I had to be a little bit creative with the keymaps, given I generally use Ctrl, Alt, and Meta keys for keyboard shortcuts. I used Mod-Tap on the home row keys to work around this.

```bash
# left side modifiers
MT(MOD_LCTL, KC_S), MT(MOD_LGUI, KC_D), MT(MOD_LALT, KC_F)

# right side modifiers
MT(MOD_RALT, KC_J), MT(MOD_RGUI, KC_K), MT(MOD_RCTL, KC_L)
```

- `Caps Lock` remapped as soft escape (Esc when held, Ctrl when tapped)
- Vim-style arrow keys
- Layers for Linux/Win and Mac
- Home row modifiers e.g. `MT(MOD_LCT , KC_A)`