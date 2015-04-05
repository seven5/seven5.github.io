---
title: Key Elements Of Seven5
layout: article
---

# Key Elements Of Seven5

Seven5 is a toolkit for constructing modern web applications.  Applications
written against Seven5 are written go---_entirely_ in go.  Seven5 provides
facilities for developing both the server and client portion of a web 
application as well as for exchanging go data structures between them. 

This article is intended to provide a light overview of the key ideas of
[Seven5](http://github.com/seven5/seven5).  More detail, including a fairly
complex real application, can be found in [the tutorial](tutorial.html).
The current release of Seven5 is named [cure](https://github.com/seven5/seven5/releases).
If you have questions about this article or the toolkit in general, you
can ask for help in the [google group](https://groups.google.com/forum/#!forum/seven5).
Seven5 is so named because the author began work on it while living in
Paris, France and all the postal codes for Paris begin with 75.


## Both sides of the wire

In this article, we will focus most of our attention on the _client_ side of
an application. In the past, one would have been forced to build this part
in [Javascript](https://www.destroyallsoftware.com/talks/wat). The 
primary reason for this focus on the client side is that go on the server 
side is far more common, and far more commonly understood.  The server portion
is what most people expect a "go-based web toolkit" to be. 

To build go programs that run in a browser, they must be compiled to Javascript.
For this, Seven5 expects you to use the wonderful [Gopherjs](http://www.gopherjs.org)
compiler.  "It just works" is the perhaps the highest compliment you can give to a 
compiler, and that certainly applies here.   It is a testament to its quality
that this article won't even mention Gopherjs further, in the same way that it 
will not mention that the go compilation tools on the server 
([6g](https://golang.org/cmd/6g/) and the like)--because _they just work_.

## Modern

We have referred to modern web applications previously.  To be more precise, we
mean applications which do *not* generate web content on the server but rather 
supply an API for clients--including a web browser--to obtain content from.  The
returned content is then formatted for display.  The "server side" of a Seven5 
app has two basic tasks: Respond to requests for static, unchanging files and 
respond to an API of the developer's choice.  Seven5 expects that API to be 
[RESTful](http://en.wikipedia.org/wiki/Representational_state_transfer), although
this is not  because REST is the best possible choice, but rather because REST is 
simple, common, and well-known. 

## Ajax

If the server's job is to expose a REST API, the client's job is to consume
that API and present results to the user in a browser.  In Seven5, the client
application can use the function AjaxGet to fetch some data. In this example
we are retrieving a slice of `PaymentMethod` structures from the server:

{% highlight go %}
func (p myPage) getPaymentMethods() {
	var methods *[]*wire.PaymentMethod
	content, errChan := s5.AjaxGet(methods, "/rest/paymentmethods")
	go func() {
		select {
		case raw := <-content:
			methods := raw.(*[]*wire.PaymentMethod)
			processSomeMethods(*methods)
		case err := <-errChan:
			print("unable to get information about payments: ", err.StatusCode, err.Message)
		}
	}()
}

{% endhighlight %}

Two conventions about Seven5 can be seen in this example.  First, Seven5 is
imported into a go program as `s5`.  For the server side of an application
this corresponds to `github.com/seven5/seven5` but for a client program, as
above, this corresponds to the package `github.com/seven5/seven5/client`. 
Second, the type being exchanged `PaymentMethod` is part of a package called
"wire".  Wire types are those that are exchanged between the client and server.
A package that includes wire types is compiled _by both the client and server_ 
since it refers to the structures being exchanged between them.  There is no 
way for the client and server to get "out of sync". 

The call to `s5.AjaxGet` returns two channels, one for content in the success
case and one for the error case.  Only one of these channels will receive data.
So it is appropriate to call `select` on these two channels to wait until the
content has been received from the server.  Because this can take an unknown
amount of time--browsers will typically try for 60 seconds to reach a server
that is down--we wrap the select in a goroutine so as not to "lock up" the
user's browser while we wait for the server to provide this data.  It is natural
in go to think of asynchronous calls to the server as something that will produce
a value on a channel when it completes.

## HTML Generation

Once data is received by the client, such as in our example above, it has
to be converted into HTML that can be rendered by the browser.  Typically,
the "framing" of a web page is static content---a simple HTML file---and
the dynamic portion is added to the page based on results returned from an
Ajax call.  Seven5 provides a tree-building library for generating 
[DOM](http://en.wikipedia.org/wiki/Document_Object_Model) subtrees that can
be attached to a page.

This is a trivial example of a code snippet that builds a small DOM tree.
This tree has a `div` HTML element with two children that also `div` elements.
Each child `div` has a `span` element child, and the text to display in these
`span` elements is a fixed string.

{% highlight go %}

s5.DIV(
	s5.DIV(
		s5.SPAN(
			s5.Text("foo"),
		),
		s5.SPAN(
			s5.Text("bar"),
		),
	),
)

{% endhighlight  %}

The snippets in this section use formatting to make the tree structure easier 
to see.  This formatting is allowed by [gofmt](https://golang.org/cmd/gofmt/).

You can easily add some 
[CSS](http://en.wikipedia.org/wiki/Cascading_Style_Sheets) classes to make 
your HTML tree look nicer when it appears on screen. CSS classes, and
many other HTML-related entities, are modelled as types in Seven5.

{% highlight go %}

var (
	row = s5.NewCssClass("row")
	offset1 = s5.NewCssClass("col-sm-offset-1")
	col10 = s5.NewCssClass("col-sm-10")
)

...

s5.DIV(
	s5.Class(row)
	s5.DIV(
		s5.Class(offset1)
		s5.Class(col10)
		s5.SPAN(
			s5.Text("foo"),
		),
		s5.SPAN(
			s5.Text("bar"),
		),
	),
)

{% endhighlight  %}

You can even add event handlers directly in the tree building code, and we will
show this is in the next example.  This is used far less commonly in Seven5 than 
most web toolkits, for reasons that will  be explained in the next section, but 
this example  prints out a message when the "foo" on screen is clicked.  

Although Seven5's client package is built on top of 
[JQuery](http://jquery.com) internally, this is not exposed
frequently, as Seven5 provides its own abstractions that are less error-prone
that the JQuery mechanisms.  One place this is exposed, however, is the `jquery.Event`
object that is passed to the handler of a click event.

{% highlight go %}

var (
	row = s5.NewCssClass("row")
	offset1 = s5.NewCssClass("col-sm-offset-1")
	col10 = s5.NewCssClass("col-sm-10")
)

...

s5.DIV(
	s5.Class(row)
	s5.DIV(
		s5.Class(offset1)
		s5.Class(col10)
		s5.SPAN(
			s5.Text("foo"),
			s5.Event(s5.CLICK, func(evt jquery.Event) {
				print("clicked foo")
			}),
		),
		s5.SPAN(
			s5.Text("bar"),
		),
	),
)

{% endhighlight  %}

## Constraints and Attributes

Seven5 uses a technique called "constraints" to make building the user interface
of a web application easier.  A _constraint_ is simply a function that computes
a result.  The values that a constraint operates on--its parameters--and 
produces are called _attributes_.   These correspond directly to go's
functions and variables.

Let's define a structure that has a few attributes so we can see how this will
work:

{% highlight go %}
type lightSwitches struct {
	s1     s5.BooleanAttribute
	s2     s5.BooleanAttribute
	output s5.StringAttribute
}

{% endhighlight  %}

This structure definition uses `s5.BooleanAttribute` and `s5.StringAttribute` 
instead of go's builtin `bool` and `string` because Seven5 does some extra 
bookkeeping around each attribute.  Let's define a constraint, which is
just a function:

{% highlight go %}
func eitherSwitchIsOn(raw []s5.Equaler) s5.Equaler {
	s1Status := raw[0].(s5.BoolEqualer).B)
	s2Status := raw[1].(s5.BoolEqualer).B)
	if s1Status || s2Status {
		return s5.StringEqualer{S:"on"}
	}
	return s5.StringEqualer{S: "off"} 
}
{% endhighlight  %}

The types here are not static as one would like, but it should be clear that
the parameters passed to `eitherSwitchIsOn` are two booleans and it returns
the string "on" if either one of these booleans is true, otherwise it returns
"off".  "equalers" in the above example such as `s5.BoolEqualer` and `s5.StringEqualer`
represent the _value_ of an attribute.

Let's attach the constraint now:

{% highlight go %}
	//ls is an instance of the type lightSwitches

	ls.output.Attach(
		s5.NewSimpleConstraint(
			eitherSwitchIsOn,
			ls.s1,
			ls.s2))

{% endhighlight  %}

This snippet bears scrutiny.  This attaches the constraint function we've written
above `eitherSwitchIsOn` to the string field `output`.  The inputs to the
function are the two other boolean fields `s1` and `s2`.  Once attached,
Seven5 guarantees that this constraint is always met. 

> For the curious, the algorithm used to insure constraint evaluation is
> both correct, and close to minimal is [eval_vite](ftp://ftp.cc.gatech.edu/pub/gvu/tr/1993/93-15.pdf) from 93.

In an of itself, this ability to have functions of variables, even ones that are
maintained automatically, would be of little value.  The big win comes from
the ability to *connect* attributes and constraints to the DOM that is 
generated by a web applications.  Let's connect our switches and the output
to the DOM:


{% highlight go %}

	//onClass is a css class that changes the display for "on"
	//ls is an instance of the type lightSwitches

s5.DIV(
	s5.Class(row)
	s5.DIV(
		s5.Class(offset1)
		s5.Class(col10)
		s5.SPAN(
			s5.CssExistence(onClass, ls.s1),
			s5.Text("switch1"),
			s5.Event(s5.CLICK, func(evt jquery.Event) {
				ls.s1.Set(!ls.s1.Get())
			}),
		),
		s5.SPAN(
			s5.CssExistence(onClass, ls.s2),
			s5.Text("switch2"),
			s5.Event(s5.CLICK, func(evt jquery.Event) {
				ls.s2.Set(!ls.s2.Get())
			}),
		),
		s5.SPAN(
			s5.TextEqual(ls.output)
		),
	),
)


{% endhighlight  %}

In this example, we have added three additional constraints to the attributes
in the `lightSwitches` struct.  Two of these are to constrain the presence 
or absence of the CSS class `onClass` on the DOM elements to the appropriate
span elements for switches s1 and s2.  Seven5 will guarantee that if 
`ls.s1` is true, the CSS class will be attached to the first span and it will not
be present if the attribute is false.  This can provide feedback to the user
about the state--such as perhaps changing the color, background, or any other
attribute that can be manipulated through style sheets.

The other constraint is that the text of the third span will always be the
same as the value of the field `ls.output`.   Since this is computed by
the constraint we wrote above, value displayed is always the logical
or of the two switches.

Finally, we have added two small event handlers, one for each "switch".  When
the appropriate "switch" text is clicked, the value of the boolean `s1` or
`s2` will get inverted.  This, naturally, cascades through the function 
`eitherSwitchIsOn` that then updates the attribute `output`, that 
causes the constraint on the last span's text content to be updated appropriately.
Also, the constraint that chooses to add or remove the CSS class `onClass` will 
update the display for the particular switch span that is clicked on.
 
This type of event  handler is common is Seven5: once you have expressed your
display as a set of constraints, the event handler's job is simply to update
the state, Seven5 takes care of an necessary screen updates and is careful to
_not_ update things that are not affected by the change in the data. 
Constraints are not a free lunch, they require work from you 
in structuring your application.  They require you to think 
carefully about the inputs and outputs of your user interface, and 
require clear articulation of the processing to be done.
This articulation must be done so that the processing of inputs to
outputs can be encoded in constraint functions.  Our experience has
shown that the effort required to structure a UI with constraints is
easily outweighed by the benefits gained in cleaner UI code.

# Summary

In this short article, we've touched on three things that make building a 
modern web application easier when you use Seven5. First, the ability to
use the familiar go abstraction of channels to handle asynchronous
connections from a web browser to the server.  Second, Seven5's tree
building utilities that make it convenient to programmatically construct
trees of DOM elements that reflect the semantics of your app.  Finally,
we discussed Seven5's use constraints to make the connections between
application data structures and the user interface shown in the browser.


