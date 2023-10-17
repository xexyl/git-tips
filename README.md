# git tips README.md

## Introduction

This is at this time (as of 17 October 2023) a rather small list of tips for
`git(1)` that I've either discovered on my own or have come across or have been
told by others. If you have any that you'd like to share feel free to open a
pull request and I'll consider merging it, making sure to give you credit. I
don't expect this to be useful to some but it is useful to me if nobody else.

### Disclaimer

I cannot consider myself a `git(1)` expert but I do have a reasonable amount of
experience using it and this is another reason the list is fairly short for now.

## Special mentions

A special mention where a number of things come from is my good friend [Landon
Curt Noll](http://isthe.com/chongo/) ([GitHub](https://github.com/lcn2),
[Mastodon](https://fosstodon.org/@landonnoll),
[Wikipedia](https://en.wikipedia.org/wiki/Landon_Curt_Noll)), the founder and
one of the [International Obfuscation C Code Contest](https://www.ioccc.org)
judges.

Below I will just refer to Landon and not link to him again.


## gitattributes

As I'm a C programmer I naturally like `diff(1)` output to be in C format for C
files so whether or not this is strictly necessary, I have simply:

```
*.c     diff=c
*.h     diff=c
*.cpp   diff=c
```

(Though I really, really, really do **NOT** like C++.)


## A powerful alternative to aliases

This is something I discovered the other day that's very useful as it allows one
to do much more than what `git(1)` aliases allow for.

**NOTE: this has higher precedence than git aliases!**

If you put a script (maybe program too?) in your path that is named in the form
of `git-foo` you can run it like a `git(1)` command. For instance if you have a
file `~/bin/git-foo` and it's executable (`chmod +x ~/bin/git-foo`) you can run:

```sh
git foo
```

and it will run the script.

## **Warning: this can run commands that are NOT git commands as long as it is run in a git repository!**

For instance the above file might be:

```sh
#!/usr/bin/env bash

ls
```

Running `git ls` will then run `ls` in a git repo and not as an alias.

Thus there is a potential danger of commands being injected if someone does
something nasty for instance if it's in `/usr/local/bin` and it's in your path!


### Alias scripts

These are some that I have made that I find useful though one can certainly name
them whatever they like.

#### List branches

The following script will list branches of the repository, skipping the branches
`master` and `main`:

```sh
#!/usr/bin/env bash

git branch | \
    tr -s ' '| \
    sed -e 's/^ //g' -e 's/ /\n/g' | \
    tr ' ' '\n'|grep -vE '^(master|main|\*)$'
```

If you want to see `master` or `main` as well you can remove the `grep -vE ..`
part. If you wish to filter another branch out you can add it to the `grep -vE`
command. For instance you might wish to filter out `foo`:

```sh
#!/usr/bin/env bash

git branch | \
    tr -s ' '| \
    sed -e 's/^ //g' -e 's/ /\n/g' | \
    tr ' ' '\n'|grep -vE '^(master|main|foo|\*)$'
```

though that's not been tested.

A fun exercise is to run (if you have more branches than master or main) the
script manually and also run it as a `git(1)` command and then see what happens.

#### Delete branches that are NOT master or main

**Warning: this will delete branches that you have not created!**

This script uses a function that consists of the `git-ls-branches` script shown
above. It is used as a function in the script but it might be possible to use
another script I just haven't tested it.

```sh
#!/usr/bin/env bash
git_ls_branches()
{
    git branch |tr -s ' '|sed -e 's/^ //g' -e 's/ /\n/g' | \
	tr ' ' '\n'|grep -vE '^(master|main|\*)$'
}

for i in $(git_ls_branches); do
    git branch -D "$i";
done
```

#### Summarise how many commits each committer has made, highest count first

This script is far from perfect and will not pass `shellcheck(1)` either but
it's useful for what I desire:


```sh
#!/usr/bin/env bash

for i in $(git shortlog -ns|cut -f2-); do
    echo "git shortlog | grep -E "^$i"";
done
```

## Aliases

Here are more basic aliases that are in the `.gitconfig` file in the `[alias]`
section. One shows that you can create functions and use them as well.

### Diff involving the most recent commit

This is from Landon and it goes like:

```git
# diff involving most recent commit
cdiff = diff @~ @
```

To have just the name of files modified, try:

```git
# diff involving most recent commit but name only
cdiff-nameonly = diff @~ @ --name-only
```

### List already committed files that are being tracked

This pair is a pair I came up with as often I have files that are not being
tracked but are nonetheless important or useful in some way.


```git
# list already committed files tracked from the current directory
tracking-cwd = ls-tree -r --name-only HEAD
# list full tree of already committed files tracked
tracking = ls-tree --full-tree -r --name-only HEAD 
```

The first one only shows that in the current working directory whereas the
second shows all files in the repo.

### Making `git co` work like `co` of older RCS software

This was suggested to me by Landon. It's not something I thought of resolving
but it has always bothered me that `git(1)` does not natively support it.

```git
co = checkout
```

### git log with graph decorations on the side

Suggested to me by Landon:

```git
# git log with graph decorations on the side
glog = log --graph --color --decorate
```

### List modified files under git control

This was also suggested to me by Landon:

```git
# list files modified
check = diff --name-only
```

### Information on a commit

Also from Landon:

```git
# information on a commit
cinfo = cat-file -p
```

Use like:

```sh
git cinfo <object>
```

### List files, allowing for globs as well

It bothered me that one has to use `ls-files` rather than just `ls` so I made an
alias:

```git
# list files, allowing for globs to find specific files
ls = ls-files
```

### Full check on the repository and fix dangling commits

These come from Landon as well and it shows how to use functions in `git(1)`
aliases.

```git
# full fsck of the repository
fullfsck = fsck --full
# fix dangling commits and do a full fsck of the repository
fullfix = "!f_fullfix() { \
    git reflog expire --expire=now --all && \
    git gc --prune=now && \
	git fsck --full; \
    }; \
f_fullfix"
```

