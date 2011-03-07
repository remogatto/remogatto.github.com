---
layout: post
title: "Installing Go: go simple!"
---
# Installing Go: go simple!

So finally I decided to run a blog. And yes, you're reading my first
blog post ever!

This afternoon I decided to give a whirl to Go Programming Language. I
don't know yet so much about it but I'm intrigued by some concepts
like goroutines and efficient GC. Moreover, I was amazed by the very
first impact with the language: the building process is foolproof (at
least on my Ubuntu box) so I decided to document the process as a
tribute to this neatness. It's an example of how things should be done
in computer programming. Moreover, it's a good starting point for my
first blog post ever, isn't it?

Assuming you're on a debian based system you need - first of all - to
install the toolchain to building the language:

<pre class="terminal">
sudo apt-get install bison gcc libc6-dev ed gawk make
</pre>

And, if you decide to clone the mercurial repository, you need mercurial of course!

<pre class="terminal">
sudo apt-get install mercurial
</pre>

Now that the prerequisites are satisfied let's configure our
build. Here's the neatness of the whole process. You setup a bunch of
environment variables and you're done! So for example, assuming that
we wish to:

* Clone Go repository in $HOME/go
* Build Go for a linux distribution
* Build Go for a i386 architecture

all we need is to export the following variables:

<pre class="terminal">
export GOROOT=$HOME/go
export GOARCH=386
export GOOS=linux
</pre>

And we can save them in .bashrc if we want a persistent configuration.

The installation process assumes that $HOME/bin folder exists and it
places binary files in it. Let's meet this last prerequisite:

<pre class="terminal">
mkdir $HOME/bin
</pre>

You can change binaries' folder setting the $GOBIN variable. In order
to run the executables, don't forget to update your $PATH variable
appending $HOME/bin.

<pre class="terminal">
export PATH=$PATH:$HOME/bin
</pre>

Well, now let's go cloning the repo and building the whole stuff:

<pre class="terminal">
hg clone -r release https://go.googlecode.com/hg/ $GOROOT
cd go/src
./all.bash
</pre>

At this point the installation process assumes that $HOME/bin folder
exists and it places binary files in it (you can change folder name
setting the $GOBIN variable). And finally let's test our build
starting the compiler:

<pre class="terminal">
$ 8g
flags:
-I DIR search for packages in DIR
-d print declarations
-e no limit on number of errors printed
-f print stack frame structure
-h panic on an error
-o file specify output file
-S print the assembly language
-w print the parse tree after typing
-x print lex tokens
</pre>

And that's all, no autogen.sh, no configure scripts to run! And is
even more beautiful because we are actually cross-compiling! That is,
we could produce builds for any other supported OS/ARCHs by simply
changing the values of the environment variables above! Can't wait to
test on my shining new beagleboard when it will arrive! :D

For further readings see:

* [Getting Started](http://golang.org/doc/install.html)
