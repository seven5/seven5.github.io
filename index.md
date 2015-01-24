---
title: Seven5
layout: page
---

# Seven5

<img src="/assets/img/gargoyle-1.jpg" alt="the seven5 gargoyle" 
style="width:50%;height:50%">

## Why
Look, let's just be honest and get it out now: 

> _You're here because you hate javascript._  

You hate all this dynamic crap including, but not limited to, ruby, rails,
python, django, angular, and the webbish-dynamic-terdball-of-the-month. 
Node.js, and its foul brethren bower and npm, are both an abomination 
and an offense to the spirit of getting useful work done with a 
computer.  You like  programming in  static languages, and 
[go](http://golang.com)  is currently the best combination of a 
static language, simplicity, and big built-in library.

## The Web
Seven5 is web toolkit for building modern web applications.  The web, in
and of itself, is not terrible.  It's a great way to let lots of people
use your software with minimal deployment hassles.  What makes building
"the front end" so horrible is the awful tools.  Seven5 aims to make it 
reasonable to build the web portion of your app--it's never going to 
be "easy", despite the claims of many of the above.

## Modern Web Applications
Modern web applications are highly interactive and leverage the browser's
capabilities.  Browsers are already deployed to billions of 
users, it seems silly, even wasteful, to not use them.  
Modern web applications, under  no circumstances "generate" html content.  
If you want server-side templating language, these are rife--go elsewhere.   
Mustache may be small, but it's doing it wrong. 

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
important.  I am sure you can hire many consultants who will tell you "go"
is a terrible name for as many reasons as you have money to pay them.


