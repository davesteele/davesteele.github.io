---
layout: post
title:  "Debian watch file format for signed Github tar files - redux"
date:   2022-10-05 21:44:00
categories: development
---

Seven years ago I wrote down how to [format Debian uscan watch files for
GitHub](https://davesteele.github.io/development/2015/05/02/debian-watch-file-format-for-signed-github-source-tars/)
sources, so I would be able to find it later. Last month, it got
[harder](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019696) to use this
type of strategy to track some files, because GitHub no longer will let
you scrape the *Releases* page for source tarballs.  That bug report
[suggested](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1019696#20) a
workaround strategy - enumerate the possibilities on the *Tags* page, and morph
the results to the *Releases* page

Here's how to do that. I'm going to use the
[_sirikali_](https://github.com/davesteele/sirikali) package as an example.
Refer to the [previous
post](https://davesteele.github.io/development/2015/05/02/debian-watch-file-format-for-signed-github-source-tars/),
or to the [uscan](http://manpages.debian.org/cgi-bin/man.cgi?query=uscan) man
page, for detail on the file format.

First, edit the *watch* file to point to the *Tags* page, looking for *.tar.gz* files:

    format=4
    
    https://github.com/mhogomchungu/@PACKAGE@/tags \
    /mhogomchungu/@PACKAGE@/archive/refs/tags/(.*)\.tar\.gz

If you are looking for a *tar.gz*, you may be done at this point. I am after a
signed *tar.xz* file on the *Releases* page, so lets remap the download url to point
to that:

    format=4
    
    opts=downloadurlmangle=s/archive\/refs\/tags\/(.*)\.tar\.gz/releases\/download\/$1\/@PACKAGE@-$1\.tar\.xz/ \
    https://github.com/mhogomchungu/@PACKAGE@/tags \
    /mhogomchungu/@PACKAGE@/archive/refs/tags/(.*)\.tar\.gz

... and specify the location of the expected signature file (append ".asc"):

    format=4
    
    opts=downloadurlmangle=s/archive\/refs\/tags\/(.*)\.tar\.gz/releases\/download\/$1\/@PACKAGE@-$1\.tar\.xz/,\
    pgpsigurlmangle=s/$/.asc/ \
    https://github.com/mhogomchungu/@PACKAGE@/tags \
    /mhogomchungu/@PACKAGE@/archive/refs/tags/(.*)\.tar\.gz

Test with:

    uscan -v -dd

If you see a "Good signature" in the output, you are in good shape.
