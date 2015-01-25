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
"go install tutorial/fresno-dummy-main" will work.

### Godep workflow in Seven5

When doing development, the pattern for dealing with godeps is:

* go get -u blah
* godep save myprogram
* test myprogram to make sure everything is ok
* git add -A Godeps
* git commit

This "staging" area allows you to experiment with different version of various
packages and then "godep save" a snapshot.  Once you are satisfied with the
snapshot you commit that to the repository.

<a name="simple-server"></a>

# Creating And Deploying A Simple Server

For this lesson and all the ones following, we'll assume that you can
get the godeps set up correctly (it was explained in the previous lesson
if you are jumping around).  These are not checked into the 
tutorial source code and anytime you change lessons, you'll probably
have to "go get" some dependencies and "godep save" to copy them into
the lesson.  The go compiler will quickly tell you what dependencies
you need and don't have if you forget!

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

## Environment Variables

All configuration of Seven5 applications is done through environment
variables.  This makes them [twelve factorish](http://12factor.net) and
easy to deploy on heroku.  If you look at the enable script, there are
now some new variables `HEROKU_NAME` and `PORT` that are used to 
help initialize the app.  The variable `FRESNO_TEST` is *only* set in
the case of local development.

## Create heroku app

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
creates to sample users and sample posts.  This is handy for running tests
because you can just assume that if the database is there, these users
and posts are present.

### Caveat (Qbs data type matching)

When create a table in SQL, you have to create it with the structure
(data types and names) that will mesh with Qbs. Generally, this fairly
straightforward as the names are converted from camel case in go to
snake case for postgres.  However, you may need to experiment with
the column types to make sure that the structures in go match up them.
This is less burdensome than you'd expect because the set of types that
you can express in the go structures is limited.

The next lesson will discuss what structures in go correspond to the
tables created.

## Building and running the migrations locally

You can build and run the migration application like this:

{% highlight bash %}
$ godep go install tutorial/migrate
$ migrate --up
[migrator] attempting migration UP 001
001 UP migrations performed
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
of doing "migrate --down".   There options you can pass to run a specific
number of up or down migrations with the "--step" option.

## Building and running the migrations on heroku

If you push this version of the code to heroku, you can get a bash shell on
the remote (heroku) machine and then use the migrations just as with the 
local case.


