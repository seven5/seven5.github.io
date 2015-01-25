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

> After the "git checkout" command you get a long-winded warning about 
> being in "detached head state".  This is normal, can be ignored, and more
> permanent solution--creating a new branch for your work--will be explained
> in future lessons.

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

The use of "QbsWrapFind" or othe "QbsWrap*" methods  also implies the 
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
some tooling to help you generate web pages.  "Vhat?!? Generatung
veb kontent [_ist verboten_](/index.html#modern-web-pages)!?!"   As was
stated previously, this content is not generated by the server itself,
but rather is built completely before the server runs, so it does not
fall foul of the "no generating content" rule.

An easy way to insure that content does not break the prescription
against server-generated HTML is to ask if the server can (correctly) generate a [304](http://httpstatusdogs.com/304-not-modified) 
[Not Modified](http://httpstatus.es/304) response to a web request for
that html content.  The go default "serve file"
functions are careful to generate 304 for files that are just sitting in
a directory so, relax, everything is fine.

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
one cannot know all the  HTML is in advance because these are 
calculated at run-time and a different strategy is necessary.


## Preparation for this lesson
In the `TUTROOT/src/tutorial` directory:

## Preparation for this lesson

You'll need to get a copy of 
[gopherjs](http://github.com/gopherjs/gopherjs] installed locally for
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
it's own small template in `pages/template/form.tmpl`.  Then we re-used that
by changing the json:

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




