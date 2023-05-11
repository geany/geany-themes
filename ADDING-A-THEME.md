Adding a Theme to Geany-Themes Project
======================================

This document is meant to describe the steps needed to successfully
add a theme to the Geany-Themes project. This is the stuff I have to
do when someone contributes just a .conf file in order to integrate
it into the repository/project.

If you want to perform these steps yourself and contribute the complete
work as a pull request on Github as some have done before, that makes
my life a little easier (and gets your scheme in quicker), but even
just the bare (tested) .conf files are a fine contribution.

Style Guide
-----------

While there's no strict style guide for how the .conf file is formatted,
here's some notes about preferred style:

* It's easiest to start by using an existing colour scheme that is
similar to the one you want to create/port.
* Use HTML-style colours starting with a pound symbol (`#`), in
lower-case hex notation, compressing to 3 digits if possible. Examples:
  - #ff0
  - #e4b211
* Use `[named_colors]` (see Geany Manual) where appropriate, if you want.
* Most import is just to make it styled like most other schemes.

Adding the .conf file to the tree
---------------------------------

The name of the file should be similar to the name of the scheme,
unique amongst all other schemes, be lower-case, have words separated
by dashes (`-`) and end with the `.conf` extension.

Some examples:

    bespin.conf
    inkpot.conf
    solarized-dark.conf
    solarized-light.conf
    dark-fruit-salad.conf

The file goes into the `colorschemes` directory.

Testing the .conf file
----------------------

The most basic test needed is to run Geany from the command line
with the `-v` option and then load your color scheme. If Geany's
color scheme parser encounters any problems it will spit out some
debugging info onto the console. You should fix these warnings. You
can also access the same info by running Geany normally from a
shortcut/launcher and looking at Help->Debug Messages.

You should check out what the scheme looks like in a few different
language styles. For example, C++, Python, and XML. Using one
statically typed, one dynamically typed, and one tag/structured markup
language and you can find some weird differences that you might not
have noticed by just checking one or two very similar languages.

Some other important, not-obvious things to check:

* View->Show Whitespace
* View->Show Markers Margin
* View->Show Line Numbers
* View->Show Indentation Guides

Ensuring License and Credits
----------------------------

The top of the .conf file should contain any copyright and license
info that pertains to the scheme in a comment. If you ported from
another editor's colour scheme, try and keep the same license and
credit the original authors. If you create the scheme yourself or are
otherwise able to choose a license, it is recommended to use GPL v2
as a default.

You should add yourself and anyone else who have contributed or
originally authored the scheme to the `AUTHORS` file. There's a note
at the top of that file which explains how to use it. If you don't
wish to maintain the scheme (fix bugs, tweak colours, etc) then add
me (Matthew Brush) as the current maintainer like many of the plugins
have.

Please use your real name and a human-readable version of an email
address where you can be reached.

Adding a Screenshot
-------------------

This is the stupidest part since it's fully manual. I'll just
describe the way I do it, but it may be easier for others to do it
differently.

The screenshots are in PNG format. The file should be named the same as
the `.conf` file (obviously except for the `.png` extensions instead of
`.conf`). The filename should be all lower-case.

The font used in the screenshots is Andale Mono (don't ask why, it just is).
The font size is 10pt. If you can't get Andale Mono font (I think it might
be a non-free MS core font), use an existing font that looks similar.

The contents of the file in the screenshot are a "Hello World"
program in the C programming language, and are exactly this:

```c
#include <stdlib.h>
#include <stdio.h>

#define MESSAGE "Hello World"

/* Prints a message to standard output */
void print_message(const char *msg) {
	printf("%s\n", msg);
}

int main(int argc, char *argv[]) {
	print_message(MESSAGE);
	return 0;
}

```

You should turn off `View->Show Whitespace` and
`View->Show Indentation Guides`. The tab mode should be set to 4
character width real/hard tabs. Turn on markers and line number margin
under View menu. Place the caret on the 3rd line (which is an empty line)
and click on line four's marker margin to add a mark on that line. Just
look at existing screenshots and make it look the same/similar.

I just use the screenshooter tool that comes with my distro (Xubuntu)
but many tools could be used to grab a screenshot. I look at existing
screenshots and try to select the same region of the screen to capture
so that it looks roughly the same size/area as the existing screenshots.
It's really not scientific at this point, the key is to just make them
all look the same except for the color scheme.

If you're on MS Windows, you could get the Geany instance all ready
and then press the Print Screen key to copy the screen contents into
the clipboard and then go into a drawing program like MS Paint or GIMP
or whatever program you like and crop out the similar region as the
existing screenshots.

I would really like to somehow automate this whole step, suggestions
and ideas are most welcome.

Updating Meta-Data
------------------

There are some meta-data files in the repository that (are not yet)
used by some plugin to list/update geany schemes. It's simple to update
these files by running `make index` command in the root directory
of the tree.

If you don't have GNU make (ex. on Windows), don't worry about doing
this step, it's entirely trivial for me to do it.

IMPORTANT NOTE 2023-05-11 by Evgueni Antonov (github: StrayFeral): 
The Python code in scripts sub-directory
is strictly Python2, which was not clear initially to me, neither it
was specified. However Python 2 is officially "sunseted" as of
January 1, 2020. I thought small adjustments would make it work on
Python 3.10, however it looks there are more differences on how the
modules work. Therefore, to update the meta-data I personally pulled
the official Python2 container from the Docker repo, instantiated a
container in interactive mode, put the code there and ran `make index`
on the container. In short, this is what one could do in 2023:

The folowing is a bash session on Lubuntu Jammy 22.04 LTS, Docker 
version 23.0.6, build ef23cbc (for clarity, I left only the commands,
not the output). If you already have Python2 on your system, you
won't need to do all this. However the code would need a Python 3
migration sooner or later and I don't plan to maintain a Python 2
eco-system on my machine, so I got it as a container environment:

```
sudo docker image pull python:2
sudo docker run -it python:2 bash

# From now on, the rest is executed on the container shell.
# By the way the container is a Debian 10 image with Python 2.7.18

apt update
apt upgrade
apt install wget

# This however does not seem to work, maybe it is only the Python 3 version
apt install python-pil

python -m pip install --upgrade pip

# Pillow is a fork of PIL. In my case PIL did not worked
pip install pillow

cd /tmp
wget https://github.com/geany/geany-themes/archive/refs/heads/master.zip
unzip master.zip
cd geany-themes-master/

# Now copy-paste your new theme_name.conf in colorschemes/
# and then your screenshot_name.png in screenshots/

make clean
make index

# In 2023 the folowing is a bit barbarian, I know. But it works.
# You can go in a better way if you mount your native filesystem to
# the container, but I was in a hurry and as I said - it works.

# So now we simply cat the files we need and we copy-paste them
# in our native filesystem:
cat index.json.md5
cat index.json # This one is large !
```

Now as you finally have a properly generated JSON indexes, you can
commit to the repo.

Making a Pull Request
---------------------

The pull request to add the scheme should ideally be a single commit with
all of the required changes made.

Here's a pretty good example of a commit to the repo:

https://github.com/codebrainz/geany-themes/commit/d81d7b5142034f89e9e19eac58bd43ed54121888

This is the actual commit I made while writing this guide:

https://github.com/codebrainz/geany-themes/commit/f9043abdd7247b742176df5d0d867656f24f9f88

I find it easiest to clone the geany-themes repo, checkout a new branch
(ex. `git checkout -b my-new-theme`) and then keep adding changes to
that until it's all ready. From there you can use `git rebase --interactive` to
squash the commits into a single commit and add your nice descriptive
commit message for the whole lot. With your branch having a single commit
difference from Geany-Themes master branch, create a pull request on
Github to get me to add it into the master branch.

These are just recommendations, however you provide the scheme, I'll try
and get it integrated into the repo. Just don't expect fast response time
unless you've done most of the work for me :)
