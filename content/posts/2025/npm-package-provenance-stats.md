---
title: npm package provenance stats
date: 2025-09-18T23:19:09+08:00
description: A static site that tracks which of the top 500 most-downloaded packages on npm have attestations
draft: false
categories:
tags:
  - npm
  - trusted-publishing
  - attestations
series:
images:
featured_image:
mastodonPostID:
---
While writing the slides for my [Trusted Publishing & Digital Attestations talk](/posts/2025/trusted-publishing-attestations), I was looking for a resource similar to [Are we PEP 740 yet?](https://trailofbits.github.io/are-we-pep740-yet/) -- which shows the top 360 most-downloaded packages on [PyPI](https://pypi.org) and which ones have been uploaded with attestations.

![PEP 740](/images/annotated-slides/trusted-publishing-attestations/slide-013.png)

I couldn't find one, so obviously the only logical next step is to build my [own](https://jduabe.dev/npm-package-provenance-stats/). Check out the source code at https://github.com/j4ckofalltrades/npm-package-provenance-stats.

Using the _Are we PEP 740 yet?_ site as the reference implementation, I needed a way to fetch the _n_ top most-downloaded packages from npm, extract the relevant details for each package from the npmjs registry API, write the results to a file and display the data in a static site, and configure all this to run on a scheduled interval i.e., every _x_ hours.

The [download-counts](https://npmjs.com/package/download-counts) package, updated with a new version twice per month, just exports a single giant static object whose keys are package names and whose values are monthly download counts.

Using this package we can easily get the top 500 most-downloaded packages (by monthly download count) from npm.

```js
const downloadCounts = require('download-counts');

/** Get top 500 most downloaded packages */
const mostDownloadedPackages = Object.entries(downloadCounts)
	.sort(([_, cnt1], [__, cnt2]) => cnt2 - cnt1)
	.slice(0, 500)
	.map(([name, _]) => name);
```

Given the package names, the next step is parsing the relevant details from the npm registry at https://registry.npmjs.org.

The endpoint to get details for a specific package is `https://registry.npmjs.org/<package>`. The following is an abridged version of the JSON data for the `@j4ckofalltrades/steam-webapi-ts` package.

```json
{
  "name": "@j4ckofalltrades/steam-webapi-ts",
  "dist-tags": {
    "latest": "1.2.2"
  },
  "versions": {
    "1.2.2": {
      "name": "@j4ckofalltrades/steam-webapi-ts",
      "description": "Isomorphic Steam WebAPI wrapper in TypeScript",
      "version": "1.2.2",
      "repository": {
        "type": "git",
        "url": "git+https://github.com/j4ckofalltrades/steam-webapi-ts.git"
      },
      "dist": {
        "attestations": {
          "url": "https://registry.npmjs.org/-/npm/v1/attestations/@j4ckofalltrades%2fsteam-webapi-ts@1.2.2",
          "provenance": {
            "predicateType": "https://slsa.dev/provenance/v1"
          }
        }
      },
      "_npmUser": {
        "name": "GitHub Actions",
        "email": "npm-oidc-no-reply@github.com",
        "trustedPublisher": {
          "id": "github",
          "oidcConfigId": "oidc:4bcac187-e959-43d6-a0b2-995f804750a1"
        }
      }
    }
  },
  "time": {
    "1.2.2": "2025-08-02T16:03:41.470Z"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/j4ckofalltrades/steam-webapi-ts.git"
  }
}
```

The data can then be structured and pared down to only the relevant details and written out to a JSON file.

```javascript
const version = data['dist-tags']?.latest;
const versionData = data.versions?.[version];

const data = {
	package: data.name,
    version: data['dist-tags']?.latest,
    lastUploaded: data.time?.[version] || "",
    repositoryUrl: data.repository.url,
	// Non-empty if the package was published with attestations
    attestationsUrl: versionData.dist?.attestations?.url || "",
	// Non-empty if the package was published with a Trusted Publisher
    trustedPublisher: versionData._npmUser?.trustedPublisher?.id || "",
};
```

The static site basically just reads the contents of the resulting JSON file and displays them in a searchable columnar layout. The packages are color coded and have different icons to differentiate package status.

- Green packages with a 🔏 have attestations for their latest release
- Grey packages with a ⏰ come from a supported CI/CD provider but were uploaded before attestations were available
- Yellow packages with a ➖ come from a supported CI/CD provider but have no attestations (yet!)
- Pink packages with a 🚫 come from an unsupported CI/CD provider
- Packages with a 📄 use Trusted Publishing instead of long-lived API tokens

![npm package provenance stats](/images/annotated-slides/trusted-publishing-attestations/slide-020.png)

The last step is just a matter of configuring a scheduled workflow that fetches the package names and attestations, then deploys the static site.

```yaml
---
name: Deploy site

on:
  push:
    branches:
      - main
  # Scheduled tasks only run on the main branch.
  schedule:
    - cron: '0 */4 * * *'  # Every 4 hours.
  workflow_dispatch:

permissions: {}

jobs:
  deploy:
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - name: Checkout repository
        uses: actions/checkout@08c6903cd8c0fde910a37f88322edcfb5dd907a8 # v5.0.0

      - name: Setup Node.js
        uses: actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.4.0
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Fetch attestations data
        run: node fetch-attestations.js

      - name: Prepare deployment files
        run: |
          mkdir site
          cp index.html site/
          cp results.json site/

      - name: Setup Pages
        uses: actions/configure-pages@983d7736d9b0ae728b81ab479565c72886d7745b # v5.0.0

      - name: Upload artifact
        uses: actions/upload-pages-artifact@7b1f4a764d45c48632c6b24a0339c27f5614fb0b # v4.0.0
        with:
          path: 'site'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@d6db90164ac5ed86f2b6aed7e0febac5b3c0c03e # v4.0.5
```
