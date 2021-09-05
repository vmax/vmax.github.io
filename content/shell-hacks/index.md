---
title: Shell hacks
date: 2021-09-05
description: on becoming a 10x developer
---

Over years of using the shell on Unix-like systems, I've collected some tricks that dramatically
increase productivity as well as give me great joy ~~and street credibility among other developers~~
when I utilize them. Here they are, in no particular order:

# Aliases

[Alias](https://en.wikipedia.org/wiki/Alias_(command)) is a shorthand command that expands to a long one.
You can define as many of them as you want in your shell's rc file (`~/.bashrc`, `~/.zshrc` etc.).

* Finding and editing files quickly with the `e` alias:

```
alias e="fd --type=f | fzf --bind 'enter:execute(nvim {1})+abort' || true"
```

This requires [`fd`](https://github.com/sharkdp/fd) and [`fzf`](https://github.com/junegunn/fzf) to be installed.
`nvim` can be adjusted to the editor of your choosing - `nano` would be more beginner-friendly alternative.

* Interacting with clipboard on Linux:

```
alias pbcopy="xsel --clipboard --input"
alias pbpaste="xsel --clipboard --output"
```

On macOS these commands are available natively.

* Finding file size:

```
alias size="du -d1 -h"
```

* Uploading a file to share publicly:

[Transfer.sh](https://transfer.sh) is a great service that allows you to quickly share a file publicly:

```
$ curl --upload-file ./hello.txt https://transfer.sh/hello.txt
```

However, we can optimize it a step further by automatically copying the resulting public link into clipboard:

```
_transfer(){ if [ $# -eq 0 ];then echo "No arguments specified.\nUsage:\n transfer <file|directory>\n ... | transfer <file_name>">&2;return 1;fi;if tty -s;then file="$1";file_name=$(basename "$file");if [ ! -e "$file" ];then echo "$file: No such file or directory">&2;return 1;fi;if [ -d "$file" ];then file_name="$file_name.zip" ,;(cd "$file"&&zip -r -q - .)|curl --progress-bar --upload-file "-" "http://transfer.sh/$file_name"|tee /dev/null,;else cat "$file"|curl --progress-bar --upload-file "-" "http://transfer.sh/$file_name"|tee /dev/null;fi;else file_name=$1;curl --progress-bar --upload-file "-" "http://transfer.sh/$file_name"|tee /dev/null;fi;}

function transfer { _transfer "$*" | xsel --clipboard --input }
```

* Shorthands for `git` to copy latest commit's message or SHA1:

```
alias git-copy-last-commit-message="git log -1 --pretty=%B | tr -d '\n' | pbcopy"
alias git-copy-last-commit-sha="git rev-parse HEAD | tr -d '\n' | pbcopy"
```

* Removing all unused Docker images:

```
alias docker-images-gc="docker image ls --format='{{ .ID }}' | xargs docker image rm"
```

* Triggering CI with an empty commit:

```
alias t="git commit --allow-empty -m "Trigger CI" && git push"
```

* Switching between Java 8 and Java 11 (Ubuntu-only):

```
alias J8='sudo update-java-alternatives -s java-1.8.0-openjdk-amd64'
alias J11='sudo update-java-alternatives -s java-1.11.0-openjdk-amd64'
```

* Finding and replacing content in files:

Install [`ag`](https://github.com/ggreer/the_silver_searcher) and define `agr` function ("ag replace"):

```
function agr { ag -0 -l "$1" | AGR_FROM="$1" AGR_TO="$2" xargs -r0 perl -pi -e 's/$ENV{AGR_FROM}/$ENV{AGR_TO}/g'; }
```

Usage:

```
$ ag needle
$ agr oldtext newtext
```

# Keyboard shortcuts

Due to using [GNU Readline library](https://en.wikipedia.org/wiki/GNU_Readline), these keyboard shortcuts are commonly available:

| Shortcut    | Description |
| ----------- | ----------- |
| `Ctrl+A`    | Go to beginning of the line |
| `Ctrl+E`    | Go to end of the line |
| `Ctrl+R`    | Search backwards in history |
| `Ctrl+S`    | Search forwards in history |

A good developer is a **lazy** developer; instead of typing commands over and
over again, you should use history as much as possible.
