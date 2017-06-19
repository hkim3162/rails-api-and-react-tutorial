# Blueapron Rails/React Tutorial

This tutorial will be a step-by-step guide on building a Rails 5 API with
a single page React application.

### Who this is for?

Lifelong learning is a value at Blue Apron! Even though this tutorial will
probably be used mostly by new hires in the engineering department, it can really
be used as a learning guide for engineering-interested non-engineers. The next section
provides some beginner-to-intermediate materials for anyone interested in learning
the basics of programming in Ruby and Rails. The Hartl Tutorial is a detailed
step-by-step tutorial in building a full-fledged Rails micro-blogging application.
Once you've finished building that, I would then recommend following this tutorial
in building a Rails API with a single page React app.

### Some reading before we start:

**Ruby Basics**  
[How does I Ruby?](http://tutorials.jumpstartlab.com/projects/ruby_in_100_minutes.html)

[RubyMonk](https://rubymonk.com/) (I recommend the primer and the primer ascent sections.
  The metaprogramming section is interesting, but optional.)

**Rails Basics**   
Follow this tutorial for a few chapters, especially the intro chapter. It'll get you
setup with Rails, Ruby, and Git:

[Michael Hartl's Rails tutorial, chapters 1-6](https://www.railstutorial.org/book/beginning)

**On JSON:**
[Who is Jason?](https://www.copterlabs.com/json-what-it-is-how-it-works-how-to-use-it/)

**Some docs on Rails API:**
[Rail 5 API docs](http://edgeguides.rubyonrails.org/api_app.html)

**What is an API?:**
Technically, API stands for Application Programming Interface.
A lot of people think API is a server. An API isn’t the same as the remote server — 
rather it is the part of the server that receives requests and sends responses.

In short, an  API is a set of programmatically accessible commands used so that
programs can communicate with each other.

With the advent of client-side frameworks, more developers are using Rails to build
a backend that is shared between their web application and other native applications
like a client-side Javascript client application or a mobile application.

For example, Twitter uses its public API on its remote server for sending data to its
web application, which is built as a static site that consumes JSON resources.

### Setup Check

The Hartl tutorial above should have gotten you setup with Rails, Ruby, and Git.

Some prerequisites:
- Ensure that you have Ruby 2.2 or greater
- Install Rails

If you type these commands in terminal, you should see this:
```bash
ruby -v # ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-darwin16]
rails -v # Rails 5.0.1
```

If your ruby version is not up to date, you can update it with a ruby version manager like rbenv.

```bash
rbenv install 2.3.1
rbenv global 2.3.1
```

If your rails version is not up to date, update to the latest version by running:

```bash
gem update rails
```

Once you have the above working, fire away:

```bash
rails new rails_5_practice_api --api -T --database=postgresql
```

Note that we're using the `--api` argument to tell Rails that we want an API application
and the `-T` flag to exclude Minitest the default testing framework. Don't worry,
we will write tests... but using RSpec. Lastly, we're also going to use postgresql.

Let's get started with building out our API on the next branch!
