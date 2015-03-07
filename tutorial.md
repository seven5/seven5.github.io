---
title: Setting Up For Seven5 Development
layout: tutorial
---

<a name="start"></a>

# Fresno: The Seven5 Tutorial

This tutorial builds a blog engine called Fresno. The world does not need
more blog engines; Seven5 needs a tutorial based around an application
that has the minimal amount of application-specific things for you to learn.

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

Meta-information and commentary about the tutorial itself is written like
this:

> We would love to have somebody test out this tutorial on windows
> and provide a pull request with the necessary changes to get it working.

## Prerequisites

This document assumes you understand 

* go 
* the basics of the web (http, html, css)
* the basics of git

Further, this tutorial assumes you are developing on a Mac using OSX or 
a similar-enough Linux installation. 

You will need to have a version of go installed that is at least 1.4. On
OSX the recommend way to do this is via brew.

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
`TUTROOT`.

Clone the tutorial repository:

{% highlight bash %}
$ cd
$ cd tutroot/src
$ git clone git@github.com:seven5/tutorial.git 
$ cd tutorial
$ git checkout lesson-setup
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

The output of the last command should be verison 1.4+ and appropriate
for your operating system. 

<a name="godep-dance"></a>

## How to use godep with Seven5

[Godep](http://github.com/tools/godep) is an important, but tricky to use, 
tool.  For this tutorial, you will be using godep to "vendor" any and all
depencies you are using and commit them to git.  However, you have to have
a way to "set things up" so that godep can do its work.

The `external` directory (aka `TUTROOT/external`) is part of `GOPATH` and 
is the *staging* area for godep.  It also contains external tools you will 
be using so  `external/bin` needs to be in your path.  Let's start by 
getting godep itself installed.

{% highlight bash %}
$ go get github.com/tools/godep
$ which godep
TUTROOT/external/bin/godep
{% endhighlight %}

Now let's install Seven5 with go get:

{% highlight bash %}
$ go get github.com/seven5/seven5
{% endhighlight %}

At this point Seven5 is installed in 
`TUTROOT/external/src/github.com/seven5` but it's not part of your 
project yet (can't be committed to git).  You need to "copy in" dependencies.

{% highlight bash %}
$ godep save tutorial/fresno-dummy-main
$ cat Godeps/Godeps.json
{% endhighlight %}

The output of the last command is the dependencies of the dummy project
`fresno-dummy-main` which is used as an example here. You'll notice that
this includes Seven5 itself plus all the things it depends on, recursively. 

You may want to look at `fresno-dummy-main/main.go` to see the imports
that are read by godep.  We use the "import _" trick to force godep to
pull in the Seven5 dependencies.

You should be able to build and run the dummy main program like this:

{% highlight bash %}
$ godep go install tutorial/fresno-dummy-main
$ fresno-dummy-main
2015/01/24 11:06:01 Seven5 is now vendored
{% endhighlight %}

Note that the build command builds using godep, not go directly.  This means
that you are using the vendored godeps in the `Godeps/workspace/_src`
directory.  If you for some reason want to _not_ use the vendored 
godeps, your enable script sets `GOPATH` so just 
"go install tutorial/fresno-dummy-main" will work.  Use of this technique
is not recommended as you want to be building/running locally on
precisely the same things that are going to be used in production, which
is the contents of the `Godeps` directory.



### Godep workflow in Seven5

When doing development, the pattern for dealing with godeps is:

* go get -u blah
* rm -rf Godeps
* godep save myprogram
* [test myprogram to make sure everything is ok]
* git add -A Godeps
* git commit

This "staging" area allows you to experiment with different version of various
packages and then "godep save" a snapshot.  Once you are satisfied with the
snapshot you commit that to the repository.

Because of the use of the external staging directory, 
it is *always* safe to delete the Godeps directory and "try again" 
with "godep save" if something gets messed up in your project.

> Yes, it's a pain in the tutorial but in steady state, on a real project,
> this issue only comes up when you are changing the versions of dependent
> libraries.  In that case, the clear separation of staging is a win.

<a name="simple-server"></a>

# Creating And Deploying A Simple Server

For this lesson and all the ones following, we'll assume that you can
get the godeps set up correctly (it was explained in the 
[previous lesson](#setup) if you are jumping around). 
These are not checked into the tutorial source code and 
anytime you change lessons, you'll probably have to "go get" 
some dependencies and/or "godep save" to copy them into
the lesson working tree.  The go compiler will quickly tell you 
what dependencies you need and don't have if you forget!

## Preparation for this lesson

For this lesson, you'll need to install the lesson source code,
the sources to 
[gopherjs](http://www.gopherjs.org) and to save the dependencies.
In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-simple-server
$ go get github.com/gopherjs/gopherjs
$ godep save tutorial/fresno
{% endhighlight %}


You'll need to insure that you have the [heroku toolbelt](https://toolbelt.heroku.com) installed on your system.  You should have an account with
heroku, try using "heroku login"

## Environment variables

All configuration of Seven5 applications is done through environment
variables.  This makes them [twelve factorish](http://12factor.net) and
easy to deploy on heroku.  If you look at the enable script, there are
now some new variables `HEROKU_NAME` and `PORT` that are used to 
help initialize the app.  The variable `FRESNO_TEST` is *only* set in
the case of local development.

## Create heroku app

> It would be great if somebody could submit a pull request that
> has deployment support for [dokku](http://progrium.viewdocs.io/dokku)
> running locally or [tutum](https://www.tutum.co).

You'll need to create a new heroku app so you can deploy your copy of
fresno.  You may need to login to authenticate your account with the
heroku toolbelt, that's not shown below.

{% highlight bash %}
$ heroku apps:create
Creating damp-sierra-7161... done, stack is cedar-14
https://damp-sierra-7161.herokuapp.com/ | https://git.heroku.com/damp-sierra-7161.git
{% endhighlight %}

You should take the output name you have received from this creation
step and put it in the enable script as `HEROKU_NAME`.

If you are wondering why the name of the application's name 
must be configured, it is because it often the case that applications 
need to generate a  _full_ URL that points to themselves, notably when 
doing oauth. 

## Build and run the application locally

{% highlight bash %}
$ godep go install tutorial/fresno
$ fresno
2015/01/24 11:42:34 [SERVE] waiting on :5000
{% endhighlight %}

You can now go to [localhost:5000](http://localhost:5000/) to see
the application and confirm it works.  You'll see "it's alive! bwah, haha!"
on your screen.

## Create a local branch for experimenting

You'll need to use git to create a local branch of tutorial so you can
commit the necessary godeps and have a branch to push to heroku. 
You may want to do this anyway if you have changes to the enable script 
or want to play with the source code.

{% highlight bash %}
$ git checkout -b my-simple-server
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}


## Build and run on heroku

You first need to tell heroku about the buildpack for Seven5 and the app's
own name (change this to the name you got when you created your app). 
After that "git push" is your deployment mechanism:

{% highlight bash %}
$ heroku config:set BUILDPACK_URL=https://github.com/seven5/heroku-buildpack-seven5.git
$ heroku config:set HEROKU_NAME=damp-sierra-7161
$ git push heroku my-simple-server:master
[bunch of output about the build procedure]
remote:        https://damp-sierra-7161.herokuapp.com/ deployed to Heroku

{% endhighlight %}

You'll notice that the branch "my-simple-server" is being pushed to "master"
on heroku because heroku only builds on pushes to master. 

> If you see an error like this:
<pre>
! [rejected]        my-simple-server -> master (non-fast-forward)
error: failed to push some refs to 'https://git.heroku.com/damp-sierra-7161.git'
</pre>
> it is usually because you have re-written your git history and you need
> to use force (-f) with the git push to get heroku to accept the push.

You should now be able to visit "https://damp-sierra-7161.herokuapp.com/" 
(with your app's name) and see the same output that you receive locally.

<a name="database"></a>

# Add A Database
Most realistic applications need access to reliable relational store.
Fresno is no exception, so we'll go ahead and set this up now.

## Preparation for this lesson

For this lesson, you'll need to install the lesson source code,
the sources to [qbs](http://github.com/coocood/qbs) and 
[pq](http://github.com/lib/pq) and to save 
the dependencies. It's also a good time to create a branch for your 
work.  In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-add-database
$ go get github.com/coocood/qbs
$ go get github.com/lib/pq
$ godep save tutorial/fresno
$ git checkout -b my-add-db
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

You'll need to have a copy of postgres running on your local system
for development.  On a mac, you can do this `brew install postgres`
and then follow its directions about how to run the database.  On a
linux system, use your package manager (yum, apt-get, pacman, or similar)
to install the postgres server. 

This document assumes postgres version 9.3, but other versions in the
9.x series will likely work.

## Add postgres your heroku app

> It would be great if somebody could try to get these applicaitons work
> under mysql since both qbs and heroku support that relational system.

You can add the production database to your heroku app like this:

{% highlight bash %}
$ heroku addons:add heroku-postgresql
$ heroku config
[configuration output]
{% endhighlight %}

You'll notice in the output of the "heroku:config" now includes a
`DATABASE_URL`.  There is now a corresponding entry in the enable
script for the local development case.

{% highlight bash %}
$ cat enable-tutorial
[ lots of settings]
export DATABASE_URL="postgres://$USER@localhost:$PGPORT/fresno"
{% endhighlight %}

## Local database setup

On your local system you'll need to create the database "freso"

{% highlight bash %}
$ createdb fresno
{% endhighlight %}

This is a reasonable sanity check that your database server is up and
your settings in the enable script are correct.  If everything is
working properly, you'll receive no output from this command.

Note that heroku creates the database user, database password, and 
database name for you and these are all "hard to guess" random values.
Because all the configuration information is drawn from the 
`DATABASE_URL` you can run the same go code on both your local system 
and on heroku (and they may very well be different operating systems
if you are developing locally on OSX).

<a name="psql"></a>

#### PSQL

If you are familar with SQL, you can get an SQL prompt with
{% highlight bash %}
$ psql $DATABASE_URL
{% endhighlight %}
on your local system or for the remote database on heroku:

{% highlight bash %}
$ heroku pg:psql
{% endhighlight %}

> By-hand updates to the "production" database on heroku may
> be an exceptionally bad idea.

<a name="migrations"></a>

# Migrations

With a web server, a database, all our dependencies captured in our
git repository, and a path to production deployment
we are now ready to start doing some real development.

## Preparation for this lesson

In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-migration
$ godep save tutorial/...
$ git checkout -b my-migration
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

Note that now we have more than one program, so we use "tutorial/..." as the 
argument to "godep save".

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
	},
	Down: map[int]migrate.MigrationFunc{
		1: oneDown,
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

If you look at the up migration, you will notice that the "oneUp" migration
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
types that you can express in the go structures is limited.

The next lesson will discuss what structures in go correspond to the
tables created.

## Building and running the migrations locally

You can build and run the migration application like this:

{% highlight bash %}
$ godep go install tutorial/migrate
$ migrate --up
[migrator] attempting migration UP 001
001 UP migrations performed
$ migrate status
current migration number is 001
{% endhighlight %}

You may find it interesting to use "psql" (see [previous lesson](#psql)) 
to look at what's in the database after you this command. 
The reverse also works:

{% highlight bash %}
$ godep go install tutorial/migrate
$ migrate --down
[migrator] attempting migration DOWN 001
001 DOWN migrations performed
$ migrate --down
at earliest migration, nothing to do
{% endhighlight %}

Again, you may find it interesting to look inside the database at the result
of doing "migrate --down".   There are options you can pass to the 
migrate program to run a specific number of up or down migrations with 
the "--step" flag.

## Building and running the migrations on Heroku

If you push this version of the code to heroku, you can get a bash shell on
the remote (heroku) machine and then use the migrations just as with the 
local case.

{% highlight bash %}
$ git push -f heroku my-migration:master
$ heroku run bash
Running `bash` attached to terminal... up, run.4328
$ migrate status
current migration number is 000
$ migrate --up
[migrator] attempting migration UP 001
001 UP migrations performed

{% endhighlight %}

<a name="simple-rest"></a>

# Serve Up Some Data Through A REST API

We've got a web server and a database that has some content in it, so 
let's create a REST API to access the "user_record" and "post" tables.


## Preparation for this lesson

In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-simple-rest
$ godep save tutorial/...
$ git checkout -b my-simple-rest
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

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
in the `shared` package.  Typically, these structures contain only very
simple data types as fields because serializing complex types to json
can be problematic.  Because our application is simple now, we are *also*
going to use these same structures as the data to be represented in
the database.  This double duty is nice, because it insures that the 
values exchanged over the wire are "in sync" with the underlying data
model, but more complex application typically need to separate the
storage layer (represented by structures used with Qbs and the database)
and the wire representation (that is shared between client and server).

## The wire types

The wire types are the "nouns" in the sense of RESTful API design. 
For this lesson, these are are in `shared/post.go` and 
`shared/user_record.go` and coded as uppercase, singular nouns ("Post").
Because they are in a separate package and because they must be
serializable with "encoding/json" the fields must be uppercase also.

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

As we discussed in the [previous lesson](#caveat-data-type) the fields
in the structures "UserRecord" and "Post" must match up to the columns
in the database tables "user_record" and "post".  The primary key field
must be called out to Qbs if the "pk" is a string, and it is good practice
to do it in any case, even though Qbs would default to choosing the 
int64 field Id for Post. 

You'll notice that we do a join on the post's author to a user record.
This is done by default by Qbs at retrieval-time and we've just accepted
that default here and told Qbs where to put the joined record.

## Building and testing locally

Compile and start the server running locally:

{% highlight bash %}
$ godep go install tutorial/...
$ fresno
{% endhighlight %}

<a name="curl-to-server"></a>

In another shell window, try:

{% highlight bash %}

$ curl localhost:5000/rest/userrecord/515f7619-8ea2-427f-8cf3-7a9201c747dd
{
 "UserUdid": "515f7619-8ea2-427f-8cf3-7a9201c747dd",
 "FirstName": "Mary",
 "LastName": "Jones",
 "EmailAddr": "mary@example.com",
 "Password": "",
 "Disabled": false,
 "Admin": true
}
$ curl localhost:5000/rest/userrecord/blah
did not find blah
$ curl localhost:5000/rest/post/1
{
 "Id": 1,
 "Title": "first post!",
 "Updated": "2015-01-24T16:33:16.970088-08:00",
 "Created": "2015-01-24T16:33:16.970088-08:00",
 "Text": "This is the first post on the site!",
 "AuthorUdid": "df12ba96-71c7-436d-b8f6-2d157d5f8ff1",
 "Author": {
  "UserUdid": "df12ba96-71c7-436d-b8f6-2d157d5f8ff1",
  "FirstName": "Joe",
  "LastName": "Smith",
  "EmailAddr": "joe@example.com",
  "Password": "",
  "Disabled": false,
  "Admin": false
 }
}
{% endhighlight %}

If you are curious, you may find it interesting to try changing the
curl command to "curl -v" so you can see the details of the error
messages, headers, etc.

## Building and testing on heroku

The build and test cycle for heroku won't be called out any further in
this document unless there is a deviation from the "normal" deployment
of:

{% highlight bash %}
$ git push -f heroku my-simple-rest:master
[deployment info]
$ curl https://damp-sierra-7161.herokuapp.com/rest/post/1
[json output]
{% endhighlight %}

You will need to adjust the application name and run migrations on heroku's
database if you haven't already.

> If you see an error like this when you push to heroku: 
<pre>
remote:  !     A .godir is required. For instructions:
remote:  !     http://mmcgrana.github.io/2012/09/getting-started-with-go-on-heroku
</pre>
> it usually means you forgot to check in your Godeps directory.

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

If you look in the "main" of the fresno application now (`fresno/main.go`),
you can see where the connection is made between a wire type and a 
resource implementation that implements methods (logically) for that 
wire type:

{% highlight go %}

	base.ResourceSeparateUdid("userrecord",
		&shared.UserRecord{},
		nil, //index
		s5.QbsWrapFindUdid(&resource.UserRecordResource{}, store),
		nil, //post
		nil, //put
		nil) //delete

	base.ResourceSeparate("post",
		&shared.Post{},
		nil, //index
		s5.QbsWrapFind(&resource.PostResource{}, store),
		nil, //post
		nil, //put
		nil) //delete

{% endhighlight %}

"base" in the above code snippets is a Seven5 
[BaseDispatcher](https://gowalker.org/github.com/seven5/seven5#BaseDispatcher)
that will parse an incoming request for a URL and dispatch it to the 
appropriate resource or subresource.  The dispatcher has different, but
similar looking methods, "ResourceSeparateUdid" and "ResourceSeparate" that
indicate if you want to use UDID-based resources or standard integer ones.

The first parameter given to both of these methods is the portion of the URL
space that this resource occupies. This should be singular and lower case;
omitting underscores is also preferred.  The second parameter is the 
resource's implementation type, and then there are five separate (thus the
"separate" in "ResourceSeparate") methods that provide the implementation
of the REST methods index, find, post, put, and delete. Each of these
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

If you look in `resource/user_record.go` you'll see the only implemented
method is "FindQbs".  This method's signature is interesting:

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

The _return_ values are less clear.  Sadly, we cannot get strong typing
on the first parameter, it is checked at run-time.  This must match the
wire type associated with this resource implementation (see above for
the association).  The latter argument is an error.  If that error is
of Seven5's [Error](https://gowalker.org/github.com/seven5/seven5#Error)
type, you can provide the http status code an error message.  If it is
different type of error, the client will receive an "internal error"
result. 

It is perhaps unsurprising that about half of the implementation of
the FindQbs method in both UserRecord and Post is error checking and
returning.


<a name="static-site"></a>

# A static site

Let's add the ability send fixed, unchanging files back from the
server to the client. 

## Preparation for this lesson
In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-static-site
$ godep save tutorial/...
$ git checkout -b my-static-site
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

You'll need to have a working copy of "make" installed on your system as
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
against server-generated HTML is to ask if the server can (correctly) generate a [304](http://httpstatusdogs.com/304-not-modified) 
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
$ godep go install tutorial/...
{% endhighlight %}

The command above builds all the tools (migrate and pagegen) and the fresno
application itself.

## Building index.html

The directory `pages` has all the static file content. You can go into
the `pages` directory and type `make` to build the content:

{% highlight bash %}
$ cd TUTROOT/src/tutorial/pages
$ make
pagegen --support=support --dir=template --start=index.html --json=index.json > ../static/en/web/index.html
$ make
make: Nothing to be done for `all'.
{% endhighlight %}

There is only one page at the moment, but let's examine that 
command that make ran carefully. 
The "base" html file is `pages/template/index.html` and 
the json data bundle for that page is `pages/template/index.json`. 

The `pages/template/index.html` file is:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
	{%raw%}{{template "BOOTSTRAP_HEAD_SECTION" .}}{%endraw%}

<body>
	<div class="container">

    <h3>Freso is a blog engine</h3>

	 {%raw%} {{template "MISC_FOOTER" .}} {%endraw%}
	
  </div> <!--container -->
	
	{%raw%} {{template "MISC_JSLOAD" .}}{%endraw%}
</body>
</html>
{% endhighlight %}

The `pages/template/index.json` file is:
{% highlight json %}
{
	"title":"Fresno",
	"css_page": "/fixed/index.css"
}
{% endhighlight %}

The "template" for our index.html page is "merged" with this json blob. 
The make command above referenced the support directory 
`pages/template/support`.  _All_ the templates found in this directory
(files ending in ".tmpl").  You can see the definition of the referenced
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

This is the "year" function defined in `pagegen/main.go` and shows you how
you can call arbitrary go code from within a template.  Note that this code
must be defined/built before the server is run.

All this work for a single output file is overkill at this point, but in
future lessons as we add more pages, and more complex pages, the advantage
of don't repeat yourself will become clear.  The careful will have noticed
the reference to `pages/template/support/json_helper.tmpl` in the source
code of `pagegen/main.go` and that this file is empty.  This template is
used to prevent repetition in the _json blobs themselves_ that are used 
to supply data the pages.  We don't need this yet, so the file is empty.

## The static directory

The output of pagegen, notably `static/en/web/index.html` in this lesson,
is placed in the `static` subdirectory.  That directory _must_ be checked
into git as it is the content your web server will respond with when 
queried for "/".  In other words, `static/en/web/index.html` will be
visible as "/en/web/index.html" when the server is running.

There are files in `static/fixed` that correspond to assets, like images
or css files, that don't change depending on the language of the user's 
browser or browsing device. Although we only support one language ("en") 
and one device ("web") right now, the content that might be dependent on 
these variables is separated into subdirectories.

## Run and test the static files

You should be comfortable enough now to build the fresno applications anytime
with "godep go install tutorial/...", so we won't call it out anymore.

From now on you should assume that you should run "fresno" from the primary
tutorial directory (`TUTROOT/src/tutorial` or the parent of `static`) because
this is how your application will be run on production. 

You can view the 
[index page](http://localhost:5000/en/web/index.html) in your web
browser on the local machine or [heroku](https://damp-sierra-7161.herokuapp.com/en/web/index.html). 

>If you are getting a 404 error in the local case
>it is probably because you are not running fresno in parent of `static`.


<a name="signup-form"></a>

# A Signup Form

Let's create a page which can handle the input of a new user signing up.
This form won't be "hooked up" to the server yet, but will show you how
to do some basic client-side, interactive content in go.

This lesson uses the "_fixed form_ strategy" for dealing with the 
DOM content. In this strategy, all the HTML ids ("div#foo") are known 
in advance  so it is easy to "hook up" the interactive parts of 
the application to  the form elements. In more complex interfaces, 
one cannot know all the  HTML ids in advance, because these are 
calculated at run-time and a different strategy is necessary.


## Preparation for this lesson
In the `TUTROOT/src/tutorial` directory:

You'll need to get a copy of 
[gopherjs](http://github.com/gopherjs/gopherjs) installed locally for
development. Gopherjs requires go 1.4. In the 
`TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ go get github.com/gopherjs/jquery
$ go get github.com/gopherjs/gopherjs
$ which gopherjs
TUTROOT/external/bin/gopherjs
$ git checkout lesson-signup-form
$ godep save tutorial/...
$ git checkout -b my-signup-form
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

## Form feedback

Once you have added the godeps above, you should be able to build the
application, run it, and 
[hit the server](http://localhost:5000/en/web/signup.html)
to get the `static/en/web/signup.html` page.

<img src="/assets/img/signup.png" hspace="30" vspace="30" 
alt="the signup page" style="border:1px solid black; width:80%;height:80%; float:right;">

You should see a page like the one at right. If you don't have the
error console open, as shown at the bottom of this
screen cap, you should open it now.  You'll need it.

Typing into the form should give you some feedback in the right-hand
portion of the screen.  Once you have filled in all the areas of the
form and there are no errors, the "Sign Up" button will become 
enabled.  Not that pressing it will do you any good in this lesson!

## HTML code changes

In the `pages` directory you'll notice there is now a 
`pages/template/signup.html` and corresponding `pages/template/signup.json`.

We have exploited our "Dont Repeat Yourself" mantra here.  The code for
the html page is:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
	{{template "BOOTSTRAP_HEAD_SECTION" .}}

<body>
	<div class="container">

    <h3>Sign Up For Fresno</h3>

	<form class="form-horizontal">

		{%raw%}{{template "FORM_6WIDE_TEXT" .first_name}}{%endraw%}
		{%raw%}{{template "FORM_6WIDE_TEXT" .last_name}}{%endraw%}
		{%raw%}{{template "FORM_6WIDE_TEXT" .email}}{%endraw%}
		{%raw%}{{template "FORM_6WIDE_TEXT" .password}}{%endraw%}
		{%raw%}{{template "FORM_6WIDE_TEXT" .confirm_password}}{%endraw%}
  	
  		<div class="form-group">
    		<div class="col-sm-offset-2 col-sm-6">
      			<button id="signup" class="disabled btn btn-primary">Sign up</button>
    		</div>
  		</div>
	</form>

	 {%raw%}{{template "MISC_FOOTER" .}}{%endraw%}
	
  </div> <!--container -->
	
	{%raw%}{{template "MISC_JSLOAD" .}}{%endraw%}
</body>
</html>
{% endhighlight %}

Since the code for each of the fields is the same, we have factored into
its own small template in `pages/template/form.tmpl`.  Then we re-used that
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
"FORM_6WIDE_TEXT" for each form element.

## Client side code

Assuming you have "gopherjs" in your path as per the above,
you can build the client side code (and the html pages) with

{% highlight bash %}
$ cd pages
$ make clean
[some files in static directory get removed]
$ make
pagegen --support=support --dir=template --start=index.html --json=index.json > ../static/en/web/index.html
pagegen --support=support --dir=template --start=signup.html --json=signup.json > ../static/en/web/signup.html
gopherjs build -o ../static/en/web/signup.js -m ../client/signup.go

{% endhighlight %}

When this make target runs, it _always_ generates the code for the client 
side code, using gopherjs to build javascript files such 
as `static/en/web/signup.js` (and the matching `static/en/web/signup.js.map`
file).

This is ok because it's quite fast as gopherjs does the same
type of rebuild-only-if-necessary that the standard go tool does.  The
make target is designed to give "fast feedback" when doing front-end
development.  Just _leave fresno running_ and then in another shell
switch to the `pages` directory and use "make" to "build everything 
again".  Then you can just refresh your browser and you'll see the changes.

The code for the client-side portion of this form is in `client/signup.go`.

### Entry point

A single html page, and its associated state, is represented as an
instance of Seven5's 
[Application](https://gowalker.org/github.com/seven5/seven5/client#Application) 
interface.  For now, there is only one method on Application, the
Start() method.  This method is critical because Seven5 will
do work to insure that this method will not be called before the 
DOM is fully ready. 

Thus in our go-level "main" for this page, we just call 
[s5.Main()](https://gowalker.org/github.com/seven5/seven5/client#Main)
with our Application object, of type "signupPage":
{% highlight go %}
func main() {
	s5.Main(newSignupPage())
}
{% endhighlight %}

In future lessons, you'll see how to exploit the fact that main()
is called _before_ the DOM is ready and thus might be a good place
to start doing things that don't require all the DOM elements to be
loaded yet.

<a name="constraints"></a>

### Attributes and constraints

Typically, you want to keep page state in your Application object
necessary for dealing with later events. 
Here is the definition of signupPage:

{% highlight go %}

type signupPage struct {
	first s5.StringAttribute
	last  s5.StringAttribute
	email s5.StringAttribute
	pwd1  s5.StringAttribute
	pwd2  s5.StringAttribute

	firstFeedback s5.StringAttribute
	emailFeedback s5.StringAttribute
	pwd1Feedback  s5.StringAttribute
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

So, how does this work in our example form?  First, all of the first
five attributes in the signup page have their values constrained to be
equal to the values in the type-in fields of the form.  The "magic" of
Seven5 is allowing constraints to be computed from parts of the DOM; all the other constraint processing algorithms/ideas have been 
around since the 1990s.  Thus, at any point in the code of `client/signup.go`
the value read from signupPage.first.Get() will be the value that is
currently in the corresponding text box on the screen.

We can go the other direction.  The last three fields of signupPage are 
used to compute the values of the feedback areas to the right of the 
text entry areas.  Again, Seven5 allows constraints to go "into" the 
DOM as well as come from it.  So, a call like 
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
feedback (emailFeedback()).

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
The articulation must be done so that the processing of inputs to
outputs can be encoded in constraint functions.  Our experience has
shown that the effort required to structure a UI with constraints is
easily outweighed by the benefits gained in cleaner UI code.

## GOPATH and jsmaps

Gopherjs generates 
[javascript source maps](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1)
that allows you to debug in the browser at the go level.

If you look at the developer console (see figure above) at the right of
the "hello, world" you'll see "signup.go:56" in italics.  You can click
on it and go to the line in the source code.  Seven5 knows how to 
interpret the request to show you the source code based on your GOPATH
(set in the enable script). 

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

Now that we have a form that takes in a user's information, we are going
to connect it to the server side so new users get created in the database.
This lesson is primarily about not trusting user input.

## Preparation for this lesson
In the `TUTROOT/src/tutorial` directory:

{% highlight bash %}
$ git checkout lesson-create-users
$ godep save tutorial/...
$ git checkout -b my-create-users
$ git add -A .
$ git commit -a -m "add godeps"
{% endhighlight %}

## Client side changes

We have slightly modified the client side code to print to the console
information about success or failure of creating users after you "push
the big blue button."  In Start() of the application code, in 
`client/signup.go`:

{% highlight go %}
func (s signupPage) Start() {
	//... elided ...
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
event and copying data out of the form (via the attributes!) into a 
shared.UserRecord structure.  This is then sent to the server with 
AjaxPost().  The details of the Ajax call will be covered in a future
lesson.


## Server changes

The server side changes are more interesting.  In main() (`freso/main.go`)
we now have informed the dispatcher that we have a Post implementation
(sometimes called "verb") in place as well as a Find:
{% highlight go %}

	base.ResourceSeparateUdid("userrecord",
		&shared.UserRecord{},
		nil, //index
		s5.QbsWrapFindUdid(&resource.UserRecordResource{}, store),
		s5.QbsWrapPost(&resource.UserRecordResource{}, store), //post
		nil, //put
		nil) //delete

{% endhighlight  %}

In `resource/user_record.go` we have implemented the method PostQbs
to handle the information sent from the client. The server needs
to take care to not simply trust the data in "proposed" (did you
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
	ur.Admin = false    //nuke the site from orbit
	ur.Disabled = false //its the only way to be sure
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
	proposed.UserUdid = s5.UDID() //generate a random UDID
	if _, err := q.Save(proposed); err != nil {
		return nil, s5.HTTPError(http.StatusInternalServerError, fmt.Sprintf("couldn't save: %v", err))
	}
	return proposed, nil
}

{% endhighlight  %}

## Testing creating users

You should be able to build the client side with "make" in `pages` and
the server with "godep go install tutorial/..." the main tutorial directory.
You can visit the new sign up page 
[here](http://localhost:5000/en/web/signup.html) if fresno is running.

You may want to try making the development console visible in your browser
and then cutting and pasting a newly created UDIDs at your server with
[curl](#curl-to-server).

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

We start with the "usual" three steps to build a fresno binary:

* Checkout the lesson's code
* Update godeps into this lesson
* Build and run fresno proper

The output of these shell commands has been elided for space:

{% highlight bash %}
$ git checkout lesson-auth-allow
$ godep save tutorial/...
$ godep go install tutorial/fresno
$ fresno
{% endhighlight %}

Then in another window try to retreive Mary's record from fresno, as 
[before](#curl-to-server):

{% highlight bash %}
$ curl localhost:5000/rest/userrecord/515f7619-8ea2-427f-8cf3-7a9201c747dd
Not authorized (FIND, UDID)
{% endhighlight %}

## Allow methods

What has changed in this lesson is that we have used _allow methods_ to
control access to this url.  Since your attempt to hit the server
with curl is not authenticated, much less authenticated as Mary, 
you cannot read this URL. 

Allow methods permit coarse-grain policies to be implemented "around"
a particular REST resource.  Allow methods run *before* any resource code
is executed and these should they indicate that access is or is 
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

In our new implementation of `resource/user_record.go` we now have 
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
is logged in, checks that the attempt to modify a record has the
same Udid as the logged in user.

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
interface{}) but typically what needs to be _hung_ on the session 
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
You can compile a program called key2hex to do this (assuming you are
at TUTROOT, the parent of the Godeps directory):

{% highlight bash %}

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

You should repeat the [previous lesson's setup](#setup-auth) but
substitute the branch name "lesson-auth-client".

## New URLs

Our new version of fresno has two authentication URLs that were describe
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

>In this and future lessons, we will frequently assume that you have
a fresno binary running.  We will put links in the tutorial like
the one in the next paragraph that point to localhost:5000.

Navigate to the 
[the fresno home page](http://localhost:5000/en/web/index.html)
and you will see this:

<img src="/assets/img/login1.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>

## Client-side checking of /me

The implementation of the client-side code for this page 
(`client/index.go`) is responsible for checking the value returned from
/me to see what to display on the upper right corner of the page.

There are three common idioms for client-side Seven5 code in the 
Start() method so we'll reproduce all of it here:

{% highlight go %}
//called after the dom is ready
func (self indexPage) Start() {
	chLoggedIn, chLoginErr := s5.AjaxGet(&shared.UserRecord{}, shared.URLGen.Me())
	postsURL := shared.URLGen.Posts(shared.MOST_RECENT, 10)
	var posts []*shared.Post
	chPosts, chPostErr := s5.AjaxIndex(&posts, postsURL)

	var haveLogin, havePosts bool
	go func() {
		for !haveLogin || !havePosts {
			select {
			case raw := <-chLoggedIn:
				user := raw.(*shared.UserRecord)
				haveLogin = true
				loginParent.Dom().Append(
					s5.SPAN(
						s5.Class(h5),
						s5.Text(user.EmailAddr),
					).Build(),
				)
			case <-chLoginErr:
				haveLogin = true
				loginParent.Dom().Append(
					s5.A(
						s5.HtmlAttrConstant(s5.HREF, "login.html"),
						s5.Text("login"),
					).Build(),
				)
			case raw := <-chPosts:
				havePosts = true
				posts := raw.(*[]*shared.Post)
				for i := 0; i < len(*posts); i++ {
					self.addPost((*posts)[i])
				}
			case perr := <-chPostErr:
				havePosts = true
				displayErrorText("Unable to retreive posts: "+perr.Message, true)
			}
		}
	}()
}
{% endhighlight %}

<a name="ajax-idiom"></a>

### The ajax idiom

We are sending the server a request for a particular url, the return value of 
shared.URLGen.Me() which works out to "/me" (see below for more on URLGen). 
We are _also_ sending a request for the last 10 posts.  These calls are
asynchronous, so any number may be launched before collecting the results.

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
of the wire type as the parameter.  For calls that do send values to the server, 
such as post or put, the _content_ of the first parameter is used to create the body
via json encoding. 

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
	Auth() string
	Me() string
	UserRecord(udid string) string
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
example Start() method above, the dynamic portion of the UI is the
value (or link) to be displayed in the upper right of the index page. 

The "to be determined" bit is encoded in go code as
the variable loginParent.  It is defined in the go code as:

{% highlight go %}
var (
	loginParent = s5.NewHtmlId("div", "login-parent")
)
{% endhighlight %}

The call to NewHtmlId searches the DOM for *exactly one* tag of type
div with Id of login-parent.  That tag is in the HTML code 
(`pages/template/index.html`):

{% highlight html %}

  <div class="row">
  	<div id="login-parent" class="col-sm-offset-10 col-sm-2">
   	</div>
  </div>
{% endhighlight %}

The above html code results in no display when the page is initally
loaded.  The content is filled in once the Ajax call in Start()
is completed.

It is important to note that the type HtmlId in Seven5 is aggresive
about checking for the existence of the identified portion of the DOM.
In the case of the go code above try changing the second argument
to "xxxlogin-parent" and then rebuilding:

{% highlight bash %}
$ cd pages
$ make
gopherjs build -m -o  ../static/en/web/index.js ../client/index.go
gopherjs build -m -o  ../static/en/web/signup.js ../client/signup.go
gopherjs build -m -o  ../static/en/web/login.js ../client/login.go

{% endhighlight %}

If you reload the http://localhost:5000/en/web/index.html page now, 
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

You should repeat the [lesson setup from two lessons ago](#setup-auth) but
substitute the branch name "lesson-auth-server".

## Performing a log in

With the background covered in the previous two lessions, 
let's log in to fresno.  Assuming fresno
is running, go to the login in page at 
http://localhost:5000/en/web/login.html:

<img src="/assets/img/login2.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>

>Don't forget to change your declaration of loginParent back and remove
the "xxx" if you made that change in the previous lesson!

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
authentication, we'll refer to some Seven5 structs like "SimpleFoo" and
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
`validating_sm.go`.

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

We will ignore the password reset code until a later lesson, but the
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
	ur.Admin = false //even if they are!
	ur.Disabled = false
	ur.Password = ""
	return s5.SendJson(w, ur)
}

{% endhighlight  %}


<a name="posts"></a>

# Display Blog Posts

## Setup for this lesson

You should checkout branch "lesson-display-posts" and do the 
[godep dance](#godep-dance) to get dependencies and build fresno.

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
of posts to return to the client side.  The index method takes some query parameters to allow the client side to do pagination if it desires. 
Finally, we do a slight bit of adjustment to the 
joined user record that represents the author of the post. 

The
code, from `resource/post.go`, is:
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
		author.EmailAddr = ""
		author.Password = ""
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

In the client-side code (`client/index.go`) for the index page, we
want to display the title on screen based on the model that 
represents what was retreived from the server. 

Here's the top part of Add(), one of the two methods (with Remove()) required
by the Joiner interface that we are implementing:

{% highlight go %}
func (self *indexPage) Add(i int, m s5.Model) {
	model := m.(*postModel)
	tree := s5.DIV(
		s5.Class(row),
		s5.DIV(
			s5.Class(blogEntry),
			s5.Class(colOffset1),
			s5.Class(col11),
			s5.DIV(
				s5.Class(row),
				s5.SPAN(
					s5.Class(col12),
					s5.Class(h3),
					s5.TextEqual(model.title),
				),
			),
			s5.DIV(
				s5.Class(row),
				s5.SPAN(
					s5.Class(col12),
					s5.Class(h5),
					s5.SPAN(
						s5.TextEqual(model.authorName),
					),
					s5.SPAN(
						s5.Text("@"),
					),
					s5.SPAN(
						s5.TextEqual(model.date),
					),
				),
			),
	//elided for space
}

{% endhighlight  %}

The critical lines are the first one where we extract the particular
model we are expecting, and the three calls to s5.TextEqual().  The
TextEqual() method takes an StringAttribute and binds its content
to the visible portion (textual part) of the tag it is nested inside.
Because this uses [constraints](#constraints), if you change the value
in the model, the screen will be updated appropriately. 

## Handling the index method for posts

Following our three steps above,  fresno now has been improved to 
load the blog posts over the network.  First it uses
the [ajax idiom](#ajax-idiom) in Start() (in `client/index.go`) to
request the posts.  At this point, it doesn't try to do pagination.
If the results are returned successfully, it receives a slice of pointers
to shared.Post() (`shared/post.go`).  It then calls addPost()
which takes the new shared.Post() object and converts it to a
postModel, and finally adds that to the collection posts.

It is worth noting that the posts collection has to now be initialized
when we create the page object.  This collection is "per-page state"
that exists during the entire time the page is visible in the browser.
We mention this per-page state because it is easy to forget about the
fact that there is code that must be holding state to make the UI respond
to various actions by the user, or data recevied via the network. 
Because of constraints, you typically can just build your Seven5 data
structures and then _let it run_.

After our work on the index page, fresno now looks like this:

<img src="/assets/img/posts1.png" hspace="30" vspace="30" 
alt="the signup page" 
style="border:1px solid black; width:80%;height:80%"/>


