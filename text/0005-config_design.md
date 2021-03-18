- Feature Name: Redesign of nushells config structure
- Start Date: (fill me in with today's date, 2021-03-18)
- RFC PR: [nushell/rfcs#0005](https://github.com/nushell/rfcs/pull/0005)

# Summary
[summary]: #summary

This RFC shall provide the basis for a structured discussion, how the nushell config could be improved.

# Motivation
[motivation]: #motivation
Important: This document is written as if, https://github.com/nushell/nushell/pull/3041 would already be accepted and merged. The PR is currently in review.


Nushell evolved over time and features got added. The current config format may need to be adapted to better represent nushells capabilities to new users.

The config structure is mostly well done, but some enhancements I can think of:
1. Every `[...]` table has a name without config, but there is `[color_config]` which I think should be renamed to `theme` (better consistency, better naming).
2. We could add a table for aliases. Example:
```toml
[aliases]
vim = "vscode" # The new kids on the block won't even notice
emacs = "echo Hit exit-meta-alt-control-shift at the same time to enter emacs" # Hehe does nothing :)
```
3. We could give $PATH ($nu.path) his own section. Example:
```toml
[path]
defaults = ["/usr/bin", "/home/sweet/home"]
vpn_client = "/opt/vpn_client/bin"
cargo = "/home/leo/.cargo"
```
4. Some settings are not put inside a table `[...]`. Those could at least be grouped under `[general]`.
5. With `.nu-env` and the "global" config under `$HOME/.config/nu/config.toml` being merged, we can think of `autoenv` as a local config (a config local to a directory). Therefore we could rename `.nu-env` to `.nu/config.toml`. (We may need to adapt the `autoenv` commands accordingly)
    - Rename `autoenv trust` to `package trust` (see 9)
    - Rename `autoenv untrust` to `package untrust` (see 9)
6. We have `startup` and `on_exit`. We could rename `startup` to `on_enter` (Better naming, better consistency (`enter`/`exit` command)).
7. `startup` for most people I guess (?), contains mostly `alias`'es and `def`'s. Aliases are short and can be put under one table `[aliases]`, `def`'s might be long and best put into individual files. We could add a directory `defs` for `def`'s.
Example: File `$HOME/.config/nu/defs/cd.nu`
```nu
def cd [path] {ls $path; ^cd $path}
```
8. `textview`, `line-editor` and `theme (aka color_config)` are not reconfigured when loading local config files. I don't think this has priority, but would be cool to have.
```diff
cd clown_repo:poop: ; ls
-────┬───────────┬──────┬─────────────┬──────────────
+ #  │ name      │ type │ size        │ modified     
!────┼───────────┼──────┼─────────────┼──────────────
#  0 │ Android   │ Dir  │       4,096 │ 2 months ago 
-  1 │ Desktop   │ Dir  │       4,096 │ 1 week ago   
+  2 │ Documents │ Dir  │       4,096 │ 1 month ago  
!  3 │ Downloads │ Dir  │       4,096 │ 1 week ago   
#  4 │ Music     │ Dir  │       4,096 │ 2 months ago 
@  5 │ Pictures  │ Dir  │       4,096 │ 1 month ago  
+  6 │ Postman   │ Dir  │       4,096 │ 2 months ago 
!  7 │ Public    │ Dir  │       4,096 │ 3 months ago 
#  8 │ README.md │ File │         984 │ 4 days ago   
-  9 │ Templates │ Dir  │       4,096 │ 2 months ago 
+ 10 │ Videos    │ Dir  │       4,096 │ 2 weeks ago  
! 11 │ bin       │ Dir  │       4,096 │ 4 days ago   
@ 12 │ nushell   │ Dir  │       4,096 │ 4 days ago   
- 13 │ pictures  │ Dir  │       4,096 │ 3 months ago 
+ 14 │ repos     │ Dir  │       4,096 │ 1 hour ago   
! 15 │ tags      │ File │ 100,126,720 │ 1 month ago  
#────┴───────────┴──────┴─────────────┴──────────────
```
9. (Note: This is very similar to neovims `package` idea.)
With adding a `defs` directory, we already make the step from a config file, to a config directory. We can treat such a directory as a `package`. We could make sharing of `[theme]`, `defs` pretty easy by loading all packages under a common path (`$nu.packpath`). Example nu startup sequence:
- Nu is loading my global config / `package` ($HOME/.config/nu) first.
- Nu now loads all `packages` in `$nu.packpath`
    - `$nu.packpath` contains a package with a `defs` directory, full of wrapper around git commands. Those are loaded.
    - `$nu.packpath` also contains a package with a config.toml file having a `[theme]` table. Now nu sets the theme from it.

The nice thing about it is:
- `package`'s can be version controlled (simple git repo) and therefore be easily made available online.
- Writing a `package manager` is now the same as writing a wrapper around git.

There are some arguments speaking in favor of this approach:
1. Relativly easy
2. Local configs already brought the possibility to load (and unload) multiple `config.toml` files. This is nothing "completly" new.
3. Proof of concept is done by neovim.

This needs more design, but could be a general consideration when restructuring the config.

10. The `config` command may needs redesign. It does currently not take local configs into account. I think the following adoptions should be done:
    - Remove `config clear` (I personally think its unnecessary)
    - Remove `config path` (see `package list`)
    - Rename `config remove` to `config rm` (Better naming)
    - Add `package list` => List of tables of currently loaded configs/packages with path
    - Add `package load` => Load a config/package
    - Add `package unload` => Unload a config/package
    - Add `package init (path)` (Initialize package structure under path, so users don't have to look it up all the time)
    - Allow passing of --local to `config rm <key>` (Remove key from last loaded local config), `config set/set_into <key>` (Set key/value into last loaded local config), `package list` (Only show local configs).

# Drawbacks
[drawbacks]: #drawbacks

Everything done to the structure of the config is a breaking change for most users.

# Prior art
[prior-art]: #prior-art

I did a very little research what other shells are doing:
- Zsh: 
    - Package Manager: https://github.com/ohmyzsh/ohmyzsh
    - https://github.com/ohmyzsh/ohmyzsh/issues/6788 (TLDR: Install oh-my-zsh, add a .zsh file under a specific path). I am not sure how easy one can add someone else package, if he didn't contribute it to the (kinda central?) plugin repository under ohmyzsh.
- Fish:
    - Package Manager: https://github.com/oh-my-fish/oh-my-fish
    - Example Package: https://github.com/oh-my-fish/plugin-expand
    - Seems like fish has a well defined config directory structure. Maybe we could steal some ideas from them :)
