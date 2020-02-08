---
title: "How to make PGI work Mac OS X Catalina"
date: 2020-02-08T11:21:13+01:00
draft: false
---

## Story

While recently trying to install PGI compilers (community edition 2019) onto my Macbook Air with Mac OS X Catalina I had several problems with compiline simple, hello_world type of program. Hopefully this post will help others with similar problems, and get them up to speed...

## Installation

Installation process is pretty straighforward, as you just need to download .dmg file that, when opened/mounted, presents the only file for your consideration and installation as a standard Mac OS X package. The easiest way to install such package is by double clicking on it in Finder window, although some  might prefer command line interface with `/usr/sbin/installer`

## Post Installation

It's not super clear during the installation, but PGI compilers land in `/opt/pgi` folder that by default is not added to the `PATH`. For my installation I have created following file in my home directory.

```shell
$ cat .pgi
# PGI compiler
export PGI=/opt/pgi
export PATH=/opt/pgi/osx86-64/2019/bin:$PATH
export MANPATH="$MANPATH":/opt/pgi/osx86-64/2019/man
export LM_LICENSE_FILE="$LM_LICENSE_FILE":/opt/pgi/license.dat
```

Depending on which exactly version have been downloaded/installed, the paths might differ. To activate PGI compilers in current shell session, following command can be issues:

```shell
$ source ~/.pgi
```

## The First Compilation

Usually I'm trying to compile simple hello-world type of program, to confirm that basic functionality works:

```fortran
       program test
       
       do 10 i=1,5
 10      print *,'hello world'
       
       end
```

Unfortunately in MacOS X Catalina, after setting all environmental variables, linking phase of the compilation throws an error:

```shell
$ pgf77 t.c
ld: file not found: /usr/lib/crt1.o
```

After spending some time and experimenting with different compiler/linker switches and environmental variables that control paths for libraries (e.g. LD_LIBRARY_PATH), I was able to identify the issue.

In PGI bin directory there is a file `localrc`, which is used by PGI compilers for default settings. One can "regenerate" this file by using provided `makelocalrc` that happen to contain follwing piece of code:

```shell
if test $xcodever -ge 10 ; then
  # /usr/lib/crt1.o doesn't exist on OS X Mojave
  print_line 'set LCRT1=;'
fi
```

## Solution

Adding following line to `/opt/pgi/osx86-64/2019/bin/localrc` solved compilation issue

```
set LCRT1=;
```

so now the code can be compiled and executed

```shell
$ pgf77 t.f
$ ./a.out
 hello world
 hello world
 hello world
 hello world
 hello world
 ```