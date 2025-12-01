---
draft: false
title: Hugo alt-text shortcode
date: 2025-12-02T12:21:00+08:00
description: "Hugo alt text short code"
tags:
  - hugo
  - alt-text
---

I wrote a simple custom `alt-text` shortcode for my blog that adds an "alt-text" button to images that displays a modal with the media description when clicked. This feature seems to be very common Mastodon web and mobile clients like [Phanpy](https://phanpy.com) and [Ice Cubes](https://github.com/Dimillian/IceCubesApp), both of which I use and can highly recommend.

The following markdown snippet renders an image with a clickable "ALT" button in the bottom left (additional spaces added between curly and angle brackets so it gets treated as text). 

```text
{{ < alt-text src="/images/example.jpg" description="Media description that appears when users click the ALT button" > }}
```

{{< alt-text src="https://res.cloudinary.com/j4ckofalltrades/image/upload/v1764607886/hugo-alt-text-btn_pwkggu.png" description="Screenshot of a slide showing a signed git commit output from git log and an image of a verified commit in GitHub. The image is rendered with the custom alt-text shortcode that renders an ALT button in the bottom left of the image." >}}

{{< alt-text src="https://res.cloudinary.com/j4ckofalltrades/image/upload/v1764608455/hugo-alt-text-modal_dy6c8a.png" description="A modal showing the media description of an image when the ALT button is clicked." >}}

Check out the source code and installation instructions on [GitHub](https://github.com/j4ckofalltrades/hugo-alt-text) and you can find me on Mastdon at [@j4ckofalltrades@infosec.exchange](https://infosec.exchange/@j4ckofalltrades).
