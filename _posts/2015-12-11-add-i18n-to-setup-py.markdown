---
layout: post
title:  "Add i18n support to your Python Package Install"
date:   2015-12-11 i10:50:00
categories: development
---

Python support for the translation file installation is not great. Once you
understand [how the the i18n build process works], it's not too hard to
hand-roll the tasks yourself. Here's how I did it.

[how the the i18n build process works]: https://davesteele.github.io/debian/development/2015/12/10/i18n-build-flow/

First, add this code to setup.py:


    from distutils.core import setup, Command
    from distutils.command.build import build
    from distutils.command.clean import clean
    
    import os
    import shutil
    
    package = "mypackage"      # <- change this
    
    podir = "po"
    pos = [x for x in os.listdir(podir) if x[-3:] == ".po"]
    langs = sorted([os.path.split(x)[-1][:-3] for x in pos])
    
    
    def modir(lang):
        mobase = "build/mo"
        return os.path.join(mobase, lang)
    
    
    def mkmo(lang):
        outpath = modir(lang)
        if os.path.exists(path):
            shutil.rmtree(path)
        os.mkdir(outpath)
    
        inpath = os.path.join(podir, lang + ".po")
    
        cmd = "msgfmt %s -o %s/%s.mo" % (inpath, outpath, package)
    
        os.system(cmd)
    
    
    def merge_i18n():
        cmd = "LC_ALL=C intltool-merge -u -c ./po/.intltool-merge-cache ./po "
        for infile in (x[:-3] for x in os.listdir('.') if x[-3:] == '.in'):
            print("Processing %s.in to %s" % (infile, infile))
    
            if 'desktop' in infile:
                flag = '-d'
            elif 'schema' in infile:
                flag = '-s'
            elif 'xml' in infile:
                flag = '-x'
            else:
                flag = ''
    
            if flag:
                os.system("%s %s %s.in %s" % (cmd, flag, infile, infile))
    
    
    class my_build(build):
        def run(self, *args):
            build.run(self, *args)
    
            for lang in langs:
                mkmo(lang)
    
            merge_i18n()
    
    
    def polist():
        dst_tmpl = "share/locale/%s/LC_MESSAGES/"
        polist = [(dst_tmpl % x, ["%s/%s.mo" % (modir(x)]), package) for x in langs]
    
        return polist
    
    
    class my_build_i18n(Command):
        description = "Create/update po/pot translation files"
        user_options = []
    
        def initialize_options(self):
            pass
    
        def finalize_options(self):
            pass
    
        def run(self):
            print("Creating POT file")
            cmd = "cd po; intltool-update --pot --gettext-package=%s" % package
            os.system(cmd)
    
            for lang in langs:
                print("Updating %s PO file" % lang)
                cmd = "cd po; intltool-update --dist \
                       --gettext-package=%s %s >/dev/null 2>&1" % (package, lang)
                os.system(cmd)
    
    
    class my_clean(clean):
        def run(self):
            clean.run(self)
    
            filelist = [x[:-3] for x in os.listdir('.') if x[-3:] == '.in']
            filelist += ['po/.intltool-merge-cache']
            for infile in filelist:
                if os.path.exists(infile):
                    os.unlink(infile)
    
            for dir in ['build/mo', 'build/scripts-2.7', 'build/scripts-3.4'
                        'build/scripts-3.5']:
                if os.path.exists(dir):
                    shutil.rmtree(dir)

Then modify the *setup()* call to use these new commands, and to add the
compiled translation files to the install:

    setup(
        name=package,
        ...
        data_files=[
            ('share/icons/hicolor/16x16/apps', ['icons/16x16/mypackage.png']),
            ...
                   ] + polist(),
        ...
        cmdclass={
            'build_i18n': my_build_i18n,
            'clean': my_clean,
            'build': my_build,
                 },
         )

This process adds a new command, *python setup.py build_i18n*, which will
create/update the *.pot* and *.po* files in */po*. The normal build/install
processed is enhanced to install compiled *.mo* files in */usr/share/locale/*.
