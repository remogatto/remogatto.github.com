---
layout: post
title: "Gospecify: Basic setup of the project's structure"
---
# Gospecify: Basic setup of the project's structure

During the last two days I played a bit with
[GoSpecify](http://github.com/stesla/gospecify) by Samuel Tesla, a BDD
framework for the Go Language. I'm coming from 4 years of intense Ruby
development and, you know, I could not live without a good BDD
framework. After watching Samuel [Google
TechTalk](http://www.youtube.com/watch?v=4RSJg_1TdpA) I decided to
explore BDD under Go. The first problem I faced and solved was with
basic files and folders setup. In this post I'll describe how I
managed the issue using project structure and makefiles taken from
gospecify project itself.

First of all, let's create the project folder

<pre class="terminal">
mkdir fib
</pre>

Yeah, you guessed right, we'll deal with yet another [Fibonacci
sequence](http://en.wikipedia.org/wiki/Fibonacci_number)
implementation ;)

Now, move in fib folder and create a basic project structure

<pre class="terminal">
cd fib
mkdir src
mkdir spec
</pre>

As you guess src will contain source files and spec will contain the
specs. Now let's go creating a Makefile in the project base
path. Create an empty Makefile using your favourite text editor and
copy/paste the code below:

{% highlight basemake %}
GOBIN=$(HOME)/bin
SPECIFY=specify
 
all: test
 
clean:
	cd src; make clean
 
format:
	cd src; gofmt -w *.go
	cd spec; gofmt -w *.go

test: package
      cd spec; $(SPECIFY) *.go
 
package:
	cd src; make package
{% endhighlight %}

These are the up-level's rules with which we will interact. Names are
self-explanatory however I'll describe the corresponding actions
below:

<pre class="terminal">
make clean
    Clean up project folder removing intermediate and backup files
make test
    Run the specs
make format
    Format the source files using gofmt
make package
    Build the package
</pre>

As you can see, some of the rules above assume there is another
Makefile inside src/ folder. Thus, move on src/ and create the
following Makefile:

{% highlight basemake %}
PACKAGE=fib
TESTPROG=test_$(PACKAGE)
 
include $(GOROOT)/src/Make.$(GOARCH)
export GC
export LD
export O
 
SRC=$(wildcard *.go)
 
all: test
 
clean:
	rm -rf *.[a68] $(TESTPROG) _test *~
 
test: testpackage
package: $(PACKAGE).a

$(PACKAGE).a: $(PACKAGE).$O
	      gopack grc $@ $(PACKAGE).$O
 
$(PACKAGE).$O: $(SRC)
	       $(GC) -o $@ $(SRC)
{% endhighlight %}

Note that we have to set the PACKAGE variable with the name of the
package (fib in this case). Also, note that the testpackage rule in
the Makefile above will create a package named test$(PACKAGE).a. So,
in our case, we will get src/testfib.a. We will import this package in
our test code to actually run the specs.

At this point, the basic project structure and makefiles are
done. Now, we can populate src/ folder with an empty source file (with
just the basic package clause within):

<pre class="terminal">
cd src
echo "package fib" > fib.go
</pre>

Now, following BDD convention (test-first approach), let's write our
test. Enter in spec/ folder and create spec/fib_spec.go with the code
below:

{% highlight go %}
package main
 
import . "specify"
import t "../src/fib"
 
func init() {
        Describe("Fib", func() {
                It("should return Fibonacci's next number", func(e Example) {
                        fib := new(t.Fib)
                        fib.N0, fib.N1 = 0, 1
                        e.Value(fib.Next()).Should(Be(1))
                        e.Value(fib.Next()).Should(Be(1))
                        e.Value(fib.Next()).Should(Be(2))
                        e.Value(fib.Next()).Should(Be(3))
                        e.Value(fib.Next()).Should(Be(5))
                })
 
        })
}
{% endhighlight %}

The spec above tell us how we expect the system behaves before
actually implement it. In particular, we setup two seed values N0 and
N1 and then we query fib about the next value in the sequence using
fib.Next() method. For the first five calls we expect the sequence: 1,
1, 2, 3, 5.

If we try to run the specs at this point we get an error of course (we
haven't an implementation yet):

<pre class="terminal">
$ make
...
fib_spec.go:9: undefined: testfib.Fib
...
</pre>

Open the text editor and paste in src/fib.go the code below:

{% highlight go %}
package fib

type Fib struct {
        N0, N1 int
}
 
func (state *Fib) Next() int {
        state.N0, state.N1 = state.N0 + state.N1, state.N0
        return state.N0
}
{% endhighlight %}

If you're new to Go, take your time reading the code. Also consider
that I'm still learning this language from the ground up so this could
not be the best idiomatic implementation (waiting for suggestions in
this case). BTW, for the purpose of this post, our code behave as we
expect. In fact, running the specs

<pre class="terminal">
$ make test
.
Passing: 1  Failing: 0  Pending: 0  Errors: 0
</pre>

they all pass.

Well, that's all for now! You can browse the source files used in this
post on
[github](http://github.com/remogatto/learning-go/tree/master/gospecify-setup/).
