---
layout: post
title:  "Debian watch file format for signed Github tar files"
date:   2015-05-02 21:44:00
categories: development
---

Edit 2022-10-08 - GitHub has just made a change that makes it harder to work
wite source files on the *Releases* page. See an
[update](https://davesteele.github.io/development/2022/10/05/debian-watch-file-for-github-redux/)
covering a workaround

I just added uscan source tar signature validation to a project stored on
GitHub. A cookbook approach may be helpful to others.

The [uscan man page][] and [wiki watch page][] both provide a watch file
recipe for pointing at GitHub pages:

[uscan man page]: http://manpages.debian.org/cgi-bin/man.cgi?query=uscan
[wiki watch page]: https://wiki.debian.org/debian/watch#GitHub

    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/<project>-$1\.tar\.gz/ \
      https://github.com/<user>/<project>/tags .*/v?(\d\S*)\.tar\.gz

Here's what's going on. You can refer to GitHub repositories as tar.gz files
from the [tags][] page, which list all tags stored on the
repository. The problem is, that the naming isn't right for debian - the
tar file name just consists of \<version\>.tar.gz. The 'filenamemangle'
option remaps the GitHub tar name to one including the project name, before
storing the tar locally.

[tags]: https://github.com/davesteele/comitup/tags

This is actually not good enough, as it stands, to work in an environment where
tags exist for multiple branches (if, for instance, the repository also has a
"debian" branch). The final glob should be more specific, to match only against
the branch holding the 'upstream' (that is, the non-debian) code branch.

Meanwhile, [typical guidance][] for specifying the signature url, using the pgpsigurlmangle
option, tells how to point to the file when it is stored in the same directory
as the tar, with an appended ".asc":

[typical guidance]: https://wiki.debian.org/debian/watch#Cryptographic_signature_verification

    opts=pgpsigurlmangle=s/$/.asc/ ftp://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-(.+)\.tar\.gz

Unfortunately, it is not practical to store the signature on the GitHub tags
page.

I stored the signatures in a dedicated [signatures][] branch, and used a
[custom][] 'pgpsigurlmangle' definition to point to a raw version of the
[appropriate signature][]. The final result:

[signatures]: https://github.com/davesteele/comitup/tree/signatures
[custom]: https://raw.githubusercontent.com/davesteele/comitup/debian/debian/watch
[appropriate signature]: https://raw.githubusercontent.com/davesteele/comitup/signatures/comitup-0.1.tar.gz.asc

    opts=filenamemangle=s/.+\/v?(\d\S*)\.tar\.gz/@PACKAGE@-$1\.tar\.gz/,\
    pgpsigurlmangle=s/github.com/raw.githubusercontent.com/;\
    s/archive\/master/signatures/;\
    s/([^\/]+)\.tar\.gz/@PACKAGE@-$1\.tar\.gz/;\
    s/$/.asc/ \
     https://github.com/davesteele/@PACKAGE@/tags .+master/(\d[\d\.]*)\.tar\.gz

A couple things to note here:

* That is a single, continued line.
* There are two white space occurances in that line, separating the definition
into an option specification, a tar listing page url, and tar search glob.
* The two options defined in the 'opts' spec are separated by a comma.
* _uscan_ replaces _@PACKAGE@_ with the package name from the most current
  _changelog_ entry.
* The multiple rules in the pgpsigurlmangle definition are separated by
semicolons.
* There are four replacement rules defined for the signature url:
  * The first one replaces the url site name with the GitHub 'raw' host.
  * The second one points to the right branch.
  * The third one fixes the file name.
  * The fourth one adds the 'asc' extension.
* I'm using 'master' as the git-buildpackage 'upstream' branch.
* Occurrences of 'davesteele' will need to be replaced for
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


