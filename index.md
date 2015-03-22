---
title: Seven5
layout: page
---

# Seven5: An Opinionated, Go-only Toolkit For Modern Web Applications

<img src="/assets/img/gargoyle-1.jpg" hspace="30" vspace="30" 
alt="the seven5 gargoyle" style="width:50%;height:50%; float:right;">

## Why
Look, let's just be honest and get it out now: 

> _You're here because you hate javascript._ 

You hate all this dynamic crap including, but not limited to, ruby, rails,
python, django, angular, and the webbish-dynamic-terdball-of-the-month. 
Node.js, and its foul brethren bower and npm, are both an affront to the honest
labor of programming and an abomination that deserves
brutal punishment. You like  programming in  static languages, and 
[go](http://golang.com)  is currently the best combination of a 
static language, simplicity, and big built-in library.

<a role="button" href="/tutorial.html" class="btn btn-primary">
I'm Convinced! Take Me To The Tutorial!</a>

## The Web
Seven5 is toolkit for building modern web applications.  The web, in
and of itself, is not terrible.  It's a great way to let lots of people
use your software with minimal deployment hassles.  What makes building
"the front end" so horrible is the tools foisted on you by 
the well-meaning but misinformed. Seven5 aims to make it 
reasonable to build the web portion of your app--it's never going to 
be "easy".  There is no easy way.

<a name="modern-web-applications"></a>

## Modern Web Applications
Modern web applications are highly interactive and leverage the browser's
capabilities.  Browsers are already deployed to billions of 
users, it seems silly, even wasteful, to not use them.
Modern web applications, under  no circumstances, _generate_ html content.
If you want a server-side templating language, these are rife--go elsewhere. 
Mustache and liquid may be small, but they are doing it wrong. 

It is one of the few  major flaws  of the  go standard library that it included html templating.  Seven5 expects you to generate all the 
content statically, at compile time, or build the  content on the 
client-side of the wire. No exceptions.

## Both sides
It uses go for both the *client and server*.  If you  want to use it 
just for the client side, and use a different server-side toolkit, or the standard library, that's fine too.  Server-side go frameworks are legion
and not super-important.  Seven5 exposes REST as its server-side framework
not because that is right or just, but because its good enough for many things
and lots of people know it.

## Why is it called Seven5
It's written as _Seven5_ and in code the package is imported as `s5`. 
It's called Seven5 because I started writing it when I lived in Paris, France.
All the zipcodes of Paris proper start with "75".  Names are not very 
important.  You can hire many name consultants who will tell you "go"
is a terrible name for as many reasons as you have money to pay them.
Seven5's mascot is one of the gargoyles from _Notre Dame_ because he looks
threatening. We hope to scare away some javascript people.

## The current release
The current [release](https://github.com/seven5/seven5/releases) of Seven5
is [cure](http://en.wikipedia.org/wiki/The_Cure), and the next release,
when it is ready, will be called [dcd](http://en.wikipedia.org/wiki/Dead_Can_Dance).


<a role="button" href="/tutorial.html" class="btn btn-primary">
Read The Tutorial</a>
