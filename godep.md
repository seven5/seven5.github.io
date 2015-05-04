---
title: Godep detour
layout: detour
---

# Godeps detour

When developing the tutorial or experimenting with different software versions,
the easiest way is to temporarily set your GOPATH to `TUTROOT/external` and
use `go get` to download software into that "staging area":

{% highlight bash %}
$ cd 
$ export GOPATH=$TUTROOT/external
$ go get github.com/tools/godep
$ go get github.com/gopherjs/gopherjs
$ # etc. for whatever software you want to install into the staging area
$ ls TUTROOT/external/src/github.com
{% endhighlight %}

Once you have the set of software installed in there that you want to experiment
with, typically including a version of godep itself, you can do this:

{% highlight bash %}
$ cd 
$ cd TUTROOT/src/tutorial
$ rm -rf Godeps
$ PATH=$PATH:TUTROOT/external/bin godep save ./...
{% endhighlight %}

This procedure _copies_ the code installed in `TUTROOT/external/src` directory
into the Godeps folder.  At this point, you can `source enable-tutorial`
and your environment will return to normal but with your new/different
vendored dependencies.

This is made more complicated by three tools that have to be added manually
to the project to create a working tutorial.  These are the packages 
`github.com/gopherjs/gopherjs`, its `jquery` subpackage, and `github.com/tools/godep`.
There are some helpful lines, usually commented out, at the top of `fresno/main.go` 
that can help you force the `godep save` command to put these packages into the
Godeps  directory.  You can't actually compile the project with these lines
uncommented, but it is enough to fool `godep save`.

[Return To Main Tutorial](tutorial.html#godep)
