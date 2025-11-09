---
title: Trusted Publishing & Digital Attestations in the OSS Ecosystem
date: 2025-09-18T23:19:09+08:00
description: Usage and adoption of trusted publishing and digital attestations in OSS package registries
draft: false
tags:
  - trusted-publishing
  - attestations
---
I recently presented a talk at [Python WA](https://www.meetup.com/pythonwa/events/308746488) about two security features that are available in software registries like PyPI (Python Package Index); trusted publishing and digital attestations and how they increase trust in the software supply chain.

Trusted Publishing is a mechanism for uploading packages that uses the [OpenID Connect (OIDC)](https://openid.net/connect/) standard to exchange short-lived identity tokens between a trusted third-party service and PyPI (and other software registries) instead of long-lived API tokens.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-005.png" description="Trusted Publishing flow" >}}

Enabling a trusted publisher requires different configuration requirements based on your CI/CD platform. The following platforms are supported by PyPI:

- GitHub Actions
- GitLab CI/CD
- Google Cloud
- ActiveState

For GitHub Actions, you **must** provide the repository owner's name, the repository's name, and the filename of the GitHub Actions workflow that's authorized to upload to PyPI. In addition, you may **optionally** provide the name of a [GitHub Actions environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment).

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-006.png" description="Configuring a Trusted Publisher in PyPI - screenshot showing the configuration form with fields for repository owner, repository name, workflow filename, and optional environment name" >}}

Publishing requirements will also differ depending on the platform, if you're using GitHub Actions the _blessed way_ is to use PyPA's (Python Package Authority) [pypi-publish](https://github.com/marketplace/actions/pypi-publish) action.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-007.png" description="Trusted Publishing to PyPI Github Action" >}}

This removes the need of managing a shared secret between PyPI and the CI/CD platform that provides usability and security benefits.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-009.png" description="Benefits of trusted publishing vs long-lived API tokens" >}}

This is not limited to PyPI and has seen adoption across the broader open-source ecosystem with other package registries also adding support. As of this writing, the following package registries (that I know of) support Trusted Publishing:

- Ruby's [RubyGems](https://guides.rubygems.org/trusted-publishing/)
- Rust's [Crates](https://blog.rust-lang.org/2025/07/11/crates-io-development-update-2025-07/)
- Dart's [Pub Package Registry](https://dart.dev/tools/pub/publishing)
- JavaScript's [npm](https://github.blog/changelog/2025-07-31-npm-trusted-publishing-with-oidc-is-generally-available/) and [JSR](https://jsr.io/docs/trust)

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-010.png" description="Trusted Publishing adoption in software registries" >}}

Trusted Publishing gives us _provenance_ i.e. verifiable metadata that details a software artifact's origin and unlocks _attestations_ -- which are cryptographically signed signatures that provide a verifiable link to an artifact or package's original source location (_provenance_).

Support for digital attestations in PyPI was added in [PEP 740](https://peps.python.org/pep-0740/). PyPI currently supports the following attestations:

- [SLSA Provenance](https://slsa.dev/spec/v1.0/provenance)
- [PyPI Publish](https://docs.pypi.org/attestations/publish/v1/)

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-013.png" description="PEP 740 Index support for digital attestations" >}}

[Sigstore](https://www.sigstore.dev/) is a set of tools that is used for digital signing, verification, and checking of package provenance.

It integrates with the Trusted Publishing process through the CI/CD platform, the latter sends the OIDC token together with a public key (from an ephemeral private + public key pair) to [Fulcio](https://docs.sigstore.dev/certificate_authority/overview/) which validates the token and sends back a signing certificate.

The signing certificate is then used to sign the attestations that are uploaded to [Rekor](https://docs.sigstore.dev/logging/overview/) which saves an immutable record to which the signatures can be verified against, the attestations are then published together with package to the target software registry.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-015.png" description="Trusted Publishing with Sigstore" >}}

PyPI supports the following platforms that can produce attestations:

- GitHub Actions
- GitLab CI/CD
- Google Cloud

If you're using GitHub Actions and PyPA's `pypi-publish` action (for version v1.11+), attestations are generated and uploaded automatically by default with no additional configuration necessary.

PyPI provides two ways of consuming attestations, one through a _Provenance_ section in the package UI and programatically through its [Integrity API](https://docs.pypi.org/api/integrity/).

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-017.png" description="PyPI Provenance UI" >}}

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-018.png" description="PyPI Integrity API" >}}

[npm](https://npmjs.com) also recently added support for generating provenance statements for packages published to its registry.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-019.png" description="npm support for generative provenance statements" >}}

In summary, these security features increase trust in the software supply chain by replacing long-lived API tokens with short-lived OIDC tokens from trusted providers and including cryptographically signed and verifiable signatures with packages when publishing to software registries.

The ideal state is getting to a point where verification of signatures is integrated with tooling like _uv_ and _pip_. Lastly, these features are not a _panacea_ -- they are important steps that we can use to improve overall software supply chain security.

{{< alt-text src="/images/annotated-slides/trusted-publishing-attestations/slide-025.png" description="Takeaways" >}}