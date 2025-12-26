---
title: "Obsidian Tab Numbers"
date: 2025-12-26T17:48:56+08:00
description: Obsidian plugin that shows tab number hints when the Ctrl or Cmd key is held
draft: false
tags:
  - obsidian
  - plugin
categories:
series:
asciinema: false
mastodonPostID:
---

I wrote a simple Obsidian plugin complements the default Obsidian
keyboard shortcuts for switching tabs (`Ctrl/Cmd 1-8`, `9` selects the last tab)
by adding tab number _hints_.

This is similar to how I switch between browser tabs using the same keyboard
shortcuts. I tend to have around ~5 tabs open at most so that I can see the
title of the tab I'm switching to.

The tab number _hints_ only show up when the `Ctrl/Cmd` key is held down and
the text and background colors can be customized via the plugin settings.

Check out the code on [GitHub](https://github.com/j4ckofalltrades/obsidian-tab-numbers).

## Installation

Manual installation is currently the only supported method but there is an open PR
to add it to the [plugin directory](https://github.com/obsidianmd/obsidian-releases/pull/8535).

1. Download the following files `main.js`, `manifest.json`, `styles.css` from
the latest release on [GitHub](https://github.com/j4ckofalltrades/obsidian-tab-numbers/releases/).
2. Copy the files to your Obsidian vault's plugin directory `/path/to/your/vault/.obsidian/plugins/tab-numbers/`.
3. Reload Obsidian by opening the Developer Console with `Ctrl/Cmd + Shift + I` and running `app.commands.executeCommandById('app:reload')`
or restart Obsidian.
4. Navigate to `Settings -> Community plugins` and enable the "Tab Numbers" plugin.

## Screenshots

![Obsidian window with 3 open tabs showing tab numbers](https://res.cloudinary.com/j4ckofalltrades/image/upload/v1766659104/foss/obsidian-tab-numbers-demo_cnxsza.png)
