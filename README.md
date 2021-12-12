# yogslaw

## What is this?

This is a thought experiment for a potentially different "default approach" to software licensing of my Open Source Projects. I'm not proposing you do this: but rather thinking out how I could do this.

Some of my projects have reached a level of use in the Rust ecosystem, including some commercial usage. To date, I've not received (or requested!) funds for these libraries, but also have never been hassled by my users for support or unreasonable requests.

## Background

This was originally written in the aftermath of the `log4j` vulnerability, and the discussion that resulted from the lack of financial support for the authors and maintainers.

The name comes from writer Jim Macdonald's "Yog's Law", who reminded writers that **"Money flows toward the writer."**. This was suggested (indirectly) by [Naomi Wu](https://twitter.com/RealSexyCyborg/status/1470044032290402307) on twitter.

## What are my goals?

In my open source projects, I aim to:

* Share my projects with people to see, learn from, be able to debug.
* Make clear what other people's expectations should be (regarding support, change requests, security updates, maintenance) for code I write: namely none, unless otherwise agreed.
* Allow generally unrestricted use for non-commercial users
* Set the expectation that **if people derive monetary benefit from the code I write: I expect to be paid for it.**
* Provide an easy, low-hassle way to pay for additional support and increased maintenance expectations

## What are **NOT** my goals?

In my open source projects, I don't generally care about the following:

* I am not interested in making my code "as widely usable/used as possible", at least not with this approach.
* I am not interested in worrying about making code that is "incompatible with commercial usage", at least at the "free" tier
* I don't particularly care if my approach is compatible with existing OSI or FSF paradigms

## How do I (think) it would work?

I would like to be able to have two to three levels of licensing for my projects:

* **Free for Non-Commercial Usage** - basically if you are using my code as a student, hobbyist, or for an initial prototype: just use it. No warranty or expectations of support should exist.
* **Licensed, no warranty** - If someone is a passive user of my code for a commercial project, but does not need customization or support, I'd like them to pay a smaller, one time fee in order to support my work.
* **Licensed, with warranty** - If someone makes heavy usage of my work for a commercial project, and expects customization, support, security updates and response, etc., I'd like them to pay a recurring fee in order to support my work.

In the **Free for Non-Commercial Usage** case, the software would be released under a specifically non-commercial license, such as one of the [Creative Commons Non-Commercial](https://creativecommons.org/licenses/by-nc-sa/4.0/) licenses, or the [Prosperity Public License](https://prosperitylicense.com/). The source would be widely available (likely published on GitHub and package managers like crates.io), but with no expectations of support or warranty.

In the **Licensed, no warranty** case, the expectations would be similar to the **Free for Non-Commercial Usage** case, however the user would be granted a limited, non-transferrable, proprietary license for the software in question based on a one-time purchase. Ideally: This would be achieveable with little human interaction, such as a web store (incl. Stripe, Github Sponsors, Patreon) with a flat purchasing rate.

In the **Licensed, with warranty** case, the user could be expected to make feature requests, receive support, and have timely security updates. The user would be expected to make a recurring yearly payment for this level.

## How do I think I would do this?

I spend nearly all of my time working in Rust, so here is a "strawman" implementation of how I think this could work in that environment. This post assumes you are familiar with Rust, Cargo, and the Crates.io package registry.

In particular, any DRM scheme I can imagine would be easily defeated by determined individuals, so this approach aims to be as unobstructive as possible to both commercial and non-commercial usages.

### Summary of process

1. When authoring a library or application, the author lists the package under a non-commerical license, such as the Prosperity License.
2. The Author also includes a unique-to-the-project asymmetric public key (e.g. ed25519) used to sign licenses of the project
3. The author includes a `build.rs` script that validates the license selected by the user. This would be invoked at compile time.
4. When depending on the library in question, the user would be expected to provide a metadata file in their project, such as `license-$libraryname.toml`. This file would contain one of the following kinds of content:
    1. A line stating that the project is for non-commercial usage, e.g. "no license"
    2. Contents that include the contact email, project name, a randomly generated license ID (and/or one-time pad), as well as a signature created from the library's private key of the above data
    3. Contents that include the contact email, project name, license ID (and/or one-time pad), AND an expiration date, as well as a signature created from the library's private key of the above data
5. If the user failed to provide the above metadata, the `build.rs` script would cause the build to fail.

An example of a non-commercial license file:

```toml
# license-example-lib.toml

usage = "non-commercial"
```

An example of a commercial license file:

```toml
# license-example-lib.toml

contact-email = "user@example.com"
project = "good-example"
usage = "licensed"

# uuid
license-id = "5a001d53-b6ab-4f64-996a-f710dcdbc637"

# 64-byte ed25519 signature of the above metadata, as hex
license-signature = "feedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafe"
```

An example of a warrantied license file:

```toml
# license-example-lib.toml

contact-email = "user@example.com"
project = "good-example"
usage = "licensed"

# uuid
license-id = "5a001d53-b6ab-4f64-996a-f710dcdbc637"

# datetime, as rfc3339/iso8601 format
expiration = "2022-04-20T23:20:50.52Z"

# 64-byte ed25519 signature of the above metadata, as hex
license-signature = "feedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafefeedcafe"
```

### Technical notes

In this approach, the license key would be tied to a specific project and contact person (could also be a blanket `license@example.com` address).

By checking the signature at build time, the `build.rs` script would not require internet access at the time of build, only the library public key (contained in the library itself), and the signature (contained in the project directory, which is accessible from a `build.rs` script).

This means that internet access is only required when purchasing a license, which is either a one-time task (for the second tier of support), or a once-per-year task (for the third tier of support).

It is *absolutely* possible to subvert this approach trivially, including:

* Disabling the `build.rs` script
* Changing the public key in the crate metadata

If someone wants to get around the restrictions of the license, I likely cannot stop them, at least not indefinitely. This is intended to be a speed bump for well-meaning and law-abiding commercial users.

### Financial notes

Unfortunately, "receiving money from people potentially in other countries" is often the most challenging part of the equation, especially for individuals not set up as a freelancer or as part of a company.

The technical side of issuing a license is likely relatively trivial: consisting of a bot that generates and signs tokens after a transaction.

For indivuals interested in this license approach but NOT set up to receive funds, they could instead request a donation to a charity of their choice, or to a fellow OSS maintainer that is. Automation of this is likely more difficult, but could probably be implemented in a lo-fi "send me a screenshot of your donation" method, and manually issuing licenses using a CLI script.

## Where to go from here?

I dunno. This was on my brain, and I had to write it out to get it off of my head.

If there is a positive response to this, I'll likely write a proof of concept implementation.

I'd be interested in your feedback in the form of PRs or Issues on this repo, as well as sharing it with people you think might be interested.

Feel free to [send me an email](james@onevariable.com) if you'd like to chat privately about it.
