---
draft: false
date: 2025-09-27T22:55:46+08:00
title: "Are we Trusted Publishing yet?"
description: "A listing of package registries that support Trusted Publishing"
tags:
  - trusted-publishing
---

[Trusted Publishing](https://repos.openssf.org/trusted-publishers-for-all-package-repositories) is a recommended security capability by the OpenSSF Securing Software Repositories Working Group as it removes the need to securely manage an API token in the build system.

[pub.dev](https://pub.dev), [PyPI](https://pypi.org), and [RubyGems](https://rubygems.org) added support in 2023 and other package registries have adopted it since. The following package registries (that I know of) support Trusted Publishing:

- [pub.dev](https://dart.dev/tools/pub/automated-publishing)
- [PyPI](https://docs.pypi.org/trusted-publishers/using-a-publisher/)
- [RubyGems](https://guides.rubygems.org/trusted-publishing/)
- [JSR](https://jsr.io/docs/publishing-packages#publishing-from-github-actions)
- [crates.io](https://crates.io/docs/trusted-publishing)
- [npm](https://docs.npmjs.com/trusted-publishers)
- [NuGet](https://learn.microsoft.com/en-us/nuget/nuget-org/trusted-publishing)

I built a [site](https://jduabe.dev/are-we-trusted-publishing-yet) that shows a timeline of when each package registry added support for Trusted Publishing.

{{< alt-text src="https://res.cloudinary.com/j4ckofalltrades/image/upload/v1758984405/are-we-trusted-publishing-yet_s0e0du.png" description="Screenshot of the main page of Are we Trusted Publishing yet" >}}
