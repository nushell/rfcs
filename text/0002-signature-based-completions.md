- Feature Name: signature-based-completions
- Start Date: 2020-07-25
- RFC PR: [nushell/rfcs#0002](https://github.com/nushell/rfcs/pull/2)
- Nushell Issue: [nushell/nushell#0000](https://github.com/nushell/nushell/issues/0000)

# Summary

Add a completion engine to nushell that can determine how to complete an item at the cursor by
parsing the line, extracting a signature, and using the information in that signature to determine
what kind of thing needs to be completed.

# Motivation

We currently use rustyline's completer to do our completions. This completer is unfortunately
limited, as it focuses on filenames only. We've tried to expand this with command names, but it
scatters completion code all over the codebase.

We already have incredibly rich information in signatures for commands, and intend to eventually use
this to describe external commands. By building our completions around signatures, we can take
advantage of highly contextual information to make nushell's completion system highly contextual,
powerful, and delightful to use.

# Guide-level explanation

Completion is the act of responding to a user request to find a list of recommendations that fit
some incomplete information that exists at their cursor. For example, if the user presses tab at the
end of this line:

    > ls dir/

all path entries under the subdirectory `dir` could be offered as suggestions. Similarly, for

    > ls dir/a

we'd expect all path entries under `dir` that start with `a`. It's also possible for the cursor to
be in the middle of a line, too. For

    > ls di<TAB>/a

Here we can show all directories whose name starts with "di". If the user chooses one of the
suggestions, completion would complete up to the `/`. Note, there are alternative approaches to how
completion could work here, but this RFC will assume the aforementioned, as it is perhaps the
simplest.

In all of the above examples, there's the concept of a "completion location". The completion
location scopes the possible completions. Some examples of such locations, emphasized by angle
brackets:

  - Command name: `<command> ...`
  - Flag name: `-<f>` and `--<flag>`
  - Flag value `-f <val>` and `--flag=<val>`
  - Positional arg: `command --flag -s <arg>`
  - Math operator: `= value <op> value`
  - Variable: `$<var>`, `$nu.<var>`
  - Column path: `thing.<path>`

All of these are found in the context of a pipeline, which is just a series of commands connected by
`|`. Pipelines can be found in many locations:

  - `pipeline ; pipeline`
  - `where { pipeline }`
  - `echo $(pipeline)`
  - `` `format string {{$(pipeline)}}` ``

The completion engine should understand the context it is in, and this is easiest if we defer most
of the work to the parser. The parser should be able to inform us about the location we're in. If
it's the position for a value, it should even describe the shape needed for the value. For example,
`where` takes one required positional argument, which has a "math" shape. The completion engine can
then look for a completer or set of completers that understand how to complete a math shape, and
collect the suggestions (if any).

Being able to write a completer that only cares about a specific shape is a powerful concept. It
takes away the need from writing completers that need to understand the entire line (although we can
still allow them to request the full line).

# Reference-level explanation

The major technical portion of this PR will be to overhaul the parser, so that it can better hint at
the shape under the cursor. The parser currently is two-phased: lite parse and full parse. The lite
parse does the basics of pulling out some basic components on a line, but knows nothing about
signatures. If it fails to parse, an error is returned. We need to extend this error with
information about what was successfully parsed, because the error may have occurred beyond the
cursor location. For example:

    > ls "some dir/

doesn't parse successfully, since the string is unterminated. That being said, the parser knows that
it's attempting to parse a string, so it should be able to convey that to the caller so that they
can take better action on this error state.

After lite parse, we need to follow up with a full parse to figure out which shape is under the
cursor. If this succeeds, we have a shape. If it errors, we provide no completion suggestions.

The second technical aspect of this completion engine is to make it store a registry of completers.
This registry would map completion locations and syntax shapes to a list of completers. For example,
we may have a registry like this:

  |---------------------|--------------|------------|
  | completion location | syntax shape | completers |
  |---------------------|--------------|------------|
  | command             | string       | filename, internal command, external command |
  | flag/arg value      | string       | filename |
  | flag/arg value      | path         | filename |
  | flag name           | string       | flag name |
  | variable            | string       | variable name |
  |---------------------|--------------|------------|

A completer would be described by the following trait:

```rust
trait Completer {
    fn complete(line: &str, span: Span, context: completion::Context) -> Vec<Suggestion>;
}
```

Concrete implementations are expected to take the given span for the line and provide some
completions based on the partial data. The completion context will allow completers access to the
nushell context, so they can discover variable names, environment variables, and so on. For example,
a simple completer may look like this:

```rust
impl Completer for MyCompleter {
    fn complete(line: &str, span: Span, context: completion::Context) -> Vec<Suggestion> {
        if span.string(line) == "hello" {
            vec![Suggestion::from("hello world")]
        } else {
            Vec::new()
        }
    }
}
```

Completers don't have to worry about replacement, they simply say what the span should be replaced
with. It's uncertain whether or not being able to replace a different span is necessary, so the
completion engine will start with the assumption that it is unnecessary.

The completion engine should deal with special needs of various shapes. For example, a string with
spaces will need to be quoted. Completers should not have to worry about this, and leave it up to
the engine. For example, when completing `hello` to `hello world`, like above, the completion engine
would need to add the appropriate quoting when replacing the span. Continuing with this example,

    git co hello

could be completed to

    git co "hello world"

The span will never include quoting characters and other syntactical elements that aren't considered
part of the value. Since completers have access to the full line, they can still consider those
elements, but ideally completers never have to concern themselves with syntax.

# Drawbacks

I see no reason to not do this. Beyond the effort, this approach offers far too many advantages to
completion than what we currently have, which doesn't feel idiomatic at all since it does not
tightly integrate with nushell's structured descriptions of commands.

# Rationale and alternatives

The only real alternative would be to continue doing the line-based approach we currently do. This
has a huge downside of reproducing a lot of the effort that has went into our parser in terms of
finding out the position of the cursor in terms of flags, positional arguments, etc.

# Prior art

Fish is a shell known for a great completion system. Fish keeps things simple by offering a
`complete` command that makes it very easy for someone to provide custom completions:

    complete -c myprog -s o -l output -a "yes no"

This provides completions for the command `myprog`. It's pieces are:

  - `-s o`, a short flag named "o";
  - `-l output`, a long flag named "output"; and
  - `-a "yes no"`, positional arguments having either a value of "yes" or "no".

This is similar to what nushell offers, minus the fact that nushell expresses these things through
signatures (note: we have yet to provide an external format for signatures).

Things gets a little more challenging once we get into subcommands:

    complete -c myprog -n "__fish_seen_subcommand_from set-option" -a "(myprog list-options)"

`-n` allows users to configure scripts to determine if the completion should be used. In this case,
fish has a builtin to determine if a subcommand has been used. In the above example, if the user has
typed `myprog set-option `, pressing tab will result in completion options being given based on
whatever `myprog list-options` outputs.

The beauty of a signature based approach in nushell is that subcommands are completely separate from
their root, so nushell will take care of figuring this out for the user. Each subcommand will have
its own signature.

# Unresolved questions

None that I can think of.

# Future possibilities

Completions are a huge part of shells, having the potential to provide a ton of productivity to the
user if sufficiently powerful. The improvements described here will provide a lot of completion
power to nushell users, but there's likely more room for improvement.

One thing not covered here is how users could specialize a shape. For example, `git checkout <TAB>`
would make sense to offer various refs - branch names, tags, remote refs, and so on - as completion
items. We can't include completions for every possible thing, so it would make sense for this to be
expressed externally somehow. At minimum, any implementation for this PR should keep this in mind.
The idea of a registry that maps locations/shapes to completers seems to be sufficient to satisfy
that requirement.
