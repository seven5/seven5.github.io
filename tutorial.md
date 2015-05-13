---
title: Setting Up For Seven5 Development
layout: tutorial
---

<a name="start"></a>

# Fresno: The Seven5 Tutorial

This tutorial builds a blog engine called Fresno. The world does not need
more blog engines; Seven5 needs a tutorial based around an application
that has the minimal amount of application-specific things for you to learn.

> You can ask questions about the tutorial in the 
[Seven5 Google Group](https://groups.google.com/forum/#!forum/seven5).

## Typography in this document

This document writes commands that you type like this:

{% highlight bash %}
$ ls -l /
{% endhighlight %}

The dollar sign is used to indicate you should type the command at the
shell and expected output is placed immediately afterward.

We use the same "boxed" typography for code samples that are taken from
go code in the tutorial.

In cases where we are referring to a file or directory that is part 
of the tutorial we will write it like this `somefile.go`.  We also will 
indicate environment variables the same way (`PAGER`).

Meta-information, asides, and commentary about the tutorial itself is written like
this:

> We would love to have somebody test out this tutorial on windows
> and provide a pull request with the necessary changes to get it working.

## Prerequisites

This document assumes you understand 

* go 
* the basics of the web (http, html, css, a bit about the DOM)
* the basics of git
* a tiny bit of postgres and sql

If you know how to type "make" when necessary, that's a plus.  Some experience 
with curl is helpful, but not required.

Further, this tutorial assumes you are developing on a Mac using OSX or 
a similar-enough Linux installation. 

You will need to have a version of go installed that is at least 1.4. On
OSX the recommend way to do this is via brew.

The postgres version doesn't matter too much, probably anything from 9.1
to 9.4+ will work fine.  This tutorial was developed on postgres 9.3.

Any version of curl and make should be fine.

<a name="setup"></a>

# Setting Up For Seven5 Development

You need to layout the source code in a particular way to make various
go-related tools happy. Create a directory structure for this tutorial
like this:
{% highlight bash %}
$ cd
$ mkdir tutroot
$ cd tutroot
$ mkdir -p external/src
$ mkdir src
{% endhighlight %}

This document will refer to the directory "tutroot" created first above as
`$TUTROOT`.

Clone the tutorial repository:

{% highlight bash %}
$ cd
$ cd tutroot/src
$ git clone git@github.com:seven5/tutorial.git 
$ cd tutorial
$ git checkout tutorial
$ cat enable-tutorial
{% endhighlight %}

and examine the output of the last command.  That is the _enable script_
and you should put in this file any "local" setup that you need to do
have the this tutorial work, but also to not break other projects you 
have on your system. 

Once you are satisfied with the content of the enable script, you should
source it into your environment (not execute it):

{% highlight bash %}
$ source enable-tutorial
$ go version
go version go1.4.1 darwin/amd64
{% endhighlight %}

The output of the last command should be version 1.4+ and appropriate
for your operating system and processor architecture.

<a name="godep"></a>

## Godep and Seven5

[Godep](http://github.com/tools/godep) is an important, but tricky to use, 
tool.  If you look in the tutorial you just checked out, you'll notice that
there is a directory `$TUTROOT/src/tutorial/Godeps` that has a great many
go source files in it.  The contents are all the dependencies that are needed to
build and run the lessons in this tutorial.  These are, as intended by the
go creators, committed to the source code repository with the tutorial so
you can be sure things will "just work".

> If you are deeply interested in Godep, you can veer off into the 
[Godep detour](godep.html).

### Bootstrapping godep itself

Once and only once, you'll need to build the godep tool itself:

{% highlight bash %}
$ which godep
$ cd $TUTROOT/src/tutorial
$ GOPATH=$PWD/Godeps/_workspace go install github.com/tools/godep
$ which godep
TUTROOT/src/tutorial/Godeps/_workspace/bin/godep
$ echo $PATH
{% endhighlight %}

If you look at the last command's output, you'll notice that we have carefully
added the directory where godep was found to the PATH.  This is so we can 
pull tools from the vendored sources. 

### Installing gopherjs and using godep

Continuing the previous example now that we have a copy of godep ready:

{% highlight bash %}
$ which gopherjs
$ godep go install github.com/gopherjs/gopherjs
$ which gopherjs
TUTROOT/src/tutorial/Godeps/_workspace/bin/gopherjs
{% endhighlight %}

The command that built our copy of [gopherjs](http://www.gopherjs.org) is
worth understanding.  This command "wraps" the go command (arguments 2 - 4)
with special arguments that ensure that you are using precisely two sets
of source files:

1. Things in your `GOPATH`, set by the enable script to `TUTROOT`, so
source code in `TUTROOT/src`.
2. Things in the `TUTROOT/src/Godeps/_workspace` directory. 

You'll want to *always* use the "godep go" command to build things in this
tutorial.  This insures you are always getting the source code that is known to 
work with the tutorial.

<a name="simple-server"></a>

# Building The Fresno Server

For this lesson and all the ones following, we'll assume that know
what `TUTROOT` is on your system and that you have sourced the 
`enable-tutorial` script into your shell environment.

## Environment variables

All configuration of Seven5 applications is done through environment
variables.  This makes them [twelve factorish](http://12factor.net) and
easy to deploy on heroku. 

> If you are deeply interested in heroku you may want to take the
> [heroku detour](heroku.html).

## Build and run the application locally

{% highlight bash %}
$ cd $TUTROOT/src/tutorial
$ godep go install tutorial/fresno
$ fresno
2015/03/15 18:47:39 DATABASE_URL found, connecting to postgres://iansmith@localhost:5432/fresno
2015/03/15 18:47:39 [SERVE] (IsTest=true) waiting on :5000
{% endhighlight %}

You can try going to "http://localhost:5000/" with your browser, but you wont
see much because the server wants to connect to a database and you probably
don't have one ready yet.

<a name="database"></a>

# Add A Database

Most realistic applications need access to reliable relational store.
Fresno is no exception, so we'll go ahead and set this up now.  You may
have noticed previously that our enable script sets a DATABASE_URL
environment variable.

## Postgres

You'll need to have a copy of postgres running on your local system
for development.  On a mac, you can do this `brew install postgres`
and then follow its directions about how to run the database.  On a
linux system, use your package manager (yum, apt-get, pacman, or similar)
to install the postgres server. 

This document was written with postgres version 9.3, but other versions in the
9.x series will likely work.

## Local database setup

On your local system you'll need to create the database "fresno"

{% highlight bash %}
$ createdb fresno
{% endhighlight %}

This is a reasonable sanity check that your database server is up and
your settings in the enable script are correct.  If everything is
working properly, you'll receive no output from this command.

<a name="psql"></a>

#### PSQL

If you are familar with SQL, you can get an SQL prompt with
{% highlight bash %}
$ psql $DATABASE_URL
{% endhighlight %}
on your local system or for the remote database on heroku:

<a name="migrations"></a>

# Migrations

With a web server, a database, and all our dependencies captured in our
git repository we are now ready to start doing some real development.

## Database and migrations
Seven5 uses [qbs](https://gowalker.org/github.com/coocood/qbs) to provide
a thin layer of abstraction for database querying.  However, QBS' migration
support is not sufficient for real applications--for example, it does not
support changing column names. So Seven5 provides a thin shim to allow
migrations to be written in SQL + go.

We'll use a migration here to initialize our database with the tables we
want to use as well as some test data for our app to serve up.  When using
Seven5's support for migrations, you should create a new binary which 
will be called "migrate" because it's source is in the `migrate` directory.

## How migrations work

You can see the example migrations in `migrate/main.go`. 
These migrations are just SQL statements wrapped in go.  The table at the 
top controls the behavior of the resulting program:

{% highlight go %}
var defn = migrate.Definitions{
	Up: map[int]migrate.MigrationFunc{
		1: oneUp,
		2: twoUp,
	},
	Down: map[int]migrate.MigrationFunc{
		1: oneDown,
		2: twoDown,
	},
}
{% endhighlight %}

This table is passed through to 
[migrate.Main](https://gowalker.org/github.com/seven5/seven5/migrate#Main)
which is supplied by Seven5 as an entry point for migration programs.

Each migration should create/modify tables or data in the database and
can assume that if a given migration is run, that all previous migrations
in the correct order have been run successfully.  Any migration function
that has a problem should return an error and the transaction will be
rolled back and the database left unchanged. 

Probably the simplest migration is the migration from state 1 to state 0,
which is the "oneDown" function in this program:

{% highlight go %}
func oneDown(tx *sql.Tx) error {
	//bc of foreign keys, order of these drops is siginficant
	drops := []string{
		"DROP TABLE post",
		"DROP TABLE user_record",
	}
	for _, drop := range drops {
		_, err := tx.Exec(drop)
		if err != nil {
			return err
		}
	}
	return nil
}
{% endhighlight %}

If you look at the first up migration, you will notice that the "oneUp" migration
creates two sample users and sample posts.  This is handy for running tests
because you can just assume that if the database is there, these users
and posts are present.

<a name="caveat-data-type"></a>

### Caveat (Qbs data type matching)

When create a table in SQL, you have to create it with the structure
(data types and names) that will mesh with Qbs. Generally, this fairly
straightforward as the names are converted from camel case in go to
snake case for postgres.  However, you may need to experiment with
the column types to make sure that the structures in go mate correctly
with them. This is less burdensome than you'd expect because the set of 
types that you can express in the SQL tables is limited.

The next lesson will discuss what structures in go correspond to the
tables created.

## Building and running the migrations locally

You can build and run the migration application like this:

{% highlight bash %}
$ godep go install tutorial/migrate
$ migrate --up
[migrator] attempting migration UP 001
[migrator] attempting migration UP 002
002 UP migrations performed
{% endhighlight %}

You may find it interesting to use "psql" (see [previous lesson](#psql)) 
to look at what's in the database after you this command. 

The reverse migration also works:

{% highlight bash %}
$ godep go install tutorial/migrate  #you should have done this above
$ migrate --down
[migrator] attempting migration DOWN 002
[migrator] attempting migration DOWN 001
002 DOWN migrations performed
$ migrate --down
at earliest migration, nothing to do
$ migrate status
current migration number is 000
{% endhighlight %}

Again, you may find it interesting to look inside the database at the result
of doing "migrate --down".   There are options you can pass to the 
migrate program to run a specific number of up or down migrations with 
the "--step" flag.

If you have run the down migrations, you'll have no tables in your database
so you'll want to run the up migration to restore the tables and test data
for future lessons.

<a name="simple-rest"></a>

# Serve Up Some Data Through A REST API

We've got a web server and a database that has some content in it, so 
let's try to access the post table to see what posts are available.

## Preparation for this lesson

As above, make sure you have the databse running, run the migrations (it
doesn't hurt to run them again), build the fresno server, and start it running.

## Getting some data

In another shell, use [curl](http://curl.haxx.se) to test out the REST API:

{% highlight bash %}
$ curl http://localhost:5000/rest/post/1
{
 "Id": 1,
 "Title": "first post!",
 "Updated": "2015-03-21T07:23:19.617131-07:00",
 "Created": "2015-03-21T07:23:19.617131-07:00",
 "Text": "",
 "TextShort": "This is the first post on the site!",
 "AuthorUdid": "df12ba96-71c7-436d-b8f6-2d157d5f8ff1",
 "Author": {
  "UserUdid": "df12ba96-71c7-436d-b8f6-2d157d5f8ff1",
  "FirstName": "Joe",
  "LastName": "Smith",
  "EmailAddr": "joe@example.com",
  "Password": "",
  "Admin": false
 }
}
$ curl http://localhost:5000/rest/userrecord/df12ba96-71c7-436d-b8f6-2d157d5f8ff1
Not authorized (FIND, UDID)
{% endhighlight bash %}

There are two types of resources in this program, posts and users.  As you
can see from the above, you can read the posts via the REST API but you don't
have the authentication to read the information about Joe Smith. 
If you are curious, you may find it interesting to try changing the
curl command to "curl -v" so you can see the details of the error
messages, headers, etc.


## Resources

[Resources](http://restful-api-design.readthedocs.org/en/latest/resources.html) 
are the building blocks of 
[RESTful APIs](http://restful-api-design.readthedocs.org/en/latest/). It is
customary to name your resource implementations in Seven5 applications
as "FooResource" in the package `resource` if this provides the API implementation of the noun "foo". You can see the implementations of
"UserRecordResource" and "PostResource" in the files `resource/user_record.go`
and `resource/post.go`, respectively.  Note that these are in different
package typically from your main (`fresno/main.go`) so they must be
capitalized.

The objects to be exchanged over the wire between client and sever, 
referred to as _wire types_ in Seven5, are represented as go structures
in the `shared` package.  The shared package is so named because both the
client and server compile it.   The combination of the REST resources and
the structures in the shared package define the programmatic API of your
"back end."

Because our application is simple, we are *also*
going to use these same structures as the data to be represented in
the database.  This double duty is nice, because it insures that the 
values exchanged over the wire are "in sync" with the underlying data
model, but more complex applications typically need to separate the
storage layer (represented by structures in the database)
and the wire representation (that is exchanged between client and server). 

## The wire types

The wire types are the "nouns" in the sense of RESTful API design. 
These are are in `shared/post.go` and 
`shared/user_record.go` and coded as uppercase, singular nouns ("Post").
Because they are in a separate package, and because they must be
serializable with "encoding/json", the fields must be uppercase also.

Here are the two wire types:
{% highlight go %}

type UserRecord struct {
	UserUdid  string `qbs:"pk"`
	FirstName string
	LastName  string
	EmailAddr string
	Password  string
	Disabled  bool
	Admin     bool
}


type Post struct {
	Id         int64 `qbs:"pk"`
	Title      string
	Updated    time.Time
	Created    time.Time
	Text       string
	AuthorUdid string `qbs:"fk:Author"`
	Author     *UserRecord
}
{% endhighlight %}

As we touched on in the [previous lesson](#caveat-data-type) the fields
in the structures "UserRecord" and "Post" must match up to the columns
in the database tables "user_record" and "post".  The primary key field
must be called out to Qbs if the "pk" is a string, and it is good practice
to do it in any case, even though Qbs would default to choosing the 
int64 field Id for Post. 

You'll notice that we do a join on the post's author to a user record.
This is done by default by Qbs at retrieval-time and we've just accepted
that default here and told Qbs where to put the joined record.

### Testing REST on heroku

If you have pushed your code to heroku, you do the same curl command
to your heroku instance (substituting your application name):

{% highlight bash %}
$ curl https://damp-sierra-7161.herokuapp.com/rest/post/1
[json output]
{% endhighlight %}

## Understanding rest resource "types"

Seven5 exposes two different means of addressing a rest resource.  The first
is the "traditional" way with a positive integer, as in the example above
with the "Post" wire type.  This is the normal way of accessing resources in
a RESTful API.  There are some situations where the "pretty" urls such as
"/rest/post/2" is not desirable because there is a security consideration
or you simply do not wish these to be "guessable".   For this situation,
you can use a 
[UDID](http://en.wikipedia.org/wiki/Universally_unique_identifier) written
in the standard hex format, such as "df12ba96-71c7-436d-b8f6-2d157d5f8ff1".

If you look in the setup of the fresno application now (`fresno/main.go`),
you can see where the connection is made between a wire type and a 
resource implementation that implements methods (logically) for that 
wire type:

{% highlight go %}
	result.base.ResourceSeparateUdid("userrecord",
		&shared.UserRecord{},
		nil, //index
		s5.QbsWrapFindUdid(&resource.UserRecordResource{}, store),
		s5.QbsWrapPost(&resource.UserRecordResource{}, store), //post
		nil, //put
		nil) //delete

	result.base.ResourceSeparate("post",
		&shared.Post{},
		s5.QbsWrapIndex(&resource.PostResource{}, store),
		s5.QbsWrapFind(&resource.PostResource{}, store),
		s5.QbsWrapPost(&resource.PostResource{}, store),
		s5.QbsWrapPut(&resource.PostResource{}, store),
		s5.QbsWrapDelete(&resource.PostResource{}, store),
	)

{% endhighlight %}

In the above connection of a user record to its implementing resource we have
passed nil in three places to indicate that some "verbs" are not impremented
with this resource.

{% highlight bash %}
$ curl  http://localhost:5000/rest/userrecord/
Not implemented (INDEX, UDID)
{% endhighlight %}

"result.base" in the above code snippets is a Seven5 
[BaseDispatcher](https://gowalker.org/github.com/seven5/seven5#BaseDispatcher)
that will parse an incoming request for a URL and dispatch it to the 
appropriate resource or subresource.  The dispatcher has different, but
similar looking methods, "ResourceSeparateUdid" and "ResourceSeparate" that
indicate if you want to use UDID-based resources or standard integer ones.

The first parameter given to both of these methods is the portion of the URL
space that this resource occupies. This should be singular and lower case;
omitting underscores is also preferred.  The second parameter is the 
resource's wire type, and then there are five separate (thus the
"separate" in "ResourceSeparate") methods that provide the implementation
of the REST methods index, find, post, put, and delete respectively. Each of these
has a unique method signature except put and delete which are the same.

The use of the 
[QbsWrapFind](https://gowalker.org/github.com/seven5/seven5#QbsWrapFind) is
a means of expressing that you want to use Qbs in the implementation of the
method and would like a properly initialized Qbs instance to be part of 
your method signature. 

The use of "QbsWrapFind" or othe "QbsWrap\*" methods  also implies the 
default behavior for transaction rollbacks should the implementation
panic().  In general, if you use the QbsWrap* wrapper functions you should be 
able to ignore transactions.

## Return values and wire types

If you look in `resource/user_record.go` you'll see the 
method "FindQbs", the implementation of the find verb.  This method's 
signature is interesting:

{% highlight go %}
func (self *UserRecordResource) FindQbs(udid string, pb s5.PBundle, q *qbs.Qbs) (interface{}, error) {
...
}
{% endhighlight %}

The first parameter is the _provided_ udid sent from the client. The second
parameter is a collection of "other parameters" that may or may not have been
provided by the client (via query parameters, her session, etc) and the
last parameter is a Qbs instance for use in looking up the data in the
database. 

The return values are more murky.  Sadly, we cannot get strong typing
on the first parameter, it is checked at run-time.  This must match the
wire type associated with this resource implementation (see above for
the association).  The latter argument is an error.  If that error is
of Seven5's [Error](https://gowalker.org/github.com/seven5/seven5#Error)
type, you can provide the http status code an error message.  If it is
different type of error, the client will receive an "internal error" (http
code 500) result. 

It is perhaps unsurprising that about half of the implementation of
the FindQbs method in both UserRecord and Post is error checking and
returning. It "seems" that it "ought to be" just a simple database lookup
for a Find on a simple resource,  but the error checking is required for a 
real application.

<a name="static-site"></a>

# A static site

Let's look at sending fixed, static files between the client and server. 

## Preparation for this lesson
As before, you'll need to have godep and gopherjs installed,
a database running, migrations run, and fresno built and running.

You'll also need to have a working copy of "make" installed on your system as
demonstrated here with the "which" command. If you get no output, its
not installed:

{% highlight bash %}
$ which make
/usr/bin/make
{% endhighlight %}

If you are on OSX, you have use the app store to install "XCode" application
then navigate to XCode -> Preferences -> Downloads and install the component
named "command line tools".  On most linux systems make is installed by
default but on Ubuntu you may need to install the (enormous) package
"build-essential".

## Static files and don't repeat yourself
Based on the doctrine of 
[DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself), Seven5 offers
some tooling to help you generate web pages.  "Vat?!? Generatung
veb kontent [_ist verboten_](/index.html#modern-web-pages)!?!"   As was
stated previously, this content is not generated by the server itself,
but rather is built completely before the server runs, so it does not
fall foul of the "no generating content" rule.

An easy way to insure that content does not break the prescription
against server-generated HTML is to ask if the server can correctly
generate a [304](http://httpstatusdogs.com/304-not-modified) 
[Not Modified](http://httpstatus.es/304) response to a web request for
that html content.  The go default "serve file"
functions are careful to generate 304 for files that are just sitting in
a directory... so, relax, everything is fine.

## pagegen 
HTML files, and to some extent CSS, files are notorious for having 
repititon in them. Repitition makes things hard to maintain because you
can't make any change "in just one place."  pagegen is a tool that
lets you keep everything [OAOO](http://c2.com/xp/OnceAndOnlyOnce.html).

The program pagegen is in the directory `pagegen` which sets some 
[options](https://gowalker.org/github.com/seven5/seven5#PagegenOpts)
and then calls the 
[main](https://gowalker.org/github.com/seven5/seven5#PagegenOpts_Main)
provided by Seven5.  This pattern of providing an "entry point" that
you can wrap with your own "main()" is used both with pagegen and 
migrations in Seven5.  In both cases it is arranged this way because
Seven5 has "most" of the logic but allows you to provide your own
"semantics".

If you examine `pagegen/main.go` you'll see a couple of these semantics:

{% highlight go %}

//utility function for generating current year for footers
func year() int {
	return time.Now().Year()
}

//useful in page generation for generating links
func urlgen() shared.URLGenerator {
	return shared.URLGen
}

//this table adds functionality to the "pipelines" you can use in
//go templates.
var funcs = map[string]interface{}{
	"year":   year,
	"urlgen": urlgen,
}

{% endhighlight %}

These two 
[functions](http://golang.org/pkg/text/template/#example_Template_func) 
are added to the 
[template processing](http://golang.org/pkg/text/template/) that is done
by pagegen. 

### Building pagegen

{% highlight bash %}
$ godep go install tutorial/pagegen
{% endhighlight %}

This builds the pagegen command, and you'll need to do this before going further.

## Building index.html (and other pages)

The directory `pages` has all the static file content. You can go into
the `pages` directory and type `make html` to build the content.  (You can
do `make clean` before `make html` if you want to force make to do a full 
rebuild.):

{% highlight bash %}
$ cd $TUTROOT/src/tutorial/pages
$ make html
pagegen --support=support --dir=template --start=post/index.html --json=post/index.json > ../static/en/web/post/index.html
[similar output for other files]
$ make html
make: Nothing to be done for `html'.
{% endhighlight %}

> This make command must be run from the directory `pages`, it has relative
> paths that require, for example, `../static` to resolve to `TUTROOT/static`.

We are going to start by examining the first pagegen command that was run
by make, generating `static/en/web/post/index.html`, above. 

The "base" html file is `pages/template/post/index.html`. 
The json data bundle for that page is `pages/template/post/index.json`. 

The `pages/template/post/index.html` file is:

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
	{{template "BOOTSTRAP_HEAD_SECTION" .}}

<body>
	{{template "PAGE_NAV_BAR"}}

	<div id="blog-parent" class="container">
	{{template "PAGE_ERR_AREA"}}

	<div class="row">
		<div id="login-parent" class="col-sm-offset-8 col-sm-4">
		</div>
	</div>

    <div id="bottom-item" class="row">
    </div>

	 {{template "MISC_FOOTER" .}}
	
  </div> <!--container -->
	
	{{template "MISC_JSLOAD" .}}
</body>
</html>
{% endraw %}
{% endhighlight %}

The `pages/template/index.json` file is:
{% highlight json %}
{
	"title":"Fresno",
	"css_page": "/fixed/index.css",
	"js_file" : "index.js"
}
{% endhighlight %}

The "template" for our index.html page is "merged" with this json blob to
create the final output html. If you are familiar with 
[go templates](http://golang.org/pkg/text/template/) these are just go templates
with the data provided by the json file.

The make command above referenced the support directory 
`pages/template/support`.  _All_ the templates found in this directory
(files ending in ".tmpl") are available to the page being generated.   For
example, you can see the definition of the referenced
template BOOTSTRAP_HEAD_SECTION in `pages/template/support/bootstrap.tmpl`
and the templates MISC_FOOTER and MISC_JSLOAD in 
`pages/template/support/misc.tmpl`. 

It should be noted that the calls to these "partial templates" are passed
the argument "." (dot).  This is set to the root object of the json content
(see `pages/template/index.json` above) at the start of processing.  So
when the bootstrap template is reached, it will have access to the title
and css page associated with this content.

If we look at the definition of MISC_FOOTER in 
`pages/template/support/misc.tmpl`:

{% highlight html %}

{%raw%} {{define "MISC_FOOTER"}}{%endraw%}
  <hr>

  <footer>
    <p style="font-variant:small-caps;">&copy; Fresno Rocked The House In {%raw%}{{year}}{%endraw%}</p>
  </footer>
{%raw%}{{end}}{%endraw%}
{% endhighlight %}

This is the "year" function defined in `pagegen/main.go` and discussed 
at the beginning of this section.  This example shows you how
you can invoke arbitrary go code from within a template during pagegen
processing.

For any project of a reasonable size, such as even the simple fresno blog engine
the ability to avoid repeating yourself in the HTML

## The static directory

The output of pagegen, notably `static/en/web/post/index.html` in this lesson,
is placed in the `static` subdirectory.  That directory _must_ be checked
into git as it is the content your web server will respond with when 
queried for "/".  In other words, `static/en/web/post/index.html` will be
visible as "/en/web/post/index.html" when the server is running.

There are files in `static/fixed` that correspond to assets, like images
or css files, that don't change depending on the language of the user's 
browser or browsing device. Although we only support one language ("en") 
and one device ("web") right now, the content that might be dependent on 
these variables is separated into subdirectories.

## Run and test the static files

You should be comfortable enough now to build the fresno, migrate, and
pagegen applications anytime, so we won't call it out anymore.

From now on you must assume that you should run "fresno" from the primary
tutorial directory (`TUTROOT/src/tutorial` or the parent of `static`) because
this is how your application will be run on production. 

You can view the 
[index page](http://localhost:5000/en/web/post/index.html), 
static/en/web/post/index.html in your web browser on the local machine, 
or on heroku if you are pushing code there. The urls "/" and "/posts" are 
equivalent; they correspond to the same content in 
`static/en/web/post/index.html` and we will explain these urls in more 
detail later.

>If you are getting a 404 error in the local case
>it is probably because you are not running fresno in parent of `static`.


<a name="signup-form"></a>

# A Signup Form

Let's now turn our attention the the 
[signup form](http://localhost:5000/en/web/signup.html) which can be found
in `static/en/web/signup.html`.

This lesson uses the "_fixed form_ strategy" for dealing with the 
DOM content. In this strategy, all the HTML/DOM ids ("div#foo") are known 
in advance,  thus  it is easy to "hook up" the interactive parts of 
the application to  the form elements. In more complex interfaces, 
one cannot know all the  HTML ids in advance, because these are 
calculated at run-time and a different strategy is necessary.

## Preparation for this lesson
Make sure fresno is running and that you see the signup form 
when you go to /en/web/signup.html.

In your terminal make sure all your client side (browser) code is built
with the commands below.  This builds all the html and javascript needed
for the program.

{% highlight bash %}
$ cd $TUTROOT/src/tutorial/pages
$ make
{% endhighlight %}

## Form feedback

The form you should see will look like the image to the right, minus the
pink arrows and text which are for this tutorial.

<img src="/assets/img/signup.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:80%;height:80%; float:right;">

If you don't have the error console open, as shown at the bottom of the
screen cap, you should open it now.  You'll need it.  In Safari, you 
have to access the developer's menu and select "Show Error Console".  In
Chrome you should select "View" then "Developer" then "Javascript Console".

Typing into the form should give you some feedback in the right-hand
portion of the screen.  Once you have filled in all the areas of the
form and there are no errors, the "Sign Up" button will become 
enabled.  For now, don't hit the sign up button and just stay on this
page.

## HTML code 

In the `pages` directory you'll notice the
`pages/template/signup.html` and corresponding `pages/template/signup.json`.

We have exploited our "Dont Repeat Yourself" mantra here.  The code for
the html page is:

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
	{{template "BOOTSTRAP_HEAD_SECTION" .}}

<body>
	<div id="primary" class="container">
	{{template "PAGE_NAV_BAR"}}

    <h3>Sign Up For Fresno</h3>
	 {{template "PAGE_ERR_AREA"}}

	<form class="form-horizontal">

		{{template "FORM_5WIDE_TEXT" .first_name}}
		{{template "FORM_5WIDE_TEXT" .last_name}}
		{{template "FORM_5WIDE_TEXT" .email}}
		{{template "FORM_5WIDE_TEXT" .password}}
		{{template "FORM_5WIDE_TEXT" .confirm_password}}
  	
  		<div class="form-group">
    		<div class="col-sm-offset-2 col-sm-6">
      			<button id="signup" class="disabled btn btn-primary">Sign up</button>
    		</div>
  		</div>
	</form>

	 {{template "MISC_FOOTER" .}}
	
  </div> <!--container -->
  <div id="secondary" class="container hide">
    <div class="row">
      <h3 class="col-sm-12" class="bg-warning">You are already logged in.</h3>
    </div>
   {{template "MISC_FOOTER" .}}
  </div>

	{{template "MISC_JSLOAD" .}}
</body>
</html>
{% endraw %}
{% endhighlight %}

Since the code for each of the fields is the same, we have factored into
its own small template in `pages/template/support/form.tmpl`.  Then, we re-used that
template by changing the json:

{% highlight json %}
{
	"title":"Sign Up",
	"css_page": "/fixed/index.css",
	"js_file" : "signup.js",

	"first_name": {
		"id":"first_name",
		"type": "text",
		"label_text": "First Name",
		"placeholder": "John"
	},
	"last_name": {
		"id":"last_name",
		"type": "text",
		"label_text": "Last Name",
		"placeholder": "Public"
	},
	"email": {
		"id":"email",
		"type": "text",
		"label_text": "Email",
		"placeholder": "foo@example.com"
	},
	"password": {
		"id":"password",
		"type": "password",
		"label_text": "Password",
		"placeholder": ""
	},
	"confirm_password": {
		"id":"confirm_password",
		"type": "password",
		"label_text": "Confirm Password",
		"placeholder": ""
	}
}

{% endhighlight %}

Note that the html page references a different json object when invoking
"FORM_5WIDE_TEXT" for each form element.

## Client side code

Let's look in a bit more detail a how the client side go code gets built.


{% highlight bash %}
$ cd $TUTROOT/src/tutorial/pages
$ make clean
[elided for space]
$ make
gopherjs build  -o  ../static/en/web/signup.js ../client/signup.go
[other lines elided for space]
$ make
[gopherjs commands executed again]
{% endhighlight %}

When make" target runs, it _always_ generates the code for the client 
side code, using gopherjs to build javascript files such 
as `static/en/web/signup.js` (and the matching `static/en/web/signup.js.map`
file).   This is ok because it's quite fast as gopherjs does the same
type of rebuild-only-if-necessary that the standard go tool does.  The
make target is designed to give "fast feedback" when doing front-end
development.  Just _leave fresno running_ and then in another shell
switch to the `pages` directory and use "make" to "build everything 
again".  Then you can just refresh your browser and you'll see the changes.

### Entry point
The code for the client-side portion of this form is in `client/signup.go`.

A single html page, and its associated state, is represented as an
instance of Seven5's 
[Application](https://gowalker.org/github.com/seven5/seven5/client#Application) 
interface. There is only one method on Application, the
Start() method.  This method is critical because Seven5 will
do work to insure that this method will not be called before the 
DOM is fully ready. 

Thus in our go-level "main" for this page (`client/signup.go`), 
we just call  [s5.Main()](https://gowalker.org/github.com/seven5/seven5/client#Main)
with our Application object, of type "signupPage".:

{% highlight go %}
func main() {
	s5.Main(newSignupPage())
}
{% endhighlight %}

> If you are familiar with web programming, yes, you can do operations in
> main() that don't require the DOM to be available.

<a name="constraints"></a>

### Attributes and constraints

Typically, you want to keep page state in your Application object
necessary for dealing with later events. 
Here is the definition of signupPage:

{% highlight go %}

type signupPage struct {
        *uicommon.StandardPage
        first s5.StringAttribute
        last  s5.StringAttribute
        email s5.StringAttribute
        pwd1  s5.StringAttribute
        pwd2  s5.StringAttribute

        firstFeedback s5.StringAttribute
        emailFeedback s5.StringAttribute
        pwd1Feedback  s5.StringAttribute

        alreadyLoggedIn s5.BooleanAttribute
}

{% endhighlight %}

An attribute is Seven5 is just a value.  We have to use this slightly 
awkward notation to represent a string as a value, "s5.StringAttribute"
because attributes have a property that normal go values don't have:
they can be used as the source or destination for _constraints_. 

A constraint is just a function.  Such functions must take in attributes
as their inputs and produce an value.  It's called a constraint because
Seven5 has machinery that allows it to guarantee that your constraints are
always "met". 

> For the curious, the algorithm used to insure constraint evaluation is
> both correct, and close to minimal is [eval_vite](ftp://ftp.cc.gatech.edu/pub/gvu/tr/1993/93-15.pdf) from 93.

So, how does this work in our example form?  First,  the first
five named attributes in the signup page structure have their values constrained 
to be equal to the values in the type-in fields of the form.  The "magic" of
Seven5 is allowing constraints to be computed from parts of the DOM; all the 
other constraint processing algorithms/ideas have been 
around since the 1990s.  Thus, at any point in the code of `client/signup.go`
the value read from signupPage.first.Get() will be the value that is
currently in the corresponding text box on the screen.

We can go the other direction.  The three three fields of signupPage that
end in "Feedback" are  used to compute the values of the feedback areas 
to the right of the text entry areas.  Again, Seven5 allows constraints to
go "into" the  DOM as well as come from it.  So, a call like 
signupPage.firstFeedback.Set("foobar")  will display "foobar" to the 
right of the first name text entry box and, naturally, 
signupPage.firstFeedback.Set("") will remove what was there before.

To recap the way these eight fields are used in the application: Equality
constraints have been placed on the DOM elements for the text entry fields
("input" tags) and on the output areas ("label" tags) such that they remain
equal to (have the same string content as) the corresponding fields in
the signupPage structure. 

Our 8 constraints are guaranteed by Seven5, no action is
necessary to maintain them once the constraints are established.  You
can see the constraints being established in the Start() method if you
want the details.

### Going both ways

Why not combine the two directions of input constraints and output constraints?

{% highlight go %}

func emailFeedback(raw []s5.Equaler) s5.Equaler {
	email := strings.TrimSpace(raw[0].(s5.StringEqualer).S)
	if len(email) == 0 {
		return s5.StringEqualer{S: ""}
	}
	if len(email) < 6 { //a@b.co
		return s5.StringEqualer{S: "That doesn't look like an email address!"}
	}
	if strings.Index(email, "@") == -1 {
		return s5.StringEqualer{S: "That doesn't look like an email address!"}
	}
	return s5.StringEqualer{S: ""} //no error
}

{% endhighlight %}

This function is a "constraint function".  It is used to 
"tie together" signupPage.email and  signupPage.emailFeedback.
The parameter that comes in (sadly, not strongly typed) an 
s5.StringEqualer, which means that it is the value that was produced by 
an s5.StringAttribute, in this case signupPage.Email.  The value 
returned by this function, also s5.StringEqualer, will end up being 
assigned to signupPage.emailFeedback.

A very similar function is used to provide the name feedback 
(nameFeedback() in the source) and the password comparison
feedback (pwdFeedback()).

The constraint function below is a function of five parameters, all of
them coming from the five "input" attributes that are described above. It 
produces a boolean in the form of an s5.BoolEqualer:

{% highlight go %}

func formIsBad(raw []s5.Equaler) s5.Equaler {
	firstName := strings.TrimSpace(raw[0].(s5.StringEqualer).S)
	lastName := strings.TrimSpace(raw[1].(s5.StringEqualer).S)
	email := strings.TrimSpace(raw[2].(s5.StringEqualer).S)
	pwd1 := raw[3].(s5.StringEqualer).S
	pwd2 := raw[4].(s5.StringEqualer).S

	if len(firstName) == 0 {
		return s5.BoolEqualer{B: true}
	}
	if len(lastName) == 0 {
		return s5.BoolEqualer{B: true}
	}
	if len(email) < 6 {
		return s5.BoolEqualer{B: true}
	}
	if strings.Index(email, "@") == -1 {
		return s5.BoolEqualer{B: true}
	}
	if len(pwd1) < 6 {
		return s5.BoolEqualer{B: true}
	}
	if pwd1 != pwd2 {
		return s5.BoolEqualer{B: true}
	}
	//enable button
	return s5.BoolEqualer{B: false}

}

{% endhighlight  %}

This is the function that looks at the form and decides if the 
Sign Up button should be enabled.  This function's boolean result is
connected to the DOM via the CSS class "disabled".  In other words, 
the guarantee here is that if the value of this function is true
(really an s5.BoolEqualer with a B field of true) then the class 
"disabled" will be present on the button tag.  If the value of this
function is false, then the tag is guaranteed to _not_ be there.  This
means that we can now enable or disable (really "not disable" and "disable")
elements of the UI based on constraints.

## The Big Win

>If you don't have experience with building web UIs, this may seem
>like a "small win". 

You should note that the code of 
`client/signupPage.go` has *no event handlers*. Thus, there are no
"callbacks" to handle events, and the spaghetti code that often 
results is obviated.  (You can create event handlers in Seven5
applications, but they are needed far less often.)

The most common reason for convoluted event-handling code
is attempting to deal with all the possible semantic cases that come from
a user input.  For example: Do we need to re-compute the value of the
enabled/disabled state of the Sign Up button on any keypress in the
email field?  (Answer: yes.)  Do we need to recompute the value of
the password feedback field on any keypress in the email field? 
(Answer: no.)  More fun: What if the user adds a space to the end of 
the last name  field? Do we need to recompute the state of the 
Sign Up button?  When you use constraints, you can simply ignore this 
type of question.

Constraints are not a free lunch, they require work from you 
in structuring your application.  They require you to think 
carefully about the inputs and outputs of your user interface, and 
require clear articulation of the processing to be done.
This articulation must be done so that the processing of inputs to
outputs can be encoded in constraint functions.  Our experience has
shown that the effort required to structure a UI with constraints is
easily outweighed by the benefits gained in cleaner UI code.

## GOPATH and jsmaps

Gopherjs generates 
[javascript source maps](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1)
that allows you to debug in the browser at the go level.

If you look at the developer console (see figure above) at the right of
the "hello, world" you'll see "signup.go:56" in italics.  The line number
may be slightly different in your version.  You can click
on it and go to the line in the source code.  Seven5 knows how to 
interpret the request to show you the source code based on your GOPATH
(set in the enable script).  You may find it interesting to look at the 
log messages generated by fresno when you click on a link like this.

If you look at the source code, you'll notice a couple of differences
that are related to coding in a browser:

{% highlight go %}

//called after the dom is ready
func (s signupPage) Start() {
	print("hello, world")

{% endhighlight %}

The function used to print to the console is the go builtin "print()", 
not any of the methods from the log or fmt package.  print() is fairly
smart and knows how to print complex objects into the java script
console, so pretty much _anything_ can be printed, although you'll be
looking at the javascript representation, rather than the go one. 

More interesting is to introduce a panic("foo") immediately after that print
and then rebuild ("make" in the pages dir) the client-side code.  If you
have done this, you'll see something like this in your browser if you
reload the page:

<img src="/assets/img/panic_1.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:50%;height:50%; float:right;">

You should click on the little triangle to open up the stack trace.  This
is the stack trace at the moment of the panic, with the panic error at the
top. You'll usually see a mixture of go and javascript code in the stack,
since the compiler is generating numerous functions to support your go code.
Although you can look at the javascript code, don't bother, it was generated
by a machine.  The go source code (italics at the right is more interesting):

<img src="/assets/img/panic_2.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:50%;height:50%; float:right;">

Once you click through a link, you'll need to "go back" from the source code
to the stack.  This is often confusing for people new to debugging inside the
browser, so I've highlighted the little "go back" button that affects the
debugging/error console.  If you click on the "app.go" link in that stack 
trace, you'll be dumped into the Seven5 source code.

### Source debugging

<img src="/assets/img/panic_3.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:50%;height:50%;">

The next screen shot shows the source code if you clicked through the link
in the previous screenshot labelled "signup.go:57".  You can click in the
margin to the left of a line to put in a breakpoint.  Insert this 
breakpoint as show and then reload the page.

<img src="/assets/img/panic_4.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:50%;height:50%;float:right;">

You will see a display somewhat like this one. You are now in the land of
debuggers and most people familiar with them should recognize that they
can single step, return from current function, etc.  The author doesn't
find a lot of value in source debugging in a browser, but some people are
really into debuggers so it seemed a bit callous to not share at least
the starting point.

<a name="create-users"></a>

# Creating Users

Now that we htave looked at the form that takes in a user's information, we are
going to examine how it connects to the server side code that actually 
creates new users in the database. This lesson is primarily about not trusting 
user input.

## Preparation for this lesson
You'll need to have fresno running for this lesson.

## Client side, sending the data

In Start() of the application code, in  `client/signup.go`, there an event
handling function associated with the user clicking the big blue button: 

{% highlight go %}
func (s signupPage) Start() {
	//... elided for space ...
	button.Dom().On(s5.CLICK, func(evt jquery.Event) {
		evt.PreventDefault()
		var ur shared.UserRecord
		ur.EmailAddr = s.email.Value()
		ur.FirstName = s.first.Value()
		ur.LastName = s.last.Value()
		ur.Password = s.pwd1.Value()
		ur.Admin = true //hee hee hee
		contentCh, errCh := s5.AjaxPost(&ur, "/rest")
		go func() {
			select {
			case <-contentCh:
			case err := <-errCh:
				print("failed to post", err.StatusCode, err.Message)
			}
		}()
{% endhighlight  %}

It should be fairly clear that this is responding to a click
event and copying data out of the form into a instance of the
wire type shared.UserRecord.  (Attributes and the wire type structures 
were discussed in previous lessons.)  This is then sent to the server with 
AjaxPost(). The details of the Ajax call will be covered in a future
lesson.

## Server changes

The server side is more interesting.  As we pointed out in the lesson
on wire types when discussing resources, in `freso/main.go`
we have informed the dispatcher that we have a Post
implementation.  This will be called, via ajax, when the user clicks the big blue
button.  We'll repeat the call that establishes the connection between
the Post (and, in this case, Find) method and the wire type 
shared.UserRecord:

{% highlight go %}
	result.base.ResourceSeparateUdid("userrecord",
		&shared.UserRecord{},
		nil, //index
		s5.QbsWrapFindUdid(&resource.UserRecordResource{}, store),
		s5.QbsWrapPost(&resource.UserRecordResource{}, store), //post
		nil, //put
		nil) //delete

{% endhighlight  %}

In the resource implementation, `resource/user_record.go`, we have implemented 
the method PostQbs to handle the information sent from the client. 
The server needs to take care to not simply trust the data in "proposed" (did you
spot the "hee hee" moment in the client-side?):

{% highlight go %}
func (self *UserRecordResource) PostQbs(i interface{}, pb s5.PBundle, q *qbs.Qbs) (interface{}, error) {
        var ur shared.UserRecord
        proposed := i.(*shared.UserRecord)
        e := strings.ToLower(strings.TrimSpace(proposed.EmailAddr))

        err := q.WhereEqual("email_addr", e).Find(&ur)
        if err != nil && err != sql.ErrNoRows {
                return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("couldn't find: %v", err))
        }
        if err == nil {
                return nil, s5.HTTPError(http.StatusBadRequest, fmt.Sprintf("email address already registered %s: %s", proposed.EmailAddr, ur.UserUdid))
        }
        //just to make doubly sure we don't inadvently trust data from the client we
        //use the newly created ur which is a zero value at this point
        ur.Admin = false //nuke the site from orbit, its the only way to be sure
        if strings.Index(e, "@") == -1 {
                return nil, s5.HTTPError(http.StatusBadRequest, fmt.Sprintf("email address not ok: %s", proposed.EmailAddr))
        }
        ur.EmailAddr = e //copy it over
        if len(proposed.Password) < 6 {
                return nil, s5.HTTPError(http.StatusBadRequest, fmt.Sprintf("password too short: %s", proposed.Password))
        }
        ur.Password = proposed.Password //copy it over
        if len(proposed.FirstName) == 0 || len(proposed.LastName) == 0 {
                return nil, s5.HTTPError(http.StatusBadRequest, fmt.Sprintf("bad first or last name"))
        }
        ur.FirstName = proposed.FirstName
        ur.LastName = proposed.LastName
        //XXX this has a race condition which could cause two or more users with same
        //XXX email, right way to fix it is a DB constraint "unique"
        //values are ok, write it
        ur.UserUdid = s5.UDID() //generate a random UDID
        if _, err := q.Save(&ur); err != nil {
                return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("couldn't save: %v", err))
        }
        return proposed, nil
}

{% endhighlight  %}

## Testing creating users

You should be able to build the client side with "make" in `pages` and
the server with "godep go install tutorial/fresno" the main tutorial directory.
You can visit the new sign up page 
[here](http://localhost:5000/en/web/signup.html) if fresno is running. You can
create users by using the signup form.  You can create many if users you log
out of after creating each one.  You may find it interesting to return to
the signup page when you are logged in; you'll have manually enter the URL
since the navigation bar will try to steer you away from the this course of
action.

<a name="pwd-allow"></a>

# Password Authentication: Allow Methods And Sessions

For the avoidance of doubt: _authentication is nasty_.  The reader 
will likely have noticed that the Seven5 programming model to this 
point was very clearly differentiated: rest services that communicate 
via apis mounted at  /rest/resourcename or static files, mounted at /. 

Static files were served up in the usual way, with http status 
code 304 allowing convenient caching, and requests to the apis 
in /rest could not be cached as they were "hot". 

Authentication creates a problem.  We are going to be forced to introduce
to new "endpoints" in our URL space that are neither fish nor fowl: 

* /auth: the place to send data that concerns an attempt to log in or out
* /me: the place to request information about the currently logged in user

Neither of these endpoints can be cached, yet they are not RESTful. 

<a name="setup-auth"></a>

## Setup for this lesson

You'll need to make sure fresno is running for this lesson.

Then in another window try to retreive Mary's record from fresno, as before:

{% highlight bash %}
$ curl localhost:5000/rest/userrecord/515f7619-8ea2-427f-8cf3-7a9201c747dd
Not authorized (FIND, UDID)
{% endhighlight %}

## Allow methods

Your attempt to retreive that record has been refused because the implementation
of the user record resources uses _allow methods_ to
control access to this url.  Since your attempt to hit the server
with curl is not authenticated, much less authenticated as Mary, 
you cannot read this URL. 

Allow methods permit coarse-grain policies to be implemented "around"
a particular REST resource.  Allow methods run *before* any resource code
is executed and they indicate that access is or is 
not allowed  via their return value. If the return value is false, 
the resource method is not called at all and error is returned to 
the client.  The allow methods for a resource that uses UDIDs for 
resource identity are represented as interfaces in Seven5:

{% highlight go %}

//For POST method to a resource
type AllowWriter interface {
        AllowWrite(PBundle) bool
}

//For INDEX method
type AllowReader interface {
        AllowRead(PBundle) bool
}

//For PUT, DELETE and FIND
type AllowerUdid interface {
        Allow(udid string, method string, pb PBundle) bool
}
{% endhighlight %}

By implementing any of these interfaces as part of your resource 
implementation, you can get the option to "accept or reject" method
calls.  You are also provided with the parameters that will be sent
to the resource method if it is invoked, so you can make decisions
based on parameters, sessions, etc. 

Each of these methods are provided with a Seven5 Pbundle (pronounced
pee-bundell) or _parameter bundle_.  This contains numerous values
that have been sent by the client side and will be detailed later. 
The last method in our code above is provided with the method name 
(2nd parameter), in uppercase letters, to allow the receiver to
discriminate based on the method invoked. 

In our implementation of `resource/user_record.go` we now have 
a implementation of Allow that rejects our effort to GET the resource.

{% highlight go %}
//You can only read or update yourself.
func (self *UserRecordResource) Allow(udid string, method string, pb s5.PBundle) bool {
	if pb.Session() == nil {
		return false
	}
	ud := pb.Session().UserData().(*shared.UserRecord)
	return ud.UserUdid == udid
}
{% endhighlight %}

This implementation uses the existence of a session in the parameter
bundle as a way to see if the user is logged in and, if the user
is logged in, checks that the attempt to "FIND", "PUT" or "DELETE" a record 
has the same Udid as the logged in user.  In other words, only Mary can
retreive info about Mary.

## Sessions

Seven5 provides some common utilities for manipulating a _session_
inside the server of an application.  The utilities make the assumption
that a user that is "logged in" to the application will have one 
and only one session, even if they have multiple browser windows connected
to fresno.   The semantics of "logged in" are defined by
the application itself, but typically involve some type of
authentication. 

For this lesson, we are assuming that the user types in
an email address and a password, sends them to the server, and the
server compares the submitted password to the value it has stored
for that email address. In the successful case, a session is created
as the _user data_ for the session is set to a UserRecord struct,
from `shared/user_record.go`.  If the passwords don't match, an error
is returned.

The user data associated with a session can be any go value (it's an 
interface{}) but typically what needs to be "hung" on the session 
is information about the current user.  This is what is done fresno.

### Session keys

Seven5 uses `SERVER_SESSION_KEY` to insure that users (client programs)
don't tamper with their sessions.  Only the server knows the 
`SERVER_SESSION_KEY`. Check this lesson's enable script 
(`enable-fresno`) for the sample value.  The user's browser will 
get a cookie named 'seven5-fresno-session' that will have an 
encrypted value in it.  For example, the cookie value might be 
something like: 

<pre>
65a16749d4a090d2abcfee571bfc3c7e3264a0a88e2ddd839f25a1a94aadf2c31d80fb62b31ae6657a0f7a2449412a3dd2598a132ae7785feca037ec68764058a7f8
</pre>

If the client _could_ tamper with this value successfully, 
it would allow them to masquerade as any user!  The primary benefit 
of using this type of encrypted session data is to allow the server 
to store a _persistent_ value in the client's browser--in our case 
an email address plus an expiration time--that survives across server
restarts.  Although Seven5 maintains the sessions in memory, 
we can recover sessions that were created prior to this server's 
process execution if we find a valid cookie.  Valid here means 
"decrypts to something sensible" using `SERVER_SESSION_KEY`.

You should pick a key if you plan to use any system like this in practice.
You can compile a program called key2hex to do this:

{% highlight bash %}
$ cd $TUTROOT/src/tutorial
$ godep go install github.com/seven5/seven5/key2hex
$ Godeps/_workspace/bin/key2hex thisisanexample0
746869736973616e6578616d706c6530
{% endhighlight %}

Obviously, you should pick your own password rather than
"thisisanexample0", we recommend using 
[a strong password generator](https://strongpasswordgenerator.com)
to generate a 16 character password.  Then you can use this in your
enable script to set the secret for your server.

If you have to change the key for some reason, this is not catastrophic.
All previously created sessions will become  invalid (and the
corresponding users immediately logged out) since their cookie will no 
longer decrypt to a "sensible" value. 

>Is this strategy vulnerable to known plaintext attacks? The plaintext
can be easily determined from the Seven5 source code, so the scheme appears
to have its security resting entirely on the inability of the attacker
to generate a "legitimate" cookie value without knowing the secret. 

<a name="pwd-client"></a>

# Password Authentication: Client side

## Setup for this lesson

You'll need to have fresno running for this lesson.

## New URLs

fresno has two authentication URLs that were describe
[previously](#pwd-allow) as neither fish nor fowl. The first of these 
is "/auth" that is the authentication point for  submitting a username 
and password.  It can also be used for other tasks, such as logging 
*out*, that will be explained in later lessons. 

The other, perhaps more suprising URL, is "/me" that allows a client to
ask the server "who am I?" This uses the cookie stored in the user's 
browser explained [previously](#pwd-allow) to determine who, if anyone,
the current browser is logged in as.  You can see try this with curl, 
of course:

{% highlight bash %}
$  curl localhost:5000/me
no cookie
{% endhighlight %}

If you change the command above to be "curl -v" you can see that the
server is actually returning a 401 error code. This is the user 
interface of fresno's cue to display a page appropriate for a user 
that is not logged in. 

Navigate to the 
[the fresno home page](http://localhost:5000/)
and you will see this:

<img src="/assets/img/login1.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>

## Client-side checking of /me

The implementation of the client-side code for the top right corner of
the page is in `client/uicommon/standard_page.go`. The function
GetLoggedInUser is responsible for checking the value returned from
/me to see what to display on the upper right corner of the page.  This
code is shared among several pages so it has been placed in the 
client/uicommon package to do avoid duplication.

There are three common idioms for client-side Seven5 code in the 
GetLoggedInUser() method so we'll reproduce all of it here:

{% highlight go %}
func (self *StandardPage) GetLoggedInUser() {
        chLoggedIn, chLoginErr := s5.AjaxGet(&shared.UserRecord{}, shared.URLGen.Me())
        go func() {
                select {
                case raw := <-chLoggedIn:
                        self.user = raw.(*shared.UserRecord)
                        self.AddCurrentUserNav()
                        self.AddLogOutNav()
                        if self.arrivalFunc != nil {
                                self.arrivalFunc(self.user)
                        }
                case <-chLoginErr:
                        if self.includeRightNav {
                                self.AddSignUpNav()
                                self.AddLogInNav()
                        }
                        if self.arrivalFunc != nil {
                                self.arrivalFunc(nil)
                        }
                }
        }()
}
{% endhighlight %}

<a name="ajax-idiom"></a>

### The ajax idiom

We are sending the server a request for a particular url, the return value of 
shared.URLGen.Me() which works out to "/me" (see below for more on URLGen). 

This is a common pattern in Seven5 client side programs:

{% highlight go %}

	channelContent, channelError := s5.AjaxGet(/*wire type*/, /*path*/)
	go func() {
		select {
		case raw := <-channelContent:
			content:=raw.(/*ptr to some content you are expecting*/)
			/* create some UI based on the content*/
		case errInfo := <-channelError:
			/* display an error based on the error returned */
		}
	}
{% endhighlight %}

A few things to note about this idiom. 

* First, the use of select would 
block, so it must be run inside a go routine.  This is in keeping with the
fact that it may take an unknown amount of time for the result of the 
s5.AjaxGet() to return.  If you forget to use a go routine, you'll get a helpful 
panic() from the runtime system. 
* Second, the wire type for Get must be a
pointer to a struct and the returned value is unpacked with the normal go 
package encoding/json.  Any of the special tags on the structure related to 
json decoding apply. For Ajax calls that do not need to send any data to 
the server, only request data, it is customary to put a zero-valued instance 
of the wire type as the parameter.  We have done this in the previous code
snippet for AjaxGet.    For calls that do send values to the server, 
such as AjaxPost or AjaxPut, the _content_ of the first parameter is used to 
create the body via json encoding but the value must also be a pointer to a 
struct (a wire type).
* Finally, only one of the two channels returned from
an Ajax call will receive a value, never both.  For the content channel,
the caller can safely assume that the value on the channel is the same as
the wire type provided in the original call to the Ajax function.

> We cannot achieve strong typing with the content channel because Seven5 
> can't know the types in your program.  Further, we use json as the wire
> format and this breaks some amount of type safety.  We encourage folks that
> are interested in this issue to experiment with using protobufs, gobs, or
> similar as the transport.  This would allow a more "tight rpc style" of 
> calls from client to server rather than the "loosey-goosey" style of json.
> See 
>[this example](https://github.com/shurcooL/play/commit/e7e89367c071462ed0d150663bba54a78c06deed) how to use jsonrpc.

## The Urlgen idiom

The file `shared/urlgen.go` declares a single variable URLGen of type
UrlGenerator. 

{% highlight go %}
type URLGenerator interface {
	IndexPage() string
	LoginPage() string
	SignupPage() string
	Auth() string
	Me() string
	UserRecord(udid string) string
	UserRecordResource() string
	PostResource() string
	Posts(offset int, limit int) string
	Post(id int64) string
	PostView(id int64) string
	PostEdit(id int64) string
	NewPost() string
}
{% endhighlight %}

Much, acutely painful, experience has shown that it is error-prone 
to have hard-coded string constants that correspond to the URLs in 
your application.  It is downright suicidal to have "fmt.Sprintfs()" 
that compute complex URLs in your application.  Use of such strings
makes it hard to change the URL space of your application during 
development.

Besides the obvious benefit of once and only once, 
typically the client and server need to share much "logic" around 
the construction of URLs.  For example, you can see above that 
the URLGenerator for fresno  has a method that allows one to create a 
URL (really just the "path" part of a URL) for a given UDID, avoiding
the deathtrap of having client an server both generating these 
URLs via strings or fmt.Sprintf(). 

Since a Seven5 application is go on both the client and server, you 
can link the _same code_ into both sides--and breath a sigh of relief
that things will stay in sync.  As an aside, if you prefer to 
version your URLs for backward compatibility, this can be 
accomplished with the use of the URLGenerator idiom.

## The parent idiom

It is common in web pages to have some part of the display that cannot
be known until data is pulled from the server at run time.  In the
example GetLoggedInUser method above, the dynamic portion of the UI is the
value (or link) to be displayed in the upper right of the index page. 
There are four method calls in GetLoggedInUser that affect the display:
AddCurrentUserNav, AddLogOutNav, AddSignupNav and AddLogInNav.   For space
we'll just show one of these:

{% highlight go %}

func (self *StandardPage) AddLogOutNav() {
        tree :=
                s5.LI(
                        s5.A(
                                s5.HtmlAttrConstant(s5.HREF, "#"),
                                s5.Text("Log Out"),
                                s5.Event(s5.CLICK, func(evt jquery.Event) {
                                        evt.PreventDefault()
                                        self.PerformLogOut()
                                }),
                        ),
                ).Build()
        self.navRight.Dom().Append(tree) //this is the crucial bit!!
}
{% endhighlight %}

The "to be determined" bit is encoded in go code as
the variable self.navRight (the right hand portion of the nav bar). This is
referred to as the parent idiom because it is common to have the variable
be the parent in the DOM of the content to be added.  We used the method
Append() in the crucial bit above to append our dynamic content to the child 
list of the self.navRight element.  The parent's 
definition is:

{% highlight go %}
     self.navRight = s5.NewHtmlId("ul", "nav-right")
{% endhighlight %}

The call to NewHtmlId searches the DOM for *exactly one* tag of type
"ul" with Id of "nav-right".  That tag is in the HTML code 
(`pages/template/support/page.tmpl`) that defines the template
PAGE_NAV_BAR that is used for several of the pages:

{% highlight html %}
{% raw %}
{{define "PAGE_NAV_BAR"}}
  <nav class="navbar navbar-default navbar-fixed-top">
    <div class="container">
      <div class="navbar-header">
        <a class="navbar-brand" href="http://seven5.github.io/tutorial.html">Tutorial</a>
      </div>
      <div id="navbar" class="navbar-collapse collapse">
        <ul id="nav-left" class="nav navbar-nav">
        </ul>
        <!-- this is the crucial bit!! -->
        <ul id="nav-right" class="nav navbar-nav navbar-right">
        </ul>
      </div>
    </div>
  </nav>
{{end}}
{% endraw %}
{% endhighlight %}

The above html code results in no display when the page is initally
loaded.  The content is filled in once the Ajax call in GetLoggedInUser
is completed.

It is important to note that the type HtmlId in Seven5 is aggresive
about checking for the existence of the identified portion of the DOM.
In the case of the html code above try changing the crucial ul element
to be this:

{% highlight html %}
	<ul id="xxxnav-right" class="nav navbar-nav navbar-right">
{% endhighlight %}

This change of the id does not generate a compile time error, sadly, so you
can rebuild the code:

{% highlight bash %}
$ cd $TUTROOT/src/tutorial/pages
$ make
{% endhighlight %}

If you reload the http://localhost:5000/ page now, 
and look at the error console you'll see:

<img src="/assets/img/sync1.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>

This is only a cross-check between the go and HTML code,
and it's still possible to get "out of sync" between the two, but this
prevents at least some of the most common problems when renaming
HTML elements.  It can also be triggered intentionally as a 
"dead HTML code" check.

<a name="pwd-server"></a>

# Password Authentication: Server side

## Setup for this lesson

You'll need to have fresno running for this lesson.

## Performing a log in

With the background covered in the previous two lessions,  let's log in to fresno.

<img src="/assets/img/login2.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>

>Don't forget to change your ul tag in `pages/template/support/page.tmpl`
>back and remove the "xxx" if you made that change in the previous lesson!

The screenshot above shows the Log In button as enabled, but that does
not happen until you have entered an email address and a password. 
Naturally, this is implemented via [constraints](#constraints) in
the function formIsBad() in `client/login.go`. 

When you push the button, a bundle is sent to the /auth URL of the
server.  This is accomplished by pulling the username and
password out of attributes of the page, encoding a json blob, and
the using the Ajax idiom as above.  Here is the code for the click
handler of the login page from newLoginPage() in `client/login.go`:

{% highlight go %}
	button.Dom().On(s5.CLICK, func(evt jquery.Event) {
		evt.PreventDefault()
		var pap s5.PasswordAuthParameters

		pap.Username = result.email.Value()
		pap.Password = result.pwd.Value()
		pap.Op = s5.AUTH_OP_LOGIN

		contentCh, errCh := s5.AjaxPost(&pap, shared.URLGen.Auth())

		go func() {
			select {
			case <-contentCh:
				uicommon.SetCurrentPage(shared.URLGen.IndexPage())
			case err := <-errCh:
				if err.StatusCode == 401 {
					displayErrorText("That's probably not your password.", false)
				} else {
					displayErrorText("Login trouble: "+err.Message, true)
				}
			}
		}()
	})
{% endhighlight %}

Two things to note about this code: First, there is no need for any
logic for the case where the user tries to click the button without a
username or password entered. That cannot happen due to the constraint
that enables the button only when the form is populated.  Second, success
case of the Ajax idiom uses a method SetCurrentPage() in the uicommon
package to "brute force" the user onto a different page.

Joe's password is set in the file `migrate/main.go` to "seekret".  When
you click the button to log in as joe, you'll return to the index
page, but it will look like this:

<img src="/assets/img/loggedin1.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>


## "Simple" in Seven5

When we are discussing the server-side implementation of the password
authentication, we'll refer to some Seven5 types like "SimpleFoo" and
the associated interface Foo.  In summary:

{% highlight go %}

type Foo interface{
	//methods
}
type SimpleFoo struct{
	//implementation
}
//which could be called, but it would take too long
type FullImplementationOfFooThatsEnoughToStartWithButMightNotBeWhatYouWantLongTerm struct{
//implementation	
}

{% endhighlight  %}

This naming convention is used throughout Seven5, so you can recognize the
intention of many types from their names. 

## Server side of password authentication

Seven5 has a type SimpleSessionManager that can handle creating
and destroying sessions and coordinating this with a cookie maintained
on the user's browser. However, it cannot know the application specific
definition of the semantics of "log in".  Seven5 provides an interface
ValidatingSessionManager which you can implement and then pass to another
Seven5 type: SimplePasswordHandler. 

This implies a layering like this, with the implementor in parenthesis:

<pre>
SimpleSessionManager (seven5) 
	MyValidatingSessionManager (you)  
		SimplePasswordHandler (seven5)
</pre>

The go code basics for this particular layering applied to fresno is in 
`fresno/validating_sm.go`.

{% highlight go %}

// fresnoValidatingSessionManager knows how to check credentials, interact
// with sessions, etc.
type fresnoValidatingSessionManager struct {
	*s5.SimpleSessionManager
}

// newFresnoValidatingSessionManager creates a new FresnoValidatingSessionManager
// but returns it as an s5.ValidatingSessionManager to insure that we meet the
// interface it requires.
func newFresnoValidatingSessionManager() s5.ValidatingSessionManager {
	result := &fresnoValidatingSessionManager{}
	result.SimpleSessionManager = s5.NewSimpleSessionManager(result)
	return result
}

{% endhighlight  %}

### ValidatingSessionManager 

ValidatingSessionManager is an interface to allow you to implement
your application-level semantics:

{% highlight go %}
type ValidatingSessionManager interface {
        SessionManager
        ValidateCredentials(username, password string) (string, interface{}, error)
        SendUserDetails(i interface{}, w http.ResponseWriter) error
        GenerateResetRequest(string) (string, error)
        UseResetRequest(string, string, string) (bool, error)
}
{% endhighlight  %}

We will ignore the password reset code for now, but the
methods ValidateCredentials and SendUserDetails are worth discussing
now.  These methods are used for checking the password and transmitting
a user record to the client side, respectively. 

These are implementations of these two methods from `validating_sm.go`;
we have removed some of the error checking for clarity:

{% highlight go %}
// Check that a username and password are as we have them in the database.
// If they match, we return the user's UDID as the uniq value for the session
// plus the user data record.
func (self *fresnoValidatingSessionManager) ValidateCredentials(username, pwd string) (string, interface{}, error) {
	q, _ := qbs.GetQbs()
	defer q.Close()

	var ur shared.UserRecord
	u := strings.TrimSpace(username)
	p := strings.TrimSpace(pwd)
	if len(u) == 0 || len(p) == 0 {
		return "", nil, err
	}
	cond := qbs.NewEqualCondition("email_addr", u).AndEqual("password", p)
	if err := q.Condition(cond).Find(&ur); err != nil {
		if err != sql.ErrNoRows {
			return "", nil, err
		}
		//normal case of bad pwd
		return "", nil, nil
	}
	//return the udid as uniq part,then the rest of the object as user data
	return ur.UserUdid, &ur, nil
}

// SendUserDetails is responsible for filtering out fields that we may not wish
// to send to the client side of the wire when returning a user record from /me
func (self *fresnoValidatingSessionManager) SendUserDetails(i interface{}, w http.ResponseWriter) error {
	ur := i.(*shared.UserRecord)
	ur.Password = ""
	return s5.SendJson(w, ur)
}

{% endhighlight  %}

<a name="posts-p1"></a>

# Display Blog Posts, Part 1

## Setup for this lesson

You'll need to have fresno running for this lesson.

## The upside-down tutorial

This tutorial, so far, has not really been about blogging.  The first
dozen-or-so lessons have been about all the _other_ things that you
need to understand to make a real, working application.  We have
discussed databases, migrations, deployment, dependency 
management, password authentication, code conventions for a
real application, and some REST with a sprinkle of data interchange. 
This was by design; most tutorials leave out all the "other stuff" 
and one ends up with an app that does a few simple things, 
usually running locally, and then the "hard part" comes when you try 
to make it real.  This tutorial has covered all the hard parts already, 
now we can focus on getting to the semantics of our application.

## Index methods

Index methods are for listing a collection of identical rest resources,
such as blog posts. In fresno, we use a database query to get the list 
of posts to return to the client side.  The index method takes some query 
parameters to allow the client side to do pagination if it desires.  Finally,
we do a slight bit of adjustment to the  joined user record that represents
the author of the post. 

The code, from `resource/post.go`, is:
{% highlight go %}

func (self *PostResource) IndexQbs(pb s5.PBundle, q *qbs.Qbs) (interface{}, error) {
	limit := int(pb.IntQueryParameter(shared.LIMIT_PARAM, 10))
	offset := int(pb.IntQueryParameter(shared.OFFSET_PARAM, 0))
	q, err := qbs.GetQbs()
	if err != nil {
		log.Printf("unable to get db connection:%v", err)
		return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("unable to get db connection:%v", err))
	}
	var posts []*shared.Post
	err = q.Limit(limit).Offset(offset).OrderByDesc("created").FindAll(&posts)
	if err != nil && err == sql.ErrNoRows {
		return []*shared.Post{}, nil
	}
	if err != nil {
		return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("error in qbs find: %v", err))
	}
	for _, p := range posts {
		author := p.Author
		author.Password = ""
		p.TextShort = markdown(p.TextShort)
		p.Text = markdown(p.Text)
	}
	return posts, nil
}
{% endhighlight  %}

## Joiner

Frequently, you will want a web page to display a series of broadly 
similar items, such as blog posts in fresno. Seven5 provides two types
that aid in this process: Joiner and Collection.  A Collection models
a sequence of items to be displayed on the screen and the Joiner does
the work of mapping an individual model--an element of the collection--
into HTML that should be added or removed from the screen.  A Collection
must be created with an associated Joiner.

<a name="ajax-display-seq"/>

Broadly, the common sequence of actions for displaying a list of
items of a resource type:

1. Make [Ajax call](#ajax-idiom) to the server
2. Create a model from each result item returned
3. Add model to collection

The last of these steps will end up calling the Add() method on the
Joiner to display each model on the screen. 

Step 2 may seem strange at first, since we are effectively 
decoding the raw content into one "model", the wire type used with the
Ajax call, and then creating a second model from this first one. This is 
typically necessary because the "model"  in step 2 is an explicit model (in the 
[MVC](http://en.wikipedia.org/wiki/Model–view–controller) sense) whereas
the model in step 1 just represents what was sent over the wire. 

Further the model in step 2 will typically use 
[attributes](#constraints) because it will need to have these types
for connecting to the display.  In step 1 a value might be a string,
where in step 2 it would be a StringAttribute.  Finally, often the
display needs are different from the simple values sent over the wire
so some processing must be done to the wire types to create the model
values in step 3.  For example, in Fresno the server sends (wire)
the post author's data as first name and last name, both as strings, but
the display wants a single StringAttribute for name.

<a name="treebuilding"/>

### Building a tree

The first job of a joiner is to build a tree of HTML nodes that can
be added to the DOM for displaying the model.  Seven5 provides a
number of functions to add in this tree construction process.  The
functions that create DOM elements are all upper-case names that
correspond exactly to the HTML element of the same name. For example:


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

The snippet above uses formatting to make the tree structure easier to
see.   This snippet creates a div element, which has a div element as
its child, which has two children, both of which are span elements.
The span elements each display a constant string of text.

You can easily add some CSS classes to make your HTML tree look nicer
when it appears on screen:

{% highlight go %}

var (
	row = s5.NewCssClass("row")
	offset1 = s5.NewCssClass("col-sm-offset-1")
	col10 = s5.NewCssClass("col-sm-10")
)

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

It is customary to put any CSS classes "first" in the list of decendents
from a given node.  We have done that above in adding the "row" class
to the top level node and two [bootstrap](http://getbootstrap.com/css/) 
classes to the inner div. 

This should make it clear that in fact the _type_ of elements passed
as parameters to a call like DIV() or SPAN() is dealt with at run-time.
There are many different types of things that can be present in the 
"child list" to a call that creates a node.  Let's bind a click handler 
for the span "foo":

{% highlight go %}

var (
	row = s5.NewCssClass("row")
	offset1 = s5.NewCssClass("col-sm-offset-1")
	col10 = s5.NewCssClass("col-sm-10")
)

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

It is customary to put event handlers such as the one above immediately
before any child nodes.  If they are placed after child nodes, it becomes
hard to see which element the event handler is bound to.

Once you have your tree constructed in code, you typically call Build()
on it and add that value to some fixed place on the screen. For example:

{% highlight go %}

var (
	row = s5.NewCssClass("row")
	offset1 = s5.NewCssClass("col-sm-offset-1")
	col10 = s5.NewCssClass("col-sm-10")

	parent = s5.NewHtmlId("div", "item-parent")

)

tree:=s5.DIV(
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
).Build()
parent.Dom().Append(tree)
 
{% endhighlight  %}

### Binding values from the model

In the client-side code (`client/post/index.go`) for the index page, we
want to display the title on screen based on the model that 
represents what was retreived from the server. 

Given what we have just learned about tree building, here is a real
example, part of Add(),  one of the two methods (with Remove()) 
required by the Joiner interface that we are implementing.

{% highlight go %}
func (self *indexPage) Add(i int, m s5.Model) {
	model := m.(*postModel)

	tree := s5.DIV(
		s5.Class(uicommon.Row),
		s5.DIV(
			s5.Class(uicommon.BlogEntry),
			s5.Class(uicommon.Col12),
			s5.ModelId(model),
			s5.DIV(
				s5.Class(uicommon.Row),
				s5.Class(uicommon.Shaded),
				s5.A(
					s5.HtmlAttrConstant(s5.HREF, shared.URLGen.PostView(model.orig.Id)),
					s5.Class(uicommon.Col10),
					s5.Class(uicommon.H3),
					s5.TextEqual(model.title), //#1
				),
				s5.SPAN(
					s5.Class(uicommon.Col2),
					s5.Class(uicommon.TopSpace),
					//button section omitted for space
				),
			),
			s5.DIV(
				s5.Class(uicommon.Row),
				s5.Class(uicommon.Shaded),
				s5.SPAN(
					s5.Class(uicommon.Col12),
					s5.Class(uicommon.H5),
					s5.SPAN(
						s5.TextEqual(model.authorName), //#2
					),
					s5.SPAN(
						s5.Text("@"),
					),
					s5.SPAN(
						s5.TextEqual(model.date),  //#3
					),
				),
			),
			//blog content section omitted for space
		),
	).Build()
}

{% endhighlight  %}

The first critical line is the first one where we extract the particular
model we are expecting.  We cannot get a "tighter" type on the second parameter
of Add() because we do not know the types in your program.  However, since you
do know the types, you can safely cast the second parameter to the type that
you know was added to the collection.  Non-homogenous collections are possible
but not a subject of this tutorial.

Three other critical lines are the three calls to s5.TextEqual().  The
TextEqual() method takes an StringAttribute and binds its content
to the visible portion (textual part) of the tag it is nested inside.
Because this uses [constraints](#constraints), if you change the value
in the model, the screen will be updated appropriately. 

Sometimes it helps to squint.  If you squint when looking at the tree
being constructed above, you can see that it forms one large parent
div tag that forms a row.  The row has a child that is blog entry and
covers the whole width of the row ([Col12](http://getbootstrap.com/css/#grid)).
The blog entry has two children, both DIVs, each of which is a shaded
row...

## Handling the index method for posts

Returning to our [three steps](#ajax-display-seq),  fresno 
loads the blog posts over the network and displays as per our steps.
First it uses
the [ajax idiom](#ajax-idiom) in Start() (in `client/post/index.go`) to
request the posts.  At this point, it doesn't try to do pagination.
If the results are returned successfully, it receives a slice of pointers
to shared.Post() (`shared/post.go`), a.k.a. a slice of wire types.  
It then calls addPost() which takes the new shared.Post() object and 
converts it to a postModel, and finally adds that to the collection 
of posts.

It is worth noting that the posts collection has to now be initialized
when we create the page object.  This collection is "per-page state"
that exists during the entire time the page is visible in the browser.
We mention this per-page state because it is easy to forget about the
fact that there is code that must be holding state to make the UI respond
to various actions by the user, or data recevied via the network. 
Because of constraints, you typically can just build your Seven5 data
structures and then _let it run_.

<a name="create-post"/>

# Creating a New Blog Post

## Setup for this lesson
You'll need to have fresno running for this lesson.

## New Post

Make sure you are logged in as someone.  The "New Post" button on the UI is 
shown to logged in users (more on that later).  Click on the New Post button
and enter some text into the form.  Here is an example that demonstrates that
you can use [markdown](http://en.wikipedia.org/wiki/Markdown).

<img src="/assets/img/newpost.png" hspace="30" vspace="30" 
alt="the new post page" 
style="border:1px solid black; width:80%; height:80%;"/>


Once you have entered the new post, click the Post button in blue at the bottom
left.  You should see _part_ of your new post in the list on the 
[index page](http://localhost:5000/).

<img src="/assets/img/posts2.png" hspace="30" vspace="30" 
alt="the new post page" 
style="border:1px solid black; width:80%; height:80%;"/>

## Sharing code

There isn't much "new" to learn from the particulars of how the client side
posts the content from the form and gets the result.  This works in much the
same way as the [signup form](#signup-form).  The code can be found in
`client/uicommon/editpost.go` in the method SubmitPost(). 

Of potentially more interest is the fact that the code is in uicommon in a 
file called editpost.go.  This is because the edit a particular post page and 
the create a new post page are very similar.  The only significant different
from the user's point of view is that the post's title and content are 
already filled in when editing an existing post.  From the code's standpoint
there is the user-visible difference plus the fact that the REST verb to
be used is POST for creating a new post and PUT for updating an existing post.
Finally, the code must also perform a different action immediately after
a successful new post (return to index page) compared to a successful edit
post (return to show post page).

It is a common problem that you have two very similar pages that need to
share a good bit of implementation.  The approach used by fresno is to create
a very weak "class" in the 
[Object Oriented](http://en.wikipedia.org/wiki/Object-oriented_programming)
sense.  The "class" is defined as this struct in `client/uicommon/editpost.go`:

{% highlight go %}
//This is per page state.  This is common between the "new post" and
//"edit post" functionality.
type EditPostPage struct {
        *StandardPage

        Title       s5.StringAttribute
        Content     s5.StringAttribute
        SetupFunc   func()
        SuccessFunc func()
        NetworkFunc func(*shared.Post) (chan interface{}, chan s5.AjaxError)

        //private stuff
        errRegion       s5.HtmlId
        errText         s5.HtmlId
        button          s5.HtmlId
        cancel          s5.HtmlId
        titleInput      s5.HtmlId
        contentTextArea s5.HtmlId
        disabled        s5.CssClass
}

//Create a new post
func NewEditPostPage(curr string, setup func(), success func(),
        network func(*shared.Post) (chan interface{}, chan s5.AjaxError)) *EditPostPage {
//omitted for space
}

{% endhighlight %}

You can see from the structure definition that there are three functions, SetupFunc,
SuccessFunc, and NetworkFunc.  These allow the callers of NewEditPost to insert (again, weakly)
subclass-like behavior into the EditPostPage implementation.  For example, the
relevant code for the new post page is in main() of `client/post/new.go` where
these three functions are passed as literals:

{% highlight go %}

func main() {

        // create a new instance of the edit/new post page with our choices
        // that are right for new
        ep := uicommon.NewEditPostPage(shared.URLGen.IndexPage(), func() {
                //nothing to do
        }, func() {
                uicommon.SetCurrentPage(shared.URLGen.IndexPage())
        }, func(p *shared.Post) (chan interface{}, chan s5.AjaxError) {
                return s5.AjaxPost(p, shared.URLGen.PostResource())
        })

        //navigation code omitted for space

        s5.Main(ep)
}
{% endhighlight %}

There is a similar but slightly more complex version of this call to 
uicommon.NewEditPostPage() in `client/post/edit.go`.  It is slightly more
complex because the content to initialize the interface with must be 
fetched from the server.  The contrast, the code above has the empty function 
as its setup function.

## Slugs

If you look at the two screen shots from earlier in this lesson, you'll notice
that the text entered in the first was substantially larger than what is shown
in the second.  You can click on the title of a blog post to see the full
blog post or you can use the url /post/1/view or similar. 

In `client/uicommon/editpost.go` in the method SubmitPost() there is some 
code that attempts to "break" a long post into two parts, the slug portion
that is shown on the index page and the rest.  It tries to break after a 
newline, if possible. The effects of this can be seen in the next lesson.

<a name="posts-p2"></a>

# Display Blog Posts, Part 2

## Preparation for this lesson
You'll need to have fresno running for this lesson.


## Markdown processing

In the implementation of the post resource (`resource/post.go`) there are
three different ways that markdown is handled.  We will examine each in turn.

### Index

{% highlight go %}
func (self *PostResource) IndexQbs(pb s5.PBundle, q *qbs.Qbs) (interface{}, error) {
	//database query and pagination omitted for space
	for _, p := range posts {
		author := p.Author
		author.Password = ""
		p.TextShort = markdown(p.TextShort)
		p.Text = markdown(p.Text)
	}
	return posts, nil
}
{% endhighlight %}

In the above code from IndexQbs() you can see that the function markdown()
is called on both the short (slug part) and long part of the text retreived
from the database.  The text stored in the database is always the raw, unprocessed
code.  The markdown() function uses [blackfriday](https://github.com/russross/blackfriday)
to process the markdown and [bluemonday](https://github.com/microcosm-cc/bluemonday)
to sanitize the user generated code.  Between these two, the content of the
wire type sent back to client has its TextShort and Text fields filled in with
_HTML_ code, not the original plain text that was submitted in the form.

This code is actually quite wasteful in that the larger portion of the text that
gets processed (the Text field) is not used by the client side and ends up
being discared.

### Find 

{% highlight go %}
func (self *PostResource) FindQbs(id int64, pb s5.PBundle, q *qbs.Qbs) (interface{}, error) {
	var p shared.Post
	p.Id = id
	err := q.Find(&p)
	if err != nil && err == sql.ErrNoRows {
		return nil, s5.HTTPError(http.StatusNotFound, fmt.Sprintf("did not find %d", id))
	}
	if err != nil {
		return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("error in qbs find: %v", err))
	}
	//don't show the password
	p.Author.Password = ""

	_, wantMarkdown := pb.Query(shared.MARKDOWN_PARAM)
	if wantMarkdown {
		p.Text = markdown(p.TextShort + p.Text)
		p.TextShort = ""
	}
	return &p, nil
}
{% endhighlight %}

The other two cases are both in the FindQbs() method of the same resource.  These
two are differentiated by the query parameter markdown, as you can see with
the call to pb.Query("markdown") that checks to see if that parameter is
present (and ignores any value it has). In the case of displaying an existing
post for editing, the client side wants the original, unprocessed text.  In
the case of displaying a full post, with formatting, the client wants all
the markdown code processed.   This is accomplished by concatenating the
TextShort and the Text field when calling the markdown() function. The
concatenation is necessary because the "split" may have done a poor job of
choosing the split point and while this ok for the index page since
it just shows the "small" portion, but we do not want to have a strange line break
in the display of the full, formatted blog entry.

### Query Parameters

The code above demonstrates how to handle query parameters on the server-side,
via the PBundle.  It is worth noting that the query parameter _generation_ is
done on the client side through the URLGen object (`shared/urlgen.go`).  The method that 
generates a URL for the rest resource post and a particular id looks like this:

{% highlight go %}
//compute the rest resource url for a given post
func (u *urlgen) Post(id int64, wantMarkdown bool) string {
	base := u.PostResource() + "/" + fmt.Sprint(id)
	if wantMarkdown {
		base += "?" + MARKDOWN_PARAM + "=true"
	}
	return base
}
{% endhighlight %}

Since the client and server both compile all the code in the shared directory,
including URLGen, it is impossible for the query parameter that is being
used here, MARKDOWN_PARAM, to get "out of sync" with the code on the 
server side that uses it in FindQbs.

## Displaying HTML

In two places, we must display already created.  We use our 
[tree building](#treebuilding) approach to inserting the already created
HTML code into the DOM.  Let's look at the code that displays a full
blog post in `client/post/view.go`.  In displayPost():

{% highlight go %}
func (self *viewPostPage) displayPost(p *shared.Post) {
	tree :=
		s5.DIV(
			//heading rows elided for space
			s5.DIV(
				s5.Class(uicommon.Row),
				s5.SPAN(
					s5.Class(uicommon.Col12),
					s5.HtmlConstant(p.Text),
				),
			),
		).Build()
	postParent.Dom().Append(tree)
}
{% endhighlight %}

The critical line here is the call to HtmlConstant().  This is a library
routine which inserts already created HTML into the DOM as the child
of the DOM node which contains the call.  This suffixed with "Constant" to
indicate that this cannot be constrained to an attributes value, it must
be a blob of text.  It seems possible to build variant of this that would
be called HtmlEqual() that could be used with constraints an attribute
string value to keep an html section of the page always up to date.  However,
the utility of such of function is dubious, since a more controlled version of
this functionality is already possible via the normal tree building mechanism 
and, more generally, inserting big blobs of uninterpreted HTML into the 
DOM is questionable, be it constant or otherwise.

<a name="control-urlspace"></a>

# Controlling The URL Space

## Preparation for this lesson
You'll need to have fresno running for this lesson.

## URLs to static files

You should try typing in some different URLs to see what effect they have on 
fresno.  Here are some samples (you need to add the http://localhost:5000 
in front, of course) for the index page.

* /
* /posts  (xxx fix me)
* /posts/
* /posts/index.html
* /posts/index
* /en/web/index.html (this will 404, Not Found)
* /en/web/post/index.html
* /en/web/posts/index.html
* /en/web/posts/
* /en/web/posts (xxx fix me)

For the first blog post:

* /post/1 (xxx fix me)
* /post/1/
* /post/1/view
* /post/1/view.html

All of the URLs in the first set correspond to a single static file,
`static/en/web/post/index.html`.  Note that this is the singular "post"
in the path, not the plural.  All of the URLs in the second group 
correspond to the single static file `static/en/web/post/view.html`.

The mapping between URLs that a user types, or links they traverse, to
static files is important for three reasons:

* It is nice for URLs to be "pretty" such as /posts/1/view to show the
first post.  This pretty URL space also aids in debugging since one can
easily construct URLs that correspond to particular back-end entities.

* We don't want to duplicate static files, despite the fact that there may
be multiple ways in the URL space to access the same logical entity.

* We want to generate HTTP code 304 for static files when possible.  In 
particular, we want a url like /post/1/view to not load the fixed part,
the HTML content, in `static/en/web/post/view.html` extra times if it has
not changed.

### /en/web
The /en/web prefix is currently a placeholder.  The "en" represents a language
(English) and the "web" represents a display device (standard web browser).
In the future, Seven5 intends to support automatic language selection based on
the browser settings and automatic device selection based on browser settings,
such as switching to "/en/mobile" for a mobile phone-based browser.


## StaticComponents

The mechanism in Seven5 for handling the mapping between the URL space and
the filesystem (really the `static` directory) is 
[StaticComponent](https://gowalker.org/github.com/seven5/seven5#StaticComponent).  
This is only for user-visible URLs, not URLs in the REST namespace ("/rest").   
In the  setup() function of fresno (`fresno/main.go`) there are four statements 
of configuration related to StaticComponents:

{% highlight go %}
	//what do we do if given empty URL, note we use the SINGULAR here
	homepage := s5.ComponentResult{
		Status: http.StatusMovedPermanently,
		Redir:  "/posts/index.html",
	}
	postComponent := s5.NewSimpleIdComponent("post", existCheck, newCheck, viewEditCheck)
	indexComponent := s5.NewIndexOnlyComponent("posts", "post/index.html") //SINGULAR for index

	result.matcher = s5.NewSimpleComponentMatcher(result.cm, result.sm, staticDir,
		homepage, result.heroku.IsTest(), postComponent, indexComponent)

{% endhighlight %}
The first statement creates a 
[ComponentResult](https://gowalker.org/github.com/seven5/seven5#ComponentResult) 
that specifies what is to occur when the user comes to the homepage (URL is "/"). 
The  middle two lines shown create StaticComponents.   The last statement uses those 
two components as initialization parameters (the last two) in creating a 
SimpleComponentMatcher. 

[SimpleComponentMatcher](https://gowalker.org/github.com/seven5/seven5#SimpleComponentMatcher)
is a utility for serving up static files from a static
directory (note the staticDir parameter to 
[NewSimpleComponentMatcher](https://gowalker.org/github.com/seven5/seven5#NewSimpleComponentMatcher)).
It has many options but we are primarily interested in how it uses the homepage,
postComponent, and indexComponent local varibles.

### ComponentResult

The variable homepage is an instance of the type ComponentResult.  A 
[ComponentResult](https://gowalker.org/github.com/seven5/seven5#ComponentResult) 
is used when a StaticComponent wishes to communicate with the matcher.
It has a number of fields, but generally the most important ones are Status,
Redir, and Path.  The Status field is an HTTP response code.  If the Status
code is 200 (OK) then the Path field is consulted for what static content
to try to serve.  If the Status field is 302 (MovedPermanently) then the Redir field
is consulted what URL the browser should be redirected to. Any other status
value is returned to the browser as unchanged, although this usage is 
rare.

It is important to understand that the Components operate _before_ other
processing.  In particular, if the user requests a URL such as 
"/post/1/hackyhack.html" the ComponentResult returned is likely to have
Status 200 with a path of "/en/web/post/hackyhack.html".  This will likely
fail to serve, and ultimately the user will receive a 404.  However, from
the Components point of view, that is a perfectly valid URL and so it
returns 200 to the matcher and the matcher ultimately sends the bad news
to the client.  Without this behavior, it becomes problematic to have
html files that have referencs to support files such as css or javascript
scripts.

### SimpleIdComponent

[SimpleIdComponent](https://gowalker.org/github.com/seven5/seven5#SimpleIdComponent) 
is a utility for creating a Component that understands URLs
like /foo/123/verb.  The parameters provided to 
[NewSimpleIdComponent](https://gowalker.org/github.com/seven5/seven5#NewSimpleIdComponent) 
are the part of the URL space ("post" in our example) to use and three functions
that provide "checks" on the URL.  These are the 
[ExistCheck](https://gowalker.org/github.com/seven5/seven5#ExistsCheck), 
[NewCheck](https://gowalker.org/github.com/seven5/seven5#NewCheck), and
[ViewEditCheck](https://gowalker.org/github.com/seven5/seven5#ViewEditCheck).

The first of these allows a function to provided
that can check for the existence of a particular id, such as 123.  If it
fails, the Component will indicate that a 404 should immediately be genarated.
If it succeeds, processing simply continues.  If nil is provided as this 
check function, processing continues.

The newCheck and viewEditCheck allow some basic permission checking on URLs
such as /foo/123/new or /foo/123/edit.  These functions in the case of 
fresno have the same name as their check, and are in `fresno/main.go`:

{% highlight go %}
//only logged in users can post
func newCheck(pb s5.PBundle) (bool, error) {
	//elided for space
}

//anybody can view a post but to edit you have to be an admin or
//you have to be the owner of the post
func viewEditCheck(pb s5.PBundle, id int64, isView bool) (bool, error) {
	//elided for space
}
{% endhighlight %}

If either of these functions returns a false, processing stops and the matcher
returns a false, the user's browser will receive a 401 response code 
(Unauthorized).

#### CookieMapper and SessionManager

If you are wondering why the parameters to 
[NewSimpleComponentMatcher](https://gowalker.org/github.com/seven5/seven5#NewSimpleComponentMatcher)
included the application's 
[CookieMapper](https://gowalker.org/github.com/seven5/seven5#CookieMapper)
and [SessionManager](https://gowalker.org/github.com/seven5/seven5#SessionManager), 
the reason is the [PBundle](https://gowalker.org/github.com/seven5/seven5#PBundle)
parameter to the functions above.  To create a proper PBundle, which may 
include session data encoded in a cookie, the matcher muster interact with
the CookieMapper and SessionManager.   Put another way, to correctly understand
if the url /foo/123/edit is an acceptable URL, we may need to know who this
user is.

### IndexComponent

In support of making URLs "pretty", we can create a component that is responsible
for a given name plus optionally the suffix index.html.  This permits the
URLs /posts/ and /posts/index.html to map to `static/post/index.html`.  It is
conventional to use the singular for the path, despite the URL because it
keeps index.html with all the other html files associated with a (singular)
post, such as "new.html" or "edit.html"

## SimpleComponentMatcher

The type 
[SimpleComponentMatcher](https://gowalker.org/github.com/seven5/seven5#SimpleComponentMatcher) 
is responsible for taking in a URL for 
content (not a rest resource) and finding the static file to serve.  In
considering the call to NewSimpleComponentMatcher() at the beginning of
this lesson, we have discussed the first two parameters the CookieMapper
and the SessionManager.  These are needed to support the creation of 
parameter bundles for verifying access to a particular URL.  The third
parameter is a directory (path) that contains the static content, typically `static`.
The fourth is a hard-coded ComponentResult that should be used when the user
requests the URL /.  This is sufficiently common and important it was
prometed to being a separate parameter.  The fifth parameter is a boolean
that indicates if the matcher should be in "test mode", and if it is in
test mode, it will allow access to URLs that start with /gopath, to aid
in debugging.  Any following parameters are Components to be used in 
matching against URLs.


# Oauth 2
To be done.

# Testing
To be done.

# Internationalization
To be done.

# Building a single image and pulling assets from binary
To be done.


EXERCISES
=========

* Disabled accounts
* GOROOT visible in browser
* Generator for wrapping around ajax methods and getting the types right
* Generate for creating a new "page" in the application
* Add a dialog box for "are you sure you want to cancel" if post has changes
* Pagination 
* Logout




