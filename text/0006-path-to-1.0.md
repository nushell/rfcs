- Feature Name: path_to_1_0
- Start Date: 2021-03-30
- RFC PR: [nushell/rfcs#0006](https://github.com/nushell/rfcs/pull/0006)

# Summary
[summary]: #summary

This document is the proposal to help direct Nushell towards a
successful 1.0 release. It's loosely divided into sections based on
what needs to happen first, followed by what needs to happen next, and
so on.

# Motivation
[motivation]: #motivation

As both a language and a tool, Nushell would benefit both by hitting 1.0 and by reaching 1.0 at the highest possible quality in a reasonable amount of time.

This allows potential users who have waited for Nushell to reach a stable release to start using Nushell. It also allows developers to create Nushell scripts with confidence that these scripts will continue working in the future.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

We'd like to follow in the steps of Rust. As it came time to pick what features would be part of Rust's 1.0 release, the Rust leadership put features into two buckets: those not ready for 1.0 and those ready for 1.0. All features started in the "not ready for 1.0" bucket and had to prove they were stable, supportable, and worked together with the other features that had made it into 1.0.

Nushell can follow the same path. All Nushell features would start in the "not ready to ship in 1.0" bucket and each one would have to prove that it was ready for 1.0.

The lists below show features we hope will make it for 1.0. It may also be possible that some features become ready that we are not currently anticipating, while other features we hoped would ship do no ultimately meet the bar.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## 0.60 milestone

The most important things to get designed and implemented first are
the changes which will be the most disruptive to current users of
Nushell. We're aware that any change at this point needs to be
well-considered and also needs to land sooner rather than later. The
longer we wait, the harder it will be for users of Nushell to absorb
the change into their growing collection of scripts and shortcuts.

### Elevator pitch

Many of us appreciate different aspects of Nushell, but one aspect we
can improve is how to share that succinctly with the rest of the
world. Nushell represents some interesting choices that help improve
developer workflows, make data processing easier, and also offers
dozens of opportunities for more easily working with your system.

We'd like to develop a way to describe the above, and more, in a way
that's easy for potential users of Nushell to grasp why Nushell is
important and how it could fit their needs.

### Solidifying core syntax

There are a set of syntax that are used often and that will cause
breakage broadly if changed. Solidifying this syntax also allows us to
more securely build on top of it for 1.0 features.

Here are some of the syntax points that would be good to solidify:

**Variables and expressions**

- [x] Variables currently use `$` sometimes and not others. We should
  settle on when and why.
- [x] Expressions invocations currently use `$()`. We should explore `()`
  as a possible simpler syntax.
- [ ] Clarify difference between `each`, `if`, and `where`. The `$it`
  variable feels a bit special and also unpredictable. We need a
  design that makes it easy to predict when it will be available and
  what it will do.
- [ ] A possible new `$in` variable, or related, to represent the input to
  a command
- [ ] A yield or `$out` style variable to allow multiple lines to
  participate in output in a command

**Additional syntax**

- [ ] Doc comments: what are they and where can we use them
- [x] Blocks have parameters but there is no syntax for them, yet

### Core data model

- [ ] Streams as values. Will it be possible? If so, how will it work?
  * eg) `ls | echo %1`
  * Related to the above: xargs-like functionality

### Types

- [ ] Future-proof type syntax
- [ ] Enable building support for external command autocomplete

### Fixing outstanding bugs

- [ ] Remove all dynamic scoping. Ensure all scoping is lexical.

### Philosophy

- [ ] Are repl/script the same? Where do we differ? Related: align `;` to work the same as `\n`.
- [ ] Should we raise exceptions if there are holes in the data as you process it? When and how?
  
## 0.80 milestone

The key part of the 0.8 milestone is to **finish language-breaking
changes**. As part of this, we'd also like to finish semantic changes
to the core commands as well, as they make up an important part of the
Nushell language.

### Core commands

- [ ] Clean up `str`
- [ ] Change `echo` to actually echo to the screen
- [x] Use another function to mean "send value to pipeline"
- [ ] Explicit iteration expansion (eg, like what `echo 1..10` is today)
- [ ] Final list of internal commands

### Ergonomics

- [ ] Improve adding paths to the PATH
- [ ] Cache config
- [ ] Set baseline performance target
- [x] Shorthand column `get`
- [ ] Completions
- [ ] Script parameters

### Language concepts

- [ ] Imports and modules
- [ ] When do we iterate? Should `=` iterate?
- [ ] Can you operate on the incoming pipeline as a single value? If so, how?

### Configuration

- [ ] Solidify the file formats we'll use

## 1.0 milestone

### Future-proofing

- [x] Future-proof string/path interpolation
- [ ] Scope-down and future-proof plugin protocol

### Documentation

- [ ] Book 1.0
- [ ] Translations of documents

### Ergonomics

- [ ] Paging
- [ ] Per-session history and global history
- [ ] Polished error messages
- [x] Fix starship issues
- [ ] Better structured data matching (like grep over structured data)

### Final designs

- [ ] External->Internal final design
- [ ] Internal->External final design

### Shipping quality

- [ ] Sign binary releases
- [ ] Default install artifacts (what will Nushell include at 1.0?)
- [ ] Move unstable features behind feature flags
- [ ] Enable self-update capabilities?
- [ ] Ship wasm demo for 1.0


### Possible inclusion

- [ ] Future-proof ability to process stream in parallel


# Drawbacks
[drawbacks]: #drawbacks

- One drawback is that this will mean moving from exploration energy to shipping energy.
- A related drawback is that some features may not make it and will need to be removed to ensure a high quality 1.0 release.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- The main wiggle room is in what features we go after and what we do not. We may decide a different set of features are a better fit for 1.0 that those in this proposal. That new set would need clear rationale for why its endpoint would be stronger than this.

# Future possibilities
[future-possibilities]: #future-possibilities

The 1.0 release marks an important achievement for Nushell. After this point, scripts written against 1.0 will continue to work into the future. This stable language will allow people to create a growing library of scripts, share scripts with each other, and trust Nushell when doing production work.

There are a number of potential future features that we could enable after 1.0 ships: more varied uses for Shells, a robust system for managing background tasks, type-provider style completions to name a few.

We, as a team, take this commitment very seriously, and are looking forward to shipping a 1.0 we're proud of. If you'd like to help, please join us on the [discord](https://discord.gg/NtAbbGn). We'd be happy to have your input.
