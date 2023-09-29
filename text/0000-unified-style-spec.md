- Feature Name: `unified_style_spec`
- Start Date: 2023-09-29
- RFC PR: [nushell/rfcs#0000](https://github.com/nushell/rfcs/pull/0000)
- Nushell Issue: [nushell/nushell#0000](https://github.com/nushell/nushell/issues/0000)

# Summary
[summary]: #summary

We do not have a unified style for the Nu language like Rust has.
This RFC is to see what the community thinks of adding one, NOT what that spec would look like. That development would be done on it's own.

# Motivation
[motivation]: #motivation

While discussing major work on the Nu language formatter, AucaCoyan mentioned that there is no standardization of style.
Creating a robust formatter requires an equally robust style specification.
Making sure everyone is happy with a new standard existence is important.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

This RFC is not about the spec itself just about the idea of creating one.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Same as above.

# Drawbacks
[drawbacks]: #drawbacks

None that I can think of. Please let me know any that may pose issues in the future. 

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Without a stalwart specification, different projects across nushell will tend to have subtle (or even major) differences that decrease the readability and maintainability of Nu code.

# Prior art
[prior-art]: #prior-art

Rust already has a style guide, crafted by the community. There doesn't seem to be any downsides because the formatter allows you to customize what rules to follow.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What the specification would look like.
- How the style specification would interact with an eventual language specification.

# Future possibilities
[future-possibilities]: #future-possibilities
