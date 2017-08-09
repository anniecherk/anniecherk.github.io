---
layout: post
title: "Command Line Cribsheet"
date: 2017-07-01
permalink: commandline-cribsheet
categories: computers
---

I've put together a list of command line utilities that are great but that I either sometimes forget about or forget how to call.

I'm missing lots of really awesome utilities for working with [networks](https://jvns.ca/blog/2017/04/01/slow-down-your-internet-with-tc/) and [kernels](http://www.brendangregg.com/blog/2014-09-11/perf-kernel-line-tracing.html) that are not on my list because I haven't had occasion to use them- but hope to someday!

# find
Find and grep are both super useful commands with a ton of different command line arguments. Going through the first few levels of [the Bandit OverTheWire Wargame](http://overthewire.org/wargames/bandit/bandit0.html) will demo/ motivate a lot of uses of `find`.

Even though find is really powerful I usually just use it like this:

```bash
find <some/dir> -type f # returns a list of all the files in the directory
find . -name "*.sh" # finds all .sh files
find . -iname "*.SH" # finds all .sh files while ignoring case
find . -type d -name "bl*" #finds all directories whose names start with bl
find . -not -iname "*.py" #finds all files that are *not* python files
```

You can find files based on ownership, group, permissions, latest access / modification time / date and file size. You can also find files that *don't* match the query and you can construct more complicated queries with the "or" operator `-o`.

Find can be chained together with other commands; for instance you can pass all the files you found as the input to scripts or log them to a file. You can also remove all the files that match a certain query, but chaining find & remove into a single command makes me nervous because there's no undo on rm (unless you [alias rm](https://apple.stackexchange.com/questions/17622/how-can-i-make-rm-move-files-to-the-trash-can)) so if I needed to remove all files that matched some query I would probably log them to a file first to verify they were the right files! 


# grep
Grep lets you search file contents (and file names!), and is about the first thing I turn to when I jump into a codebase.

```bash
# This is how I use grep 99% of the time
grep -r "char\[10\]" . # searches recursively for the string query
# ^ notice that we have to escape [ & ] because you can use regex in the query

# Searching for *filenames* with find
find . | grep "Ham*" # returns Hamlet.txt

# Possibly useful flags:
grep -n "sweet prince" Hamlet.txt # prints line number of query in the file
grep -i "Good night" Hamlet.txt # case insensitive search
grep -c "good night" Hamlet.txt # "count": reports # of lines that match query
grep -rl "ssh" ~/.ssh/ # Finds & returns only file names of files that contain ssh
find . | grep -v "\\.txt$" . # prints all filenames that don't end with .txt

# You can chain greps together to find files that contain query1 && query2
grep -rl "ssh" ~/.ssh/ | xargs grep -r "annie"
```

There are also flags for specifying how many lines before & after the matched query to display.

# screen
Screen lets you keep a process running even after you kill the shell that spawned it. I use screen for long running jobs on a remote server, or jobs that I want to keep running remotely even if my coffeeshop internet blips out or I close my laptop.

```bash
screen -S myNewScreen # create a new screen named myNewScreen

# start some long running process & background it
./longRunningJob &

screen -d myNewScreen # detach the screen
# you can now safely kill the shell without killing the process

# when you log back in, reattach screen:
screen -r -d myNewScreen # detaches the screen from any other shells

screen -list # to see the names of all screens
```

# alias
Alias lets you not mean what you say and not say what you mean.

Good for setting up shorthands for commands you can't quite remember- the example below is an alias I use to spin up a jekyll website.
```bash
alias website='bundle exec jekyll serve'
website # runs the command "bundle exec jekyll serve"
```
I usually want my aliases to persist across sessions, so I would add that "alias website=..." line to my `~/.alias` file, and then reload it by running `source ~/.alias`. (Or whatever config file you save aliases in; I'm not here to tell you how to live your life.)

Aliases are also good for setting up default flags, like if I always wanted to call `ls` with the `-l` flag, I could set up the alias:
```bash
alias ls='ls -l'
ls # this actually executes ls -l

# you can check what an alias is set to:
alias ls # returns: alias ls='ls -l'

# actually I just want ls to be ls:
unalias ls
ls # this executes just ls again
```
In fact, some linux distros ship with the alias: `alias ll=ls -al`.

I've also found aliases helpful when I often use a flag that I can't remember. I once wanted to set up an alias for a command of the form:

```bash
command arg1 --flagICanNeverRemember=arg2
```

The way I set it up was I added a "new function" to my `.bash_profile` like this:

```bash
commandWithFlag () { # create a new function
    arg1=$1; # $1 is the first arg in the arg list from the function call
    shift;   # this *removes* arg1 from the arg list
    command $arg1 --flagICanNeverRemember=$@; # $@ is the arg list
    # using $@ lets you pass more flags after supplying the argument
}
```
I can now call it like this:

```bash
commandWithFlag arg1 arg2
# or with more flags
commandWithFlag arg1 arg2 --anotherFlag=arg3
```
(Thanks to [James Keene](https://github.com/jamak) for explaining how to do this!)

# pushd / popd
Pushd is like `cd` except instead of just changing the directory, it pushes the current working directory onto a stack and then changes the directory. Then to get back to where you were before changing directories, you just use popd:

```bash
cd ~/some/obscure/and/long/path
pushd ~/another/long/but/unrelated/path
# pwd is now ~/another/long/but/unrelated/path
popd
# pwd is now ~/some/obscure/and/long/path
```

# ctags & cscope
If you're working in a giant c code base, you can instrument your code with `ctags` and then use your favorite editor to jump to definitions. Really really useful for tracing back what some strange data structure is.


You first instrument a codebase with ctags by running in the project root directory:
```bash
ctags -R .
```

Then open a file in {ctags compatible editor}, and enjoy! The keybinds will surely depend on your editor, but here are the bindings for vim.

When you want to jump to the definition of the thing under the cursor:
```bash
:tag
# or
CTRL + ]
```

and to jump back:

```bash
:pop
# or
CTRL + t
```

Another ctags tool is `cscope`. You can just run `cscope -R` in the root directory of the project, and this will open up an interface you can use to search for functions, function calls, symbols, text, assignments, and more!

Neither `ctags` nor `cscope` come pre-installed, but you can get them through your favorite package manager.

# pbcopy / pbpaste & xclip
Pbcopy / paste are mac-only commands that let you copy the contents of a file to your clipboard (or move the contents of your clipboard to a file). Particularly useful for copying ssh public keys.
```bash
# to copy buffer from file
pbcopy < ~/.ssh/id_rsa.pub
# to copy buffer from result of a process call
grep -r "HACK" . | pbcopy
# to file from buffer
pbpaste > Hamlet.txt # so that's where it came from!
```

You can set this up on linux using `xclip`, with the following aliases:

```bash
alias pbcopy='xclip -selection clipboard'
alias pbpaste='xclip -selection clipboard -o'
```

# htop
htop is like top, but more colorful/ easier to read.

```bash
htop # for all processes
htop -u <username> # for snooping on a user
htop -p <pid> # for snooping on a process
```

# free
Free shows how much memory is available/ used. Especially useful when you're wondering if you've eaten up all the memory on the server with your enormously memory-hungry job.

```bash
free -g # shows how many gigs are available
```

# wc
Wc is word count, which is funny because I usually use it to count *lines* not words.
```bash
wc -l Hamlet.txt
# In case you were wondering, there are 4462 lines in Hamlet.

# combine w/ find to count lines of code in a repo:
find . -iname "*.py" -exec cat {} \; | wc -l
```

# tee
Tee splits the output stream to go to *both* a file, and stdout. This is useful as a sanity-check for something that you're logging to file:
```bash
# calls "my_script" and logs the results,
# but also mirrors the results to stdout
./my_script.sh | tee script_results.log
```

# du
What do you mean I'm out of disk space?? What's taking up so much room?

```bash
# the -h flag prints the "human readable" version
du -h # how big are all the files in the current dir?
du -sh # cut to the chase - how big is the current directory?
du -h -d 1 # summary of all folders 1 deep
du -h <some/other/filepath>
```


# say
Get your computer to chat with you, compliment you or berate you! Surely a necessary part of any worthwhile AI.
```bash
say -v Fred oh no how did I get in here? ...and how do I get out?!

say -v '?' # to list all voices
```

# ping
Ping is probably useful for a lot of things, but I use it as a sanity-check when I've got network issues to see whether my packets are getting through:
```bash
ping www.google.com
# this will continue pinging google until you hit CTRL + C to kill it
# once you kill it, you'll get a nice summary:
--- www.google.com ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 45.756/49.170/53.077/3.009 ms
```

# who
In case you're wondering who you're sharing your machine with:
```bash
who # returns a list of users

# and for existential crises:
whoami # prints: annie. phew, I feel much better.
```

# rmtrash
I think rmtrash only works on mac; you can get it with brew. It's an alternative to `rm` which moves a file to the trash instead of irreversibly deleting it forever.


# nice & renice
You can't control your OS's scheduler, but you can give suggestions for how important certain processes by starting them with a niceness value.

```bash
# 19 is the lowest priority
nice -n 19 ./long_slow_job

# you need to be root to give jobs higher priority
# -20 is the highest priority
sudo nice -n -20 ./important_job  
```

You can update the priority of an already running job by renicing them:

```bash
# you have to renice by pid
renice -n 5 -p 1234

# you can find the pid of a process by name with:
ps ax | grep jekyll # finds the pid of the `jekyll` process
# returns:
#57036 s000  S      0:09.12 /usr/local/bin/jekyll serve
#57522 s000  S+     0:00.00 grep jekyll

# renice the jekyll serve process to make it more important:
sudo renice -n -20 -p 57036
```

# sort
Does what you think it does:
```bash
sort scrambled_alphabet.txt # prints alphabet to stdout
sort -r scrambled_alphabet.txt # reverse order: z, y, x, ...
sort -c scrambled_alphabet.txt # checks whether is sorted
# returns info about first unsorted line:
#  -> sort: scrambled_alphabet.txt:2: disorder: c
sort -c alphabet.txt # returns nothing because it *is* sorted
```

# cut
Cut lets you grab columns out of a text file. Useful for quickly hacking up some tabular data.

```bash
cut -c2 data.txt   # grabs 2nd column of the data
cut -c2-3 data.txt # grabs columns 2 & 3
cut -f1,4 -d " "   # grabs 1st & 4th fields, w/ delimiter "space"
```
