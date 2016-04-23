---
layout: post
title:  "Lightweight Server Configuration CM"
date:   2014-10-26 22:17:00
categories: administration
---

My personal data is generally covered. Most of the time, it is either hosted
somewhere else (github, Docs), or it is backed up using a straightforward
process (Dropbox, NAS). Data rides through a crash or system upgrade just
fine.

System configuration is another matter. Recovery after recovery, I've looked
into mechanisms for a system configuration backup strategy that meets my needs.
Each time, I've come to the conclusion that manual recovery is the best option
I could find.

The traditional tools are just too heavyweight for me. Puppet, Chef, SaltStack,
CFEngine ... I've tried them all. Each requires a learning curve to get started,
and each usually introduces topics I am just not interested in (masters,
slaves, keys, dependencies, ... Ruby). I need the capability rarely enough
that there is relearning required with each session. For each, generating
configuration recipes felt too much like software development. It should be
easier. Or at least it needs to be easier for me.

At Ohio LinuxFest last weekend, [Roberto Sánchez][] did a [presentation][]
that ended up being about [etckeeper][]. This is definitely a step in the
right direction for me. The one thing I'd say is that it may focus more on
managing changes for potential rollback. I just want to define and restore
quickly. It also would mix my services configurations together. I'd like to
keep them separate.

[Roberto Sánchez]: http://people.connexer.com/~roberto/main
[presentation]: http://people.connexer.com/~roberto/documents/olf2014/using_git_to_manage_your_systems_configuration.pdf
[etckeeper]: https://joeyh.name/code/etckeeper/

Serendipitously, Reddit r/programming last week had a [reference][] to the use
of [GNU stow][] for use in managing/backing up home directory configuration
files. I could see this plus etckeeper solving my problem fine. Stow serves as
a service-specific repository of system configuration, and etckeeper places
it and /etc into a git repository.

[reference]: http://www.reddit.com/r/linux/comments/1f7sh4/gnu_stow_manage_your_usrlocal_with_ease/
[GNU stow]: http://www.gnu.org/software/stow/

But, for the moment I've settled on an even easier path - [shell][]
[scripts][]. They live in an environment that I won't forget before the next
system crash. Generating scripts is only about as hard as writing a setup
howto. Configuration is stored as edits rather than complete (version
specific) files.

[shell]: https://github.com/davesteele/server-setup-scripts/blob/master/pptpd/setup-pptp.sh
[scripts]: https://github.com/davesteele/server-setup-scripts/blob/master/tor/setup-tor-relay.sh

P.S. - Another option, [cloud-init][], is a strong contender. The format is
simple enough, and many services let you use it to define configuration at
VPS creation time. But, it still is yet another format, it addresses a
cross-distribution compatibility problem that I just don't care about, and
it is not available everywhere (and would therefore need to be bootstrapped).
I'll stick to scripts for now.

[cloud-init]: http://cloudinit.readthedocs.org/en/latest/
