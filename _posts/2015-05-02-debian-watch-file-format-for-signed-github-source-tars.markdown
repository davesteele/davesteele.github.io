---
layout: post
title:  "Debian watch file format for signed Github tar files"
date:   2015-05-02 21:44:00
categories: development
---

I just added uscan source tar signature validation to a project stored on
GitHub. A cookbook approach may be helpful to others.

The [uscan man page][] and [wiki watch page][] both provide a watch file
recipe for pointing at GitHub pages:

[uscan man page]: http://manpages.debian.org/cgi-bin/man.cgi?query=uscan
[wiki watch page]: https://wiki.debian.org/debian/watch#GitHub

    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/<project>-$1\.tar\.gz/ \
      https://github.com/<user>/<project>/tags .*/v?(\d\S*)\.tar\.gz

Here's what's going on. You can refer to GitHub repositories as tar.gz files
from the [tags][] or [releases][] pages, which list all tags stored on the
repository. The problem is, that the naming isn't right for debian - the
tar file name just consists of \<version\>.tar.gz. The 'filenamemangle'
option remaps the GitHub tar name to one including the project name, before
storing the tar locally.

[tags]: https://github.com/davesteele/splitcpy/tags
[releases]: https://github.com/davesteele/splitcpy/releases

This is actually not good enough, as it stands, to work in a git-buildpackage
environment. The final glob should be more specific, to match only against the
branch holding the 'upstream' (that is, the non-debian) code branch.

[Typical guidance][] for specifying the signature url, using the pgpsigurlmangle
option, tells how to point to the file when it is stored in the same directory
as the tar, with an appended ".asc":

[Typical guidance]: https://wiki.debian.org/debian/watch#Cryptographic_signature_verification

    opts=pgpsigurlmangle=s/$/.asc/ ftp://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-(.+)\.tar\.gz

Unfortunately, it is not practical to store the signature on the GitHub tags/release
pages.

I stored the signatures in a dedicated [signatures][] branch, and used a
[custom][] 'pgpsigurlmangle' definition to point to a raw version of the
[appropriate signature][]:

[signatures]: https://github.com/davesteele/splitcpy/tree/signatures
[custom]: https://raw.githubusercontent.com/davesteele/splitcpy/debian/debian/watch
[appropriate signature]: https://raw.githubusercontent.com/davesteele/splitcpy/signatures/splitcpy-0.1.tar.gz.asc

    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/splitcpy-$1\.tar\.gz/,\
    pgpsigurlmangle=s/github.com/raw.githubusercontent.com/;\
    s/archive\/master/signatures/;\
    s/([^\/]+)\.tar\.gz/splitcpy-$1\.tar\.gz/;\
    s/$/.asc/ \
     https://github.com/davesteele/splitcpy/tags .+master/(\d[\d\.]*)\.tar\.gz

A couple things to note here:

* That is a single, continued line.
* There are two white space occurances in that line, separating the defninition.
into an option specification, a tar listing page url, and tar search glob.
* The two options defined in the 'opts' spec are separated by a comma.
* The multiple rules in the pgpsigurlmangle definition are separated by
semicolons.
* There are four replacement rules defined for the signature url:
  * The first one replaces the url site name with the GitHub 'raw' host.
  * The second one points to the right branch.
  * The third one fixes the file name.
  * The fource one adds the 'asc' extension.
* I'm using 'master' as the git-buildpackage 'upstream' branch.
* Occurrences of 'davesteele' and 'splitcpy' will need to be replaced for
other instances.

The key to be used to verify the tar file must be stored under the debian
directory.
Some [older announcements][] state that the key is stored in
*debian/upstream-signing-key.pgp*. Current documentation says that a binary key
can be stored in *debian/upstream/signing-key.pgp*, or an ascii armored one in
*debian/upstream/signing-key.asc*. All currently work, but support for the
first form will be eventually removed.

[older announcements]: http://debian-administration.org/users/dkg/weblog/106

Also, the key can be replaced with a keyring for team project support.


