---
title: Ikki68 Aurora build
date: 2023-02-18T12:21:19+08:00
description: Ikki68 Aurora keyboard build
draft: false
categories:
  - keyboards
tags:
  - qmk
  - via
  - 65_percent
series:
  - Custom mechanical keyboard builds
images:
  - "https://res.cloudinary.com/j4ckofalltrades/image/upload/v1676740613/keebs/ikki68_aurora_xa8pcq.jpg"
mastodonPostID: 109885918444766509
---

> Ikki68 Aurora is an affordable, injection molded, polycarbonate case, gasket-mount keyboard kit from Wuque Studio, the people behind Ikki68, OA Switches and more!
>
> Every Aurora comes with a multi-layout, hot swap PCB powered by QMK and VIA. You can choose any or all of the following: split backspace, split right shift, split left shift, split spacebars and ISO enter...no soldering required!
>
> The Aurora uses a variable gasket structure where you can choose how many mounting points to use in order to adjust the amount of bounce / firmness to your liking. You can also customize the keyboard with one of many extra badge designs in several different colors!

This board is great bang for your buck, with a lot of configuration options with regards to mounting points -- it also has custom badges for more personalization.

For this build I used some **Akko Diced Fruit Kiwi** switches that I won from a raffle at a meetup I attended from a while back. The switches came lubed and filmed, so that saved me a fair chunk of time. I really like the feel of these switches, very smooth to type on yet still being tactile -- you still feel that *bump* that let's you know your keypress has been registered.

As for the keycaps, the **DCS Solarized Dark** really fits in well with the blue polycarbonate case. I'm a big fan of the color theme, and just had to get these -- one thing to note is that the keycaps are quite a bit thinner than the others that I have used though the overall look and feel is good.

Keyboard layout is a standard-ish 65%, with some common customizations I configured for most of my other boards:

- `Caps Lock` remapped as soft escape (Esc when held, Ctrl when tapped)
- Vim-style arrow keys
- Media player controls

As well as home-row modifiers which I've adopted to all my other boards (since building the [NCR-80](/posts/2022/ncr-80)).

```bash
# left side modifiers
MT(MOD_LCTL, KC_S), MT(MOD_LGUI, KC_D), MT(MOD_LALT, KC_F)

# right side modifiers
MT(MOD_RALT, KC_J), MT(MOD_RGUI, KC_K), MT(MOD_RCTL, KC_L)
```
