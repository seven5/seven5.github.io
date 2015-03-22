---
title: Heroku detour
layout: detour
---

# Heroku

## Setup

For this detour, you'll need to insure that you have the 
[heroku toolbelt](https://toolbelt.heroku.com) installed on your system. 
You should have an account with heroku, try using "heroku login".  You 
may need to go to their website to create an account if you don't have
one already.

As with heroku, all Seven5 configuration is through environment variables.
If you look at the enable script, there are some variables 
`HEROKU_NAME` and `PORT` that are used to  help initialize the app.
The variable `FRESNO_TEST` is *only* set in the case of local development.

## Create heroku app

> It would be great if somebody could submit a pull request that
> has deployment support for [dokku](http://progrium.viewdocs.io/dokku)
> running locally, [tutum](https://www.tutum.co), 
[gce](https://cloud.google.com/container-engine/), 
[acs](https://cloud.google.com/container-engine/) or similar.

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

## Build and run on heroku

You first need to tell heroku about the buildpack for Seven5 and the app's
own name (change this to the name you got when you created your app). 
After that "git push" is your deployment mechanism:

{% highlight bash %}
$ heroku config:set BUILDPACK_URL=https://github.com/seven5/heroku-buildpack-seven5.git
$ heroku config:set HEROKU_NAME=damp-sierra-7161
$ heroku config:set STATIC_DIR=static
$ heroku config:set SERVER_SESSION_KEY=xxxxx #look in enable script for how to generate this value
$ git push heroku tutorial:master
[bunch of output about the build procedure]
remote:        https://damp-sierra-7161.herokuapp.com/ deployed to Heroku

{% endhighlight %}

You'll notice that the branch "tutorial" is being pushed to "master"
on heroku because heroku only builds on pushes to master. 

If you see an error like this:
<pre>
! [rejected]        my-simple-server -> master (non-fast-forward)
error: failed to push some refs to 'https://git.heroku.com/damp-sierra-7161.git'
</pre>
it is usually because you have re-written your git history and you need
to use force (-f) with the git push to get heroku to accept the push.

You should now be able to visit "https://damp-sierra-7161.herokuapp.com/" 
(with your app's name) and see the same output that you receive locally.

## Add postgres your heroku app

> It would be great if somebody could try to get this application to work
> under mysql since both qbs and heroku support that relational system.

You can add the production database to your heroku app like this:

{% highlight bash %}
$ heroku addons:add heroku-postgresql
$ heroku config
[configuration output]
{% endhighlight %}

You'll notice in the output of the "heroku:config" now includes a
`DATABASE_URL`.  There is a corresponding entry in the enable
script for the local development case.

{% highlight bash %}
$ cat $TUTROOT/src/tutorial/enable-tutorial
[ lots of settings]
export DATABASE_URL="postgres://$USER@localhost:$PGPORT/fresno"
{% endhighlight %}

### No need to create a database

You will need to be aware that heroku creates the database user, database password, and 
database name for you and these are all "hard to guess" random values.
Because all the configuration information is drawn from the 
`DATABASE_URL` you can run the same go code on both your local system 
and on heroku (and they may very well be different operating systems,
if you are developing locally on OSX).

## Getting a psql prompt

You can access psql to get an SQL prompt from the heroku toolbelt:

{% highlight bash %}
$ heroku pg:psql
{% endhighlight %}

> By-hand updates to the "production" database on heroku may
> be an exceptionally bad idea.


## Building and running migrations on Heroku

If you push this version of the code to heroku, you can get a bash shell on
the remote (heroku) machine and then use the migrations just as with the 
local case.

{% highlight bash %}
$ git push -f heroku tutorial:master
$ heroku run bash
Running `bash` attached to terminal... up, run.4328
$ migrate status
current migration number is 000
$ migrate --up
[migrator] attempting migration UP 001
001 UP migrations performed

{% endhighlight %}

## Building tags

To avoid conflicts with the "standard" go compiler you should put this
at the top of all the client-side files you create (and this tutorial
does this):

{% highlight go %}
// +build js

{% endhighlight %}

Note that this *two* lines including the blank line that must be present before
the file's package declaration.

[Return To Main Tutorial](tutorial.html#simple-server)
