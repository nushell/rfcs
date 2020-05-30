- Feature Name: establish-rfc-process
- Start Date: 2020-05-30
- RFC PR: [nushell/rfcs#0001](https://github.com/nushell/rfcs/pull/1)
- Nushell Issue: [nushell/nushell#0000](https://github.com/nushell/nushell/issues/0000)

# Summary
[summary]: #summary

Establish an RFC process, copying Rust's process.

# Motivation
[motivation]: #motivation

Adding, extending and changing commands and their sytanx, flags and other behaviour is currently happening ad-hoc, via GitHub issues and PRs, sometimes with previous discussion on Discord. To establish a process for proposing, reviewing, accepting and implementing, or rejecting such changes, we want to copy what already works well for Rust.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Since this meta-RFC adds `0000-template.md` and `README.md` itself, all details can be found in those two files. Only changes to the RFC process should be allowed exceptions like this.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

See above.

# Drawbacks
[drawbacks]: #drawbacks

While we're pre-1.0, being able to quickly change Nushell, without much discussion, might be more desirable than this formal process.

To soften the blow, for now the instruction is "should use this process", not "needs to use this process". Closer or post-1.0 that language should be changed to "needs to".

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

I've only looked at how Rust is doing it, so there might be better options.

The impact of not doing this could be high churn and overhead of designing only in code, with unclear pathways for discussing designs upfront.

# Prior art
[prior-art]: #prior-art

This started as a copy of Rust's RFC process, as indicated in the (additional) Achknowledgements section in the README. Their [README file](https://github.com/rust-lang/rfcs/blob/master/README.md) alone has had 29 contributiors at the time of writing this, and had plenty of time to evolve since its inception in March 2014.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

The concept of the "team" is currently not well defined (to my knowledge). It doesn't exist in Rust, which instead has sub-teams (for compiler, language and libraries). Actually defining "team" for Nushell, for example as all commiters to the nushell/nushell repostiroy, would be a potential solution.

If that is undesirable, some aspects of the RFC process have to be changed (search for all mentions of "team" in the README and adjust them). For example accepting RFCs might have to be done by a minimum number of commiters, instead of "all team members".

# Future possibilities
[future-possibilities]: #future-possibilities

Eventually it might be a good idea for Nushell to also establish sub-teams. In that case, it would be helpful to look again how Rust specifies sub-team specific rules and processes.