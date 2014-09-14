---
layout: post
title:  "Is Documentation Considered Harmful?"
date:   2014-09-14 08:05:12
categories: debian linux
---
Is Documentation Considered Harmful?
====================================

It's commonly observed that developers don't like to work on documentation. It's commonly concluded that the reason is that documentation is the boring part.

But, I'm starting to wonder if there is a "High Priest of Knowledge" aspect to the aversion.

Here are a couple of incidents I observed at [Debconf14](http://debconf14.debconf.org/).

***

Python BOF
----------

In the Python BOF, There was an early statement from the chair on the current preferred way to package Python modules. A participant stated that the Debian wiki directly contradicts that. The chair made several statements that he/they couldn't be expected to look beyond the main Python wiki page, and later added that he hadn't looked at that page in quite some time, and didn't know if it was right or not.

The action to fix the documentation was captured in the gobby notes, but, shortly thereafter, all references to documentation updates were gone.

I can kind of understand avoiding the work, but ... deleting the action?

Debian Keyring Management
-------------------------

There was a question that came up for me as I was doing some key management prep for the keysigning. Stated generally:

* What is the scope of a key signature? i.e.
    - What does it the signature explicitly cover?
    - What is covered by extension/convention? or not covered?

Or, a bit more specifically:

* What signed aspects of a key are invalidated by changes after the key signature?
* What aspects are covered if added after the signature?

Or, in its concrete form:

* What do you need to do to prep a signing subkey for a keysigning?

Keysigning is covered in gpg documentation and the Wiki, as are subkeys, but I found nothing that covers both in relation to each other.

I brought this up in the Debian Keyring talk, as an aspect of a bigger problem with the help available on understanding keys. The answer I got from the front is that they "knew too much" about the technology to be able to form a useful answer to the question, and that someone with much less knowledge (presumably me) should be the one expected to tackle it.

BTW, apparently the answer to the concrete question is "Nothing".

Infrastructure Update Process
-----------------------------

One session was about the paradoxes associated with the current process for updating Debian infrastructure. This was involved, but one core assertion was that the person developing a new feature needs to be responsible for insuring it works right. On the flip side, the infrastrucure folk felt no responsibility to provide platform support, saying that documentation on replication was sufficient. When it was pointed out that it was not possible to do so (possibly because the schema is not  correct in any available source), they were unswayed.

***

In each of these cases, I couldn't avoid the feeling that the documentation-avoider was deriving a benefit from the lack of information being made available.

So, why do you avoid documentation?
