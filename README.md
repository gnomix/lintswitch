
# Summary

*lint_switch* is a small Bash script that connects:

1. _incron_ which watches your files
1. _pylint_, _jslint_ and other linters, which warn you about possible problems
in your files
1. _zenity_, _notify-send_, and _imagemagick_, which display the lint result.

The result is that your code is constantly being watched and linted,
in the background, without interrupting your workflow, unless
there is an error in your code.

# Installation

I have only tested lint_switch on Ubuntu. It should work on other Unix-based operating systems, as long as you can install the requirements.

## Install the requirements

    sudo apt-get install libnotify-bin incron zenity pylint pep8 rhino gconf2 imagemagick

You will probably already have some of these.

 - libnotify-bin gives us _notify-send_
 - rhino is needed to run _jslint_
 - imagemagick converts text to an image and gconf2 sets it as the desktop background

**To get Google Closure Lint:**

    cd <temp_dir>
    svn checkout http://closure-linter.googlecode.com/svn/trunk/ closure-linter
    cd closure-linter
    python setup.py install

**To get jslint:**

The original jslint only runs on a webpage. There are a variety of wrappers to get it running in other environments. 
I am using (and I recommend) jslint4java, a Java wrapper which uses the Rhino javascript engine, allowing us to run jslint from a unix command line.

    cd <temp_dir>
    wget http://jslint4java.googlecode.com/files/jslint4java-1.4.4-dist.zip
    unzip jslint4java-1.4.4-dist.zip 
    cd jslint4java-1.4.4
    cp jslint4java-1.4.4.jar [somewhere]/

For example, I put jslint4java-1.4.4.jar in my ~/Applications/ directory.

## Check out / clone lint_switch

    git clone git@github.com:grahamking/lint_switch.git

I like to symlink it from my bin directory

    cd ~/bin
    ln -s [your checkout]/lint_switch.sh

Open _lint_switch.sh_ and edit the JSLINT_DIR to point to where jslint4java-1.4.4.jar lives.

For example my JSLINT_DIR is _/home/graham/Applications_

## Setup incron

Add your username to /etc/incron.allow:

    sudo bash -c "echo $(whoami) >> /etc/incron.allow"

Next, tell incron what to watch. 

First generate a list of relevant files:

 - Replace [full_proj_path] with the full path to your project. For example for a Django project I worked on recently I used _/home/graham/Projects/fablistic/fabproject_. That is also the directory the linter will run from.  
 - Note that you have to include the path twice in this command.
 - Replace [full_lint_switch_path] with the path and name for lint_switch.sh. For example I use _/home/graham/bin/lint_switch.sh_
 - This will find all directories which contain 'py', 'js', or 'css' files. Customise as required.

Here's the command:

    find [full_proj_path] -name "*.py" -or -name "*.js" -or -name "*.css" | xargs -l1 dirname | sort | uniq | awk '{print $1" IN_ATTRIB [full_lint_switch_path] $@/$# [full_proj_path]"}'

Run incrontab and paste the result into it:

    incrontab -e

Do this for each project you want to watch and lint.

## Make sure your editor doesn't move the file

Vim, by default, when you save a file, moves it to create a backup, and creates a new file. This means inotify (incron) is watching the old file.

Add this to your .vimrc:

    set nowritebackup 

## Test it

Open a python, javascript or css file in your favorite editor, and save it. 
You should see:

- If lint found an error: a modal popup.
- If lint found some warnings: A Gnome notification, and details written
directly on your desktop (root window).
- If lint is happy: Just a notification. Good job.

## Customise it

All this script does is wire together a number of other tools. It's just bash script, and there are lots of comments. Hopefully you can customise it for your particular situation.

## Contribute

I'd love to add linters for other languages, so if you do that locally, please send me a patch. Thanks!

