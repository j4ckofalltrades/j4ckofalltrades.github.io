---
title: "Getting permalinks to files in GitHub"
description: "Getting a permalink to a file in a specific commit in GitHub"
date: 2024-10-27T18:07:57+08:00
draft: false
categories: 
tags:
  - "til"
series: 
images: 
featured_image:
  url: 
  alt_text: 
mastodonPostID:
---

When viewing a file on GitHub, pressing the `y` key will update the URL to a permalink to the current revision you're viewing.

The default the version of a file that you see is the at the current head of a branch:

> https://github.com/j4ckofalltrades/j4ckofalltrades/blob/main/README.md

By using this keyboard shortcut you get a permalink to the specific version of the file which contains the `SHA` commit hash in the URL:

> https://github.com/j4ckofalltrades/j4ckofalltrades/blob/6afb8559a813123da1cd85142410169c825bee40/README.md

<br />

{{< notice tip >}}
Pressing `?` on any page in GitHub shows all the available keyboard shortcuts.
{{< /notice >}}
