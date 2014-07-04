---
layout: post
title:  "A Critique of MeteorJS"
date:   2014-07-4 08:00:00
categories: web
---

[MeteorJS](https://www.meteor.com/) is an opinionated and evolving
framework for building web applications.  Meteor is an ambitious and
opinionated framework that spans both the server and the client with a
unique focus on enabling real-time applications where individuals
browser is reactive to changes.  That is, if I am a user and I am
viewing some data stored in the database which is updated, I should
see the updates to that data immediately, without having to reload the
page.

Meteor is still evolving and is rumored to be hitting 1.0 sometime
(though the development seems to have a very long tail).  As I know
several individuals considering using Meteor for real project today, I
will be critiquing it as it stands at version 0.8.2.

I have had the opportunity to work on several small and one large
meteor project over the course of the past year or so.

The Good
--------

Meteor is truly exciting due to the ease with which it makes
performing many traditionally difficult tasks in web development.
Just by watching the [meteor introductory
screencast](https://www.meteor.com/screencast) or reading its
marketing documentation, one cannot help but get excited about being
able to seamlessly react to changes to collections.

Meteor also has a few other strengths:
1. Javascript on both the server and the client and the ability to
   share some code between the two.
2. If you neeed live page updates, meteor has a very compelling model
   and performance has improved significantly with blaze.
3. Deployment to official infrastructure is very easy.
4. Package management with [meteorite]() works pretty well and there
   is a growing ecosystem of packages around the
   [meteor atmosphere](https://atmospherejs.com/)

The Not Good
------------

At the end of the day, most of my complaints with most technology is
that they do not help prograemmers solve the truly difficult problem
of managing complexity.  Once the honeymoon is over and the demos are
done, a proper solution must help me efficiently solve problems in a
way that I and others can maintain.

To that end, here are some of my concrete problems with meteor.

### Javascript

Javascript is apparently the new hotness.  Truth be told, I don't like
javascript.  The lack of a real module system amongst other things
make it really difficult to reason about your system, the scope of
variables, etc.  Throw in the fact that there is a sometimes blurry
distinction between server and client?  Well, it sucks.

Languages like [coffeescript](http://coffeescript.org/) hide some of
the dark corners of the language, encourage use of the good parts, and
add syntactic sugar.  They do not, however, really change the fact
that you are writing and interacting with javascript.  At the end of
the day, this is what you are debugging... Speaking of which...

### Debugging

Simply put, debugging (with an actual debugger or REPL) on the server
side is damn near impossible.  Debugging on the client works well
enough using Chrome's developer tools.  Things get hairy again on the
client when something goes wrong with your templates.

And with meteor, I have found myself in a position where debugging on
both the server and the client is something I spend far too much of my
time doing.

### MongoDB


The only supported data store right now is MongoDB.  Although
non-relational databases have advantages in some cases, I believe that
they generally push additional complexity onto the programmer (i.e. maintaing
[ACID](http://en.wikipedia.org/wiki/ACID).

Data is naturally relational and I believe that non-relational
databases often force you to denormalize your data much earlier than
is beneficial or necessary.  Having the database enforce constraints
like "this id is a foreign key to another relation" are no longer
available... You just need to not screw up.  The data that is in any
given collection is essentially ad-hoc.

And with meteor, you just need to suck it up and use what is likely
not the right technology for your project.

### Implicit Behavior

Meteor is an opinionated framework.  I have never been a big fan of
frameworks (especially large, opinionated ones) as they push a bunch
of behavior that happens implicitly on you as a programmer.  The
general name for this is [convention over
configuration](http://en.wikipedia.org/wiki/Convention_over_configuration).

I don't want to bash convention-over-configuration entirely as there
are times which it makes great sense, but with meteor I found myself
fighting the conventions.  Gotchas for me where I did not have the
control I felt I needed to manage my applications are documented here.

Philosophically, I tend to believe there is great wisdom in the [Zen
of Python](http://legacy.python.org/dev/peps/pep-0020/) which states:

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!

Right in the second line we see that "Explicit is better than implicit"

#### File Import and Load Order

Javascript did not get the memo about namespaces being "one honking
great idea" and as such, there is really no such thing built into the
language.  Since there are no namespaces, there are no imports.

So, what does meteor do?  Well, [this is what it
does](http://docs.meteor.com/#structuringyourapp).  It basically scours your
entire project directory in alphabetical order for any files ending in
`.js` and essentially smashes them together.  Needless to say, this
can lead to quite some pain.

Looking back on it, some sanity could have been restored by trying to
figure out the logical meteor packages for the application, but this
is easier said then done and is once again pushing the problem onto
the programmer.  More than once, I had to change the name of files
just to make something function correctly.  I find that ludicrous.

#### Under the Covers

Meteor hides, or tries to hide, a lot of the complexity required for
it to provide its reactive programming model.  The problem is that in
order to write applications with meteor, understanding how things work
under the covers quickly becomes something that you do in fact need to
know, especially related to the local database mirror,
[minimongo](https://www.npmjs.org/package/minimongo).

A few cases I ran into where my initial understanding of what was
going on failed me were the following:

* With [minimongo](https://www.npmjs.org/package/minimongo), updating
  a collection always happens immediately on the client with the
  change then being propogated to the server via websockets.  Problem
  is, that things get really complicated and you run into cases where
  a change will at first appear to be successful but then fail on the
  server (where you may be doing extra validation).  Dealing with this
  state correctly can be very difficult.  Do I make things look
  successful to the user only to take it back later?  That seems
  cruel.

* Things don't always act the same on the client and the server.  In
  many cases (for instance, when using
  [CollectionFS](https://github.com/CollectionFS/Meteor-CollectionFS))
  it turned out that using meteor APIs and libraries on the server is
  not well supported.  Although this isn't the primary use case,
  meteor's focus on making it possible to do more from the client in a
  transparent fashion has led to things not always working as expected
  on the server.

### Security

By default, meteor is not secure as [well
documented](http://docs.meteor.com/#dataandsecurity) in the meteor
docs.  The defaults make for great demos by hiding the real complexity
of carefully thinking about what data I should actually be sending to
the client and where I need to perform validation on the server side.

These are non-trivial tasks which are very hard to not think about in
a traditional web application.  In meteor, however, it is equally
important to consider carefully but (in my experience) much easier to
miss and more difficult to reason about.

