# Nushell RFCs

[Nushell RFCs]: #nushell-rfcs

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Though some changes are substantial enough that we ask for these to be
put through a bit of a design process and produce a consensus among the
Nushell community.

The "RFC" (request for comments) process is intended to provide a consistent
and controlled path for new features to be added to Nushell, so that all
stakeholders can be confident about the direction the project is evolving in.


## Table of Contents
[Table of Contents]: #table-of-contents

  - [Opening](#nushell-rfcs)
  - [Table of Contents]
  - [When you need to follow this process]
  - [Before creating an RFC]
  - [What the process is]
  - [The RFC life-cycle]
  - [Reviewing RFCs]
  - [Implementing an RFC]
  - [RFC Postponement]
  - [License]
  - [Contributions]
  - [Acknowledgements]


## When you should follow this process
[When you should follow this process]: #when-you-should-follow-this-process

*Note: While we establish this RFC process, this says "should"; eventually
it should say "need to".*

You should follow this process if you intend to make substantial changes to
Nushell, or this RFC process itself. What constitutes a substantial change is
evolving based on community norms and varies depending on what part of the
ecosystem you are proposing to change, but may include the following:

  - Any semantic or syntactic change that is not a bugfix.
  - Removing existing features.
  - Changes to internal interfaces, like the plugin architecture.

Some changes do not require an RFC:

  - Rephrasing, reorganizing, refactoring, or otherwise "changing shape does
    not change meaning".
  - Additions that strictly improve objective, numerical quality criteria
    (warning removal, speedup, better platform coverage, more parallelism,
    better handle errors, etc.)
  - Additions only likely to be _noticed by_ other developers of Nushell,
    invisible to users of Nushell.

If you submit a pull request to implement a new feature without going through
the RFC process, it may be closed with a polite request to submit an RFC first.


## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

It is generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the RFC may be desirable; having a consistent
impact on the project requires concerted effort toward consensus-building.

The most common preparations for writing and submitting an RFC include talking
the idea over on our [official Discord server], in #design-discussion. You could also file issues on this repo for discussion.

As a rule of thumb, receiving encouraging feedback from long-standing project
developers is a good indication that the RFC is worth pursuing.


## What the process is
[What the process is]: #what-the-process-is

In short, to get a major feature added to Nushell, one must (for now: should)
first get the RFC merged into the RFC repository as a markdown file. At that
point the RFC is "active" and may be implemented with the goal of inclusion
into Nushell.

  - Fork the RFC repo [RFC repository]
  - Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is
    descriptive. Leave the `0000` for now).
  - Fill in the RFC. Put care into the details: RFCs that do not present
    convincing motivation, demonstrate lack of understanding of the design's
    impact, or are disingenuous about the drawbacks or alternatives tend to
    be poorly-received.
  - Submit a pull request. As a pull request the RFC will receive design
    feedback from the larger community, and the author should be prepared to
    revise it in response.
  - Now that your RFC has an open pull request, use the issue number of the PR
    to update your `0000-` prefix to that number.
  - Where applicable, each pull request will be triaged by the team in a
    future meeting and assigned to a member of the team.
  - Build consensus and integrate feedback. RFCs that have broad support are
    much more likely to make progress than those that don't receive any
    comments. Feel free to reach out to the RFC assignee in particular to get
    help identifying stakeholders and obstacles.
  - The team will discuss the RFC pull request, as much as possible in the
    comment thread of the pull request itself. Other discussion (e.g. on
    Discord) will be summarized on the pull request comment thread.
  - RFCs rarely go through this process unchanged, especially as alternatives
    and drawbacks are shown. You can make edits, big and small, to the RFC to
    clarify or change the design, but make changes as new commits to the pull
    request, and leave a comment on the pull request explaining your changes.
    **Specifically, do not squash or rebase commits after they are visible on the
    pull request.**
  - When the RFC is in a state that justifies community attention, it should be
    advertised widely, e.g. in [This Week in Nushell](https://www.notion.so/This-Week-in-Nu-ed589e19f1be47fa9ef562c1757e7ba1).
    This way all stakeholders have a chance to lodge any final objections
    before a decision is reached.
  - At some point after advertising the RFC was advertised publicly, a member
    of the team will propose a decision for the RFC (merge, close, or postpone).
    Another team member must sign off on that decision, and can then take the
    appropriate action. When closing or postponing, the reasons need to be well
    documented in the PR.

## The RFC life-cycle
[The RFC life-cycle]: #the-rfc-life-cycle

Once an RFC becomes "active" then authors may implement it and submit the
feature as a pull request to the Nushell repo. Being "active" is not a rubber
stamp, and in particular still does not mean the feature will ultimately be
merged; it does mean that in principle all the major stakeholders have agreed
to the feature and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is "active"
implies nothing about what priority is assigned to its implementation, nor does
it imply anything about whether a Nushell developer has been assigned the task of
implementing the feature. While it is not *necessary* that the author of the
RFC also write the implementation, it is by far the most effective way to see
an RFC through to completion: authors should not expect that other project
developers will take on responsibility for implementing their accepted feature.

Modifications to "active" RFCs can be done in follow-up pull requests. We
strive to write each RFC in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect every
merged RFC to actually reflect what the end result will be at the time of the
next major release.

In general, once accepted, RFCs should not be substantially changed. Only very
minor changes should be submitted as amendments. More substantial changes
should be new RFCs, with a note added to the original RFC. What exactly counts
as a "very minor change" is up to the team to decide.


## Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC pull request is up, the team may schedule meetings with the
author and/or relevant stakeholders to discuss the issues in greater detail,
and in some cases the topic may be discussed at a team meeting. In either
case a summary from the meeting will be posted back to the RFC pull request.

The team makes final decisions about RFCs after the benefits and drawbacks
are well understood. These decisions can be made at any time, but the team
will regularly issue decisions. When a decision is made, the RFC pull request
will either be merged or closed. In either case, if the reasoning is not clear
from the discussion in thread, the team will add a comment describing the
rationale for the decision.


## Implementing an RFC
[Implementing an RFC]: #implementing-an-rfc

Some accepted RFCs represent vital features that need to be implemented right
away. Other accepted RFCs can represent features that can wait until some
arbitrary developer feels like doing the work. Every accepted RFC has an
associated issue tracking its implementation in the Nushell repository; thus that
associated issue can be assigned a priority via the triage process that the
team uses for all issues in the Nushell repository.

The author of an RFC is not obligated to implement it. Of course, the RFC
author (like any other developer) is welcome to post an implementation for
review after the RFC has been accepted.

If you are interested in working on the implementation for an "active" RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).


## RFC Postponement
[RFC Postponement]: #rfc-postponement

Some RFC pull requests are tagged with the "postponed" label when they are
closed (as part of the rejection process). An RFC closed with "postponed" is
marked as such because we want neither to think about evaluating the proposal
nor about implementing the described feature until some time in the future, and
we believe that we can afford to wait until then to do so. This could be used
to postpone features until after 1.0. Postponed pull requests may be re-opened
when the time is right. We don't have any formal process for that.

Usually an RFC pull request marked as "postponed" has already passed an
informal first round of evaluation, namely the round of "do we think we would
ever possibly consider making this change, as outlined in the RFC pull request,
or some semi-obvious variation of it." (When the answer to the latter question
is "no", then the appropriate response is to close the RFC, not postpone it.)


## License
[License]: #license

This repository is licensed under the MIT license ([LICENSE](LICENSE) or http://opensource.org/licenses/MIT)


### Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.


## Acknowledgements

This RFC process started as partial copy of [Rust's RFCs repo](https://github.com/rust-lang/rfcs).
In this case, their prior art helped us kick-start the RFC process, skipping many of the process work itself.

Since Nushell at this time has no sub-teams, that concept was removed or
replaced by "team" (implying commiters to the main nushell repo on GitHub).

The FCP stage was removed, to adjust the process towards Nushell's current needs.

We're considering adding a [stage process similar to TC39](https://tc39.es/process-document/). For now we're starting without explicit stages.


[official Discord server]: https://discord.gg/NtAbbGn
[RFC repository]: https://github.com/nushell/rfcs
