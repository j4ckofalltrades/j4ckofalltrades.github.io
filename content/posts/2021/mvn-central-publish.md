---
title: "Publishing to Maven Central"
description: "Publishing to Maven Central"
date: 2021-07-25T15:38:23+08:00
draft: false
categories:
- dev
tags:
- gradle
- kotlin
- ktor
- ossrh
series:
- "Package registry publish guides"
featuredImage: "https://res.cloudinary.com/j4ckofalltrades/image/upload/v1633760759/blog/publish-guides/mvn-publish_jfglsn.png"
featuredImageAltText: "Sample package in Maven Central"
---

_Update (2024-03-08): Bump the `checkout` and `setup-java` action versions, and set target Java version to 17._ 

This guide walks you through the necessary steps to upload your package to the
[Maven Central](https://repo.maven.apache.org/maven2) repository (and optionally
to GitHub Packages registry) with some recommendations along the way.

{{< toc >}}

## Setup OSSRH Repository

- Sign up for a [new account](https://issues.sonatype.org/secure/Signup!default.jspa)
- Create a [new project ticket](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)
- Claim your namespace by setting the `groupId` for your artifact

  If you're using a free code hosting service like GitHub or Bitbucket:

  `groupId: io.github.j4ckofalltrades`

  Alternatively if you want to use a custom domain, the group id should be:

  `groupId: com.your-custom-domain`

  You will also need to prove ownership of the public repository or the custom
  domain. For the former you will be asked to create an empty repository using
  the ticket number for your created OSSRH ticket e.g. `OSSRH-73148`; for the
  latter you'll need to add a `TXT` record linked to the OSSRH ticket that was
  created to register your `groupId`.

## Configure GPG/PGP key

This will be used for signing your artifacts, at it is a requirement for
publishing them to the Central Repository.

You'll need to generate a key pair and distribute it to a known key server
(so that others can validate it).

The following examples show how to do this with [GPG](https://gnupg.org); you
can [download](https://gnupg.org) or install it using your package manager if
it is not already available in your machine.

To generate a key pair, run:

`$ gpg --gen-key`

Fill in the details you are prompted for e.g. name, email, time of validity, and
passphrase for the generated key. **This passphrase and your private key are all
that is needed to sign artifacts with your signature.**

Once the key pair is generated, you'll need to distribute your public key to a
known key server i.e. `keyserver.ubuntu.com`. In order to do so, run the
following commands:

```sh
# list gpg keys
$ gpg --list-keys

# the line starting with **pub** shows your public key details
# copy the keyid and run the following command
$ gpg --keyserver keyserver.ubuntu.com --send-keys <your_key_here>
```

To check that your key was successfully uploaded, you can search for it at
https://keyserver.ubuntu.com using your public key (and prepending '0x' to it).

Other supported key servers are `keys.openpgp.org`, and `pgp.mit.edu`.

## Configure build script

In order to publish your artifact to the Central Repository, it will need to
meet the following requirements:

- Project coordinates
- Javadoc and sources jar
- POM metadata (includes project name and description, url, SCM info, license)
- Sign artifacts with GPG/PGP

The configuration will vary slightly depending on the build tool you are using,
below is an abridged sample configuration `build.gradle.kts` file (Kotlin DSL)
used with `gradle`:

```kotlin
// build.gradle.kts

// project coordinates
group = "io.github.j4ckofalltrades"
version = "1.0.0"

// generate KDoc and Javadoc using Dokka
tasks.withType<DokkaTask>().configureEach {
    dokkaSourceSets {
        named("main") {
            outputDirectory.set(file("docs"))
            moduleName.set("steam-webapi")
            includes.from("Module.md")
            displayName.set("JVM")
            platform.set(org.jetbrains.dokka.Platform.jvm)
            sourceLink {
                localDirectory.set(file("src/main/kotlin"))
                remoteUrl.set(
                    URL(
                        "https://github.com/j4ckofalltrades/steam-webapi-kt" +
                            "/tree/main/lib/src/main/kotlin"
                    )
                )
                remoteLineSuffix.set("#L")
            }
        }
    }
}

// javadoc and sources jar
tasks {
    val sourcesJar by registering(Jar::class) {
        dependsOn(JavaPlugin.CLASSES_TASK_NAME)
        archiveClassifier.set("sources")
        archiveBaseName.set("steam-webapi-kt")
        from(sourceSets["main"].allSource)
    }

    val javadocJar by registering(Jar::class) {
        dependsOn("dokkaJavadoc")
        archiveClassifier.set("javadoc")
        archiveBaseName.set("steam-webapi-kt")
        from("docs")
    }

    artifacts {
        archives(sourcesJar)
        archives(javadocJar)
        archives(jar)
    }
}

publishing {
    publications {
        create<MavenPublication>("mavenJava") {
            groupId = project.group.toString()
            artifactId = rootProject.name
            version = project.version.toString()

            from(components["java"])
            artifact(tasks["sourcesJar"])
            artifact(tasks["javadocJar"])

            // POM metadata
            pom {
                name.set("steam-webapi-kt")
                description.set("Steam WebAPI wrapper in Kotlin and Ktor")
                url.set("https://github.com/j4ckofalltrades/steam-webapi-kt")
                licenses {
                    license {
                        name.set("MIT")
                        url.set("https://opensource.org/licenses/MIT")
                    }
                }
                developers {
                    developer {
                        id.set("j4ckofalltrades")
                        name.set("Jordan Duabe")
                        email.set("me@jduabe.dev")
                    }
                }
                scm {
                    connection.set("scm:git:git://github.com/j4ckofalltrades/steam-webapi-kt.git")
                    developerConnection.set("scm:git:ssh://github.com/j4ckofalltrades/steam-webapi-kt.git")
                    url.set("https://github.com/j4ckofalltrades/steam-webapi-kt")
                }
            }
        }
    }
    repositories {
        // Maven Central
        maven {
            name = "OSSRH"
            url = uri("https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/")
            credentials {
                username = System.getenv("OSSRH_USERNAME")
                password = System.getenv("OSSRH_PASSWORD")
            }
        }
        // GitHub Packages
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/j4ckofalltrades/steam-webapi-kt")
            credentials {
                username = System.getenv("GITHUB_ACTOR")
                password = System.getenv("GITHUB_TOKEN")
            }
        }
    }
}

// sign artifact
signing {
    setRequired({
        gradle.taskGraph.hasTask("publishMavenJavaToOSSRHRepository")
    })

    val signingKey = System.getenv("SIGNING_KEY")
    val signingPassword = System.getenv("SIGNING_PASSWORD")
    useInMemoryPgpKeys(signingKey, signingPassword)
    sign(publishing.publications["mavenJava"])
}
```

## Publishing artifacts

The configuration above uses the [in-memory ascii-armored keys](
https://docs.gradle.org/current/userguide/signing_plugin.html#sec:in-memory-keys])
approach for signing artifacts. Check out the docs for the [Gradle Signing
Plugin](https://docs.gradle.org/current/userguide/signing_plugin.html) to find
out what best suits your use case.

Once everything is configured, publishing the artifacts is as simple as running:

`$ gradle publish`

Note that this will publish your artifacts to **all** target repositories, in
order to publish to a specific repository you need to specify the repo _name_.
Using the above configuration as an example:

```sh
# publish to Maven Central
$ gradle publishMavenJavaPublicationToOSSRHRepository

# publish to GitHub Packages
$ gradle publishMavenJavaPublicationToGitHubPackagesToRepository
```

## Releasing artifacts

Login to the [Nexus staging repository](https://s01.oss.sonatype.org/#stagingRepositories),
where you should be able to see your `groupId` listed. In order to _sync_ the
artifacts to the Central Repository you'll need to click on the `Close` button
which will close said staging repository. 

The close operation will run a series of checks to ensure that the uploaded
artifacts meet the [requirements](#configure-build-script).

If the release was successful, you should be able to see your artifacts in the
Central Repository at https://repo1.maven.org/maven2/ typically within ~30
minutes. Updates to https://search.maven.org can take up to a couple of hours.

## Bonus: Automation with GitHub Actions

You'll probably want to automate this process as part of your CI/CD pipeline.
Here's how to set it up using GitHub Actions.

You'll need to add your OSSRH credentials, and signing key details as secrets
to your repository, you can find this under Settings > Secrets.

```yaml
name: Publish to OSSRH and GitHub Package Registry

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - run: ./gradlew publish
        env:
          GITHUB_ACTOR: ${{ github.actor }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OSSRH_USERNAME: ${{ secrets.OSSRH_USERNAME }}
          OSSRH_PASSWORD: ${{ secrets.OSSRH_PASSWORD }}
          SIGNING_KEY: ${{ secrets.SIGNING_KEY }}
          SIGNING_PASSWORD: ${{ secrets.SIGNING_PASSWORD }}
```

The configuration samples used here can also be checked out in full at this [GitHub repository](https://github.com/j4ckofalltrades/steam-webapi-kt).

That's it, time to get publishing.
