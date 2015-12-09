---
layout: post
title:  "Understanding the i18n Build Flow"
date:   2015-12-10 21:29:00
categories: debian development
---

Wherein I detail how human language translation is incorporated into software
packages as a part of the build process.

## Two Translation Build Environments

The grandaddy package for integrating [i18n] translations into software is GNU
[gettext]. First released in 1995, gettext initially focused on providing
translation services for software written in 'C'. While it includes copious
[documentation] on details of its many components, the expectation seems to be
that most developers interact with gettext by way of [autotools].

[i18n]: https://en.wikipedia.org/wiki/Internationalization_and_localization#Naming
[gettext]: https://www.gnu.org/software/gettext/
[documentation]: https://www.gnu.org/software/gettext/manual/html_node/index.html
[autotools]: https://www.guyrutenberg.com/2014/11/01/gettext-with-autotools-tutorial/

Note that the term 'gettext' refers both to the package name as well as the
[function]/[utility] that converts a string to its translated form.

[function]: http://linux.die.net/man/3/gettext
[utility]: http://linux.die.net/man/1/gettext

The 'C'/Autotools focus for gettext was perhaps the driving force for the
development of [intltool] by freedesktop.org. Intltool acts as a wrapper
around gettext, simplifying the interface while at the same time expanding
the types of files supported (at least historically). The core
intltool documentation consists of the included [man pages] of the components.

[intltool]: http://freedesktop.org/wiki/Software/intltool/
[man pages]: http://www.die.net/search/?q=intltool-&sa=Search&ie=ISO-8859-1&cx=partner-pub-5823754184406795%3A54htp1rtx5u&cof=FORID%3A9&siteurl=linux.die.net%2Fman%2F&ref=&ss=2303j1110801j9

There are also [other environments] available for working in this same space.
Generally, these are more limited in scope, and often not as well supported.

[other environments]: https://docs.python.org/3.5/library/gettext.html#internationalizing-your-programs-and-modules

While learning these environments, I felt that both gave too little emphasis
on the big picture. This became most acute as I was dealing with the limited
built-in i18n support for some [languages other than 'C'].

[languages other than 'C']: https://www.debian.org/doc/manuals/intro-i18n/ch-otherlanguage.en.html#s-fortran

So, here's how human language translation support gets built into your package.
For each step, I'll show the commands in both intltool and gettext to perform
the action. In the case of gettext, I'm making a stab here. I use intltool
exclusively.

## Two Translation Strategies

For software, translation information is compiled into binary,
language-specific *.mo* files. Those files are consulted at run time,
as needed, to convert English strings into appropriate localized ones.

For text and data files, all of the translated strings are often added to the
source file at build time. XML files and [.desktop] files fall into this
category.

[.desktop]: https://developer.gnome.org/desktop-entry-spec/

Though the build process handles both cases at the same time, we'll
consider them separately.

### The Build Steps (run-time translation)

#### _Step 1 - String identification in software source_

The means to do this are similar-but-different across software
languages. For Python, the following is required at the top of the file:

    import locale
    import gettext
    
    locale.setlocale(locale.LC_ALL, '')
    gettext.textdomain(<package name>)
    _ = gettext.gettext

After that, use the underscore function you just created to both identify and
convert text strings to the appropriate language:

    print(_("Please print this in my language"))

Here are similar instructions for [C], [Ruby], and [Bash].

[C]: https://fedoraproject.org/wiki/How_to_do_I18N_through_gettext
[Ruby]: http://www.rubydoc.info/gems/gettext/#Usage
[Bash]: http://stackoverflow.com/questions/2221562/using-gettext-in-bash


#### _Step 2 - Create the Translation Template (.POT) file_

Using gettext:

    $ cd po; xgettext -f POTFILES.in -f <package name>.pot -a

Using intltool:

    $ cd po; intltool-update --pot --gettext-package=<package name>

In both cases there must first be a *po/* directory in your source tree, and a
*po/POTFILES.in* file which defines the source files to be processed, relative
to the root of the project.

The result is a *po/&lt;package name&gt;.pot* file ([example]), which contains all of the strings
to be translated.

'POT' stands for "Portable Object Template", if that helps.

[example]: https://github.com/davesteele/gnome-gmail/blob/master/po/gnome-gmail.pot

#### _Step 3 - Transform the POT to individual (.PO) translation files_

This is essentially a manual process. For each target language, the
*po/&lt;package name&gt;.pot* file is copied to *po/&lt;language&gt;.po*,
or to *po/&lt;language&gt;-&lt;COUNTRY&gt;.po*, where *&lt;language&gt;*
is an [ISO 639-1] two-letter language code, and *&lt;COUNTRY&gt;.po* is a
two-letter [ISO 3166-1] (capitalized) two-letter country code ([po example]).

[ISO 639-1]: https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
[ISO 3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1
[po example]: https://github.com/davesteele/gnome-gmail/blob/master/po/pt_BR.po

The resulting text file is hand-edited to add the translation
strings. This can be done with an ordinary text editor, though there are
special editors for the job. [Poedit] is a popular choice.

[Poedit]: https://poedit.net/

The skill set required for this step is obviously very different than for
other software development tasks. A number of communities have been set up
to facilitate those with the ability to do this job well. [Ubuntu] has done
a good job of this, for software which is stored in their
source control system. [Debian] also has a means to collect and [status] all of
the software in the distribution which supports i18n. It's not difficult to
host a [translation development environment] on your own.

[Ubuntu]: https://wiki.ubuntu.com/Translations/Contact/Teams
[Debian]: https://www.debian.org/international/l10n/
[status]: https://www.debian.org/international/l10n/po/fr
[translation development environment]: https://davesteele.github.io/gnome-gmail/translation.html

#### _Step 4 - Update the PO files_

Using gettext, for every *&lt;language&gt;[-&lt;COUNTRY&gt;].po* file present in */po*:

    $ cd po; msgmerge -u --backup=none <language>.po <package name>.pot

Using intltool, for every *&lt;language&gt;[-&lt;COUNTRY&gt;]* PO file present in */po*:

    $ cd po; intltool-update --dist --gettext-package=<package name> <language>

Once created, the *.po* files need to be updated whenever there are changes
to the *.pot* file. These command will do that - refreshing the strings to be
translated and their source line numbers.

Beyond that, the steps up to this point do not need to be executed with every build.

#### _Step 5 - Transform the PO files to binary (.MO) translation files_

Using gettext, for every *&lt;language&gt;[-&lt;country&gt;].po* file present in */po*:

    $ msgfmt po/<language>[-<COUNTRY>].po -o <build_dir>/mo/<language>[-<COUNTRY>]/<package name>.mo

Intltool uses the same command.

#### _Step 6 - Incorporate the MO files into the package_

Each *.mo* file gets installed with the package, to the path
*/usr/share/locale/&lt;language&gt;[-&lt;COUNTRY&gt;]/LC_MESSAGES/&lt;package name&gt;.mo*.

### The Build Steps (build-time translation)

#### Step 1 - _String identification in text source_

Every *&lt;file&gt;* in the build environment which is gong to be translated
is renamed to *&lt;file&gt;.in*.

Edit the *.in* files to identify translatable strings. For desktop files,
prepend the translatable entries with an underscore:

    [Desktop Entry]
    _Name=My Package
    _GenericName=Doing Software Right
    _Comment=Integrate GMail with your desktop
    MimeType=application/mbox;message/rfc822;x-scheme-handler/mailto;
    ...

For XML files, prepend translatable tags with an underscore. After
translation, they will be repeated in the file for each language, with an
"xml:lang" attribute identifying the language.


    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE default-apps SYSTEM "gnome-da-list.dtd">
    <default-apps>
      <mail-readers>
        <mail-reader>
          <_name>My Package</_name>
    ...

#### _Steps 2 through 4 - Manage POT and PO files_

These steps are identical to the ones listed above. Add the *&lt;file&gt;.in* files
to *POTFILES.in*, and run as described.

#### _Step 5 - Translate the text files_

Using gettext:

    $ msgfmt <flag> --template=<file>.in -d po -o <file>

Using intltool:

    $ intltool-merge -u -c ./po/.intltool-merge-cache ./po <flag> <file>.in <file>

Note that *&lt;flag&gt;* defines the text file format. See the corresponding
man pages for details. Also note that msgfmt expects a *po/LINGUAS* file to
list all of the PO files present.

### A Special Case - GUI Definition Files

Translations are handled differently by different GUI environments. Refer to
the documentation for the environment you are using.

For Glade, translatable strings are flagged using the Glade editor. The Glade
file is added to *po/POTFILES.in*, and processed normally to the MO
translation files.

### Cleanup

The process described here will leave a *&lt;file.in&gt;* for every
*&lt;file.in&gt;*, the *&lt;build&gt;/mo* directory tree, and a
*po/.intltool-merge-cache* file. These will need to be cleaned up.

## Summary

At this point, your program supports translation. To demonstrate,
first, add support for another language to your system:

Ubuntu

    $ sudo apt-get install language-pack-<language>

Debian

    $ sudo dpkg-reconfigure locales

List the locales available:

    $ locale -a
    C
    C.UTF-8
    en_US.utf8
    fr_FR.utf8
    POSIX

Use an alternative locale to run your program:

    $ LC_ALL=fr_FR.utf8 my-program

If life is good, you'll see your work in a different language.
