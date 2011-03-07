---
layout: post
title: Is Go suitable for building DSL?
---
# Is Go suitable for building DSL?

Computer problems can be approached easier once they are expressed in
an effective way. There are modern and general-purpose languages like
Ruby and Python that are suitable for building a sort of internal
language that is specific for the problem domain: a Domain Specific
Language (DSL). A DSL tries to describe the problem with
expressivity. But of course, the mean of expressivity is relative to
the problem domain. In this blog post I'll explore Go's capability in
building DSL.

# Rake, a popular Ruby DSL

Rake, like make or ant, is a DSL for defining tasks.

{% highlight ruby %}
task :hello do  
  print "Hello "
end
	 
task :world => :hello do
  print "World!"
end
{% endhighlight %}

Tasks definitions above are written using Rake DSL. In particular,
they define two dependent tasks. They state that the :world task is
dependent from the :hello task. In other words, when the :world task
is executed, the :hello task will be executed first. This will result
in the string "Hello World!" printed on the console. The neat fact
about the code above is that it is perfectly legal ruby code. In
particular,

* task is a ruby method that accepts an hash argument
* :hello and :world are ruby symbols
* :hello => :world is a ruby hash
* the do ... end part is a ruby block

Since rake DSL uses valid ruby code to express itself we name it an
internal DSL.

# Makengo: an internal DSL in Go

When I approached to Go the first time I was intrigued by what is
stated in its homepage:

"It feels like a dynamic language but has the speed and safety of a
static language. It's a joy to use."

Belonging from years of experience with dynamic languages such as ruby
and enjoying its capability to build DSL, I was curious to see how
much the compiled and strongly typed nature of Go can be bended in
this sense. So, I started writing a small experimental project named
makengo.

Using makengo DSL, the previous tasks definitions become:

{% highlight go %}
Task("hello", func() {
    fmt.Println("Hello")
})
	 
Task("hello", func() {
    fmt.Println("World!")
}).DependsOn("world")
{% endhighlight %}

How does it look compared to the ruby version? First of all, the Go
version takes 117 chars against the 75 chars of the Ruby version. So,
it's ~56% more verbose. In particular

* There are too many parentheses
* String names are longer than symbol names (:hello vs "hello", :world vs "world")
* The block syntax is burdened by the func keyword
* In order to express the dependency we need the long named DependsOn function rather than the concise key => value hash syntax

I could shortening the DependsOn() function name and gain some chars
but what about expressivity? Rake DSL use Ruby's hash syntax to
express dependency but I can't find nothing so effective in Go.

# Go DSL capabilities

The retrospection on Makengo raises the main question of this post: is
Go suitable for building internal DSLs? Let's try answering, exploring
Go's DSL capability in more detail.

## Closures

Go has many features that help building DSL. The first thing that
comes to mind are anonymous functions that act as closures. In
makengo, I use anonymous an function to define what a task does.

{% highlight ruby %}
Task("A Task", func() {
    // Do something
})
{% endhighlight %}

Also, anonymous functions can take arguments in order to push values
in the block. For example:

{% highlight ruby %}
collection.Do(func(element interface{}) {
    fmt.Println(element)
})
{% endhighlight %}

Compared with C, this a great step towards expressivity. However,
compared with Ruby, Go's anonymous functions syntax appears to be a
lot more verbose. This is due to the necessity of a func keyword and
to the use of many parentheses. Also, considering that since Go is a
strongly typed language, anonymous function arguments need their own
type declaration.

## Method chaining

In Go, types can receive methods. In Makengo, this fact is exploited
to express dependencies among tasks. Actually, the Task function
returns a task object and task object can receive the DependsOn method
allowing for method chaining:


{% highlight ruby %}
Task("world", ...).DependsOn("hello")
{% endhighlight %}

## Dynamic reception and metaprogramming

Go lacks of a ruby-like method_missing method. In ruby, when an
unknown method is sent to an object the object responds executing
method_missing. This language feature is known as Dynamic
Reception. Dynamic Reception, combined with the metaprogramming magic
of ruby, allows for very readable DSL (taken from Machinist):

{% highlight ruby %}
User.blueprint do
  name
  email
end
{% endhighlight %}

Regarding metaprogramming, Go has an interesting reflect package that
allows the manipulation of objects with arbitrary types. This comes in
handy, for example, in variadic functions (see below).

## Literal maps

In Ruby, a literal map could be passed as argument to a function
allowing for more expressive function calls inside the DSL: view
source

{% highlight ruby %}
task { :world => :hello }
{% endhighlight %}

Moreover, Ruby allows you omit the delimiters for a literal map. So
you can shorten it to:

{% highlight ruby %}
task :world=> :hello
{% endhighlight %}

In Go, a literal map looks like this:

{% highlight ruby %}
map[string]string { "world":"hello" }
{% endhighlight %}

but there is no way to shorten it.

## Literal lists and varargs

Like maps, literal lists in Go have not optional delimiters so it's
not convenient to use them as function arguments inside DSL. BTW, Go
has good support for varargs that combined with types reflection,
types assertion and interfaces, are good substitutes for literal
lists. For example, in Makengo I can define tasks that depend on more
than a single task passing multiple arguments to DependsOn:

{% highlight ruby %}
Task("TaskA", ...).DependsOn("TaskB", "TaskC", "TaskD")
{% endhighlight %}

## A comparison table

Let's try summarize in a table what we discussed until this point.

<table class="fancy-table">
<tr><th>DSL-related features</th><th>Go</th><th>Ruby</th></tr>
<tr><td>Method chaining</td><td>Yes</td><td>Yes</td></tr>
<tr><td>Optional delimiters in maps</td><td>No</td><td>Yes</td></tr>
<tr><td>Optional delimiters in lists</td><td>No but you can use varargs</td><td>Yes</td></tr>
<tr><td>Metaprogramming</td><td>Limited</td><td>Yes</td></tr>
<tr><td>Dynamic reception</td><td>No</td><td>Yes</td></tr>
<tr><td>Short signature for blocks</td><td>No</td><td>Yes</td></tr>
</table>

# The answer

So what about the answer to the original question? Is Go suitable for
building internal DSL? The answer of course is ... it depends!

As a compiled language, Go can't compete with dynamic languages in
building complex and general purpose DSL. However, it's enough
flexible to build simple DSL. Moreover, as said at the beginning of
this post, the mean of expressivity is relative to the problem
domain. Go can be very expressive when dealing with problems that
involve network and concurrency issues.

Another road to explore is the implementation of a light VM in Go
exploiting some of its advantages against C: built-in concurrency and
GC. On top of this VM could live a set of dynamic languages more
suited to implement external DSLs. It seems that Eleanor McHugh is on
the right track with her GoLightly VM.

Go DSL capability need more investigation, of course. But a fact is
clear to me: I should not think at Go as a Panacea trying to force its
nature: Go it's not a scripting language. I should rather taking
advantage of its peculiarities in order to fully exploit all of its
expressivity.
