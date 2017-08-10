# Ruby on Light Rails

The following tutorial is intended to introduce you to many of the topics we'll be covering in Module 2. It's not expected that you fully understand each and every topic, and entirely expected that this might be your first introduction to all of them. Try to soak it in. We'll be talking more about these topics throughout the module.

Additionally, in the coming weeks we'll be introducing you to tools to do some of the work we're asking of you in this tutorial. It is far more important at this point that you try to understand the big picture ideas of how the web works than the details of how we are implementing a server (which will change). Something like the HTTP request/response cycle with status codes, headers, verbs, and bodies will be used in any web app you build in Sinatra, Rails, Node, Django, or Pheonix, but we will be moving you into different app and testing frameworks even in the next few days. Use this as an opportunity to peak into what it is that a browser does when it sends a request, and what all information an app needs to prepare in its response.

As always, if you have any questions or notice any issues, please feel free to reach out.

## Background

Ruby on Rails and Sinatra are both frameworks that use the Ruby language to serve applications on the web. One of the things these two frameworks have in common is that they use [Rack](http://rack.github.io/) to interact with the web. While these are likely the two most popular Ruby web frameworks that use Rack, there are actually many more (e.g. [Padrino](http://padrinorb.com/), [Cuba](http://cuba.is/), [Hanami](http://hanamirb.org/), [Hobbit](https://github.com/patriciomacadden/hobbit), [Utopia](https://github.com/ioquatix/utopia), [Ramaze](http://ramaze.net/), [Camping](https://github.com/camping/camping), etc.).

A little bit of knowledge about the web and Ruby will get you surprisingly far in creating a Rack-based app. In order to test this theory out, let's go ahead and see if we can put something together, see it locally, and deploy it to the web.

## But First... Testing!

At this point, you've had an opportunity to test applications that you've built in the terminal without any concern for how someone would interact with them using a mouse. Today that changes. So, what do we want to test on a webpage? We want to make sure that when we visit a page we see the content we expect, when we click on a link it takes us where we expect, when we fill out a form and hit submit it creates a new record, and when we delete something it actually deletes from our database. Where do we get all of this? Capybara.

We're going to do a little bit of testing setup before we start coding out our server. It's going to take us awhile to even have a failing test, let alone a passing one. Let's start by laying out some directories.

### Setting Up

In your Terminal, move to the directory where you'd like to store your application and enter the following:

```
$ mkdir personal_site
$ cd personal_site
$ touch Gemfile
$ touch Rakefile
$ mkdir app
$ mkdir app/controllers
$ touch app/controllers/personal_site.rb
$ mkdir test
$ touch test/test_helper.rb
```

### Gemfile

Add the following to your Gemfile

```ruby
source "https://rubygems.org"

gem "minitest"
gem "pry"
gem "rake"
gem "capybara"
gem "launchy"
gem "rack"
```

A few of these will likely be new to you. You'll have plenty of time to familiarize yourself, but at a high level.

* Capybara allows us to do test how our apps look in a web browser.
* Launchy allows us to stop in the middle of a test to see what is currently showing on our web page (using the command `save_and_open_page`).
* Rack is the gem we're going to use to allow us to receive HTTP requests and send HTTP responses.

Be sure to run `$ bundle` from the command line!

### Rakefile

With that, let's set up our Rakefile to allow us to run our tests by just typing `rake` at the command line.

```ruby
require "rake"
require "rake/testtask"

Rake::TestTask.new do |t|
  t.libs << "test"
  t.test_files = FileList['test/**/*_test.rb']
end

task default: :test
```

Note that we are using a `**` when we tell Rake where to look for our test files. This will allow us to nest test files into subdirectories under `test`. Rake will run them as long as they end with `_test.rb`.

### Test Helper

Add the following to your test helper.

```ruby
require 'minitest/autorun'
require 'minitest/pride'
require 'capybara/minitest'
require './app/controllers/personal_site'

Capybara.app = PersonalSite

class CapybaraTestCase < Minitest::Test
  include Capybara::DSL
  include Capybara::Minitest::Assertions
end
```

In this application, the Test Helper will be the way that our tests are made aware of our application. We do this by requiring `./app/controllers/personal_site`, and then setting a class called `PersonalSite` (which we haven't created yet) as the app that Capybara should look for when it's running our tests.

Note that I copied most of these lines almost directly from the [Capybara documentation](https://github.com/teamcapybara/capybara). In it, there is a [Setup](https://github.com/teamcapybara/capybara#setup) section that says the following:

> If the application that you are testing is a Rack app, but not Rails, set Capybara.app to your Rack app:

```
Capybara.app = MyRackApp
```

And later, there is a section called [Using Capybara with Minitest](https://github.com/teamcapybara/capybara#using-capybara-with-minitest) which includes the following passage:


> If you are not using Rails, define a base class for your Capybara tests like so:

```
require 'capybara/minitest'

class CapybaraTestCase < Minitest::Test
  include Capybara::DSL
  include Capybara::Minitest::Assertions

  def teardown
    Capybara.reset_sessions!
    Capybara.use_default_driver
  end
end

```

> Remember to call super in any subclasses that override teardown.

I deleted the `#teardown` method in part because 1) I didn't think we would be using sessions or changing drivers in our simple app (I don't expect you to know about/worry about these things at this point), and 2) I wanted to see if I could take it out and everything would still work.

It did.

## Can We Write a Test Already?

I think so. Let's try.

Let's make one more directory so that we can organize our test suite a little bit. We're totally going to write some feature tests.

```
$ mkdir test/features
$ touch test/features/user_sees_a_homepage_test.rb
```

Open that new test file in your favorite text editor and add the following.

```ruby
require './test/test_helper'

class HomepageTest < CapybaraTestCase
  def test_user_can_see_the_homepage
    visit '/'

    assert page.has_content?("Welcome!")
    assert_equal 200, page.status_code
  end
end
```

What's all this about now? What does each line do?

1) First we require our test helper so that we get access to minitest and all the Capybara goodness we added to it.
1) Then we create a new class for our test (consistent with Minitest tests that we've written in the past), except now we inherit from CapybaraTestCase because we want to have access to all of those Capybara methods in our test (and because Capybara told us that's how we do it).
1) Then we use one of the new methods that Capybara gives us `#visit` to go to the page at the root of our application (which... I know... still doesn't exist).
1) Then we make some assertions: the first an `#assert` that will check to see that the argument evaluates to true, and the second an `#assert_equal` that will check the equality of `200` and the `page.status_code`

`page` here is also new, but basically that is Capybara's way of holding the response that we get back from our server. We'll talk more about it shortly.

If we run this test, we should get an error saying something about an `uninitialized constant PersonalSite`. Even though we created the file and told this test about that file, we haven't actually created the class. Let's do that now.

## Making Our Test Pass

Open the `app/controllers/personal_site.rb` page and add the following:

```ruby
class PersonalSite

end
```

Run our test again, and now we get an error saying something similar to the following:

```
   1) Error:
HomepageTest#test_user_can_see_the_homepage:
TypeError: The second parameter to Session::new should be a rack app if passed.
    /Users/sespinos/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/capybara-2.15.1/lib/capybara/session.rb:79:in `initialize'
    /Users/sespinos/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/capybara-2.15.1/lib/capybara.rb:304:in `new'
    /Users/sespinos/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/capybara-2.15.1/lib/capybara.rb:304:in `current_session'
    /Users/sespinos/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/capybara-2.15.1/lib/capybara/dsl.rb:45:in `page'
    /Users/sespinos/.rbenv/versions/2.4.1/lib/ruby/gems/2.4.0/gems/capybara-2.15.1/lib/capybara/dsl.rb:50:in `block (2 levels) in <module:DSL>'
    /Users/sespinos/Desktop/personal_site/test/features/user_sees_a_homepage_test.rb:5:in `test_user_can_see_the_homepage'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
rake aborted!
Command failed with status (1)

Tasks: TOP => default => test
(See full trace by running task with --trace)
```

The main piece that I pick out of this is that something `should be a rack app if passed`. We've included `gem rack` in our Gemfile, but haven't really done anything with it at this point. Let's dig more into that now.

## Introducing Rack

### Overview

Rack allows us to write web applications that receive HTTP requests from clients (e.g. web browsers), and send HTTP responses. There are just a few requirements for our application to play nicely with Rack.

1) If we are going to create a class to hold our app, it must have a method `call` that takes an argument that we'll call `env`.
1) The `call` method must return an array with three components:
    1) An HTTP status code (a three digit number that tells a client if their request was successful, and if not provides some idea of why).
    1) HTTP headers (generally providing some information about the response).
    1) A message body (usally HTML, CSS, or JavaScript).

We will also need to require `rack` in our application file.

For more information on HTTP status codes, see [this](http://www.restapitutorial.com/httpstatuscodes.html) page. If you've ever seen a page that says something like `404 Page not found`, you've seen a status code! 404 is a server's way of telling you that you did something wrong. Other popular codes include `200` (everything is great! Here's your stuff!), and `503` (Server screwed something up. Not your fault.)

It's up to us if we want to make this `call` method an instance or a class method. We're going to use a class method because that's what Rails and Sinatra do.

### Updating PersonalSite

Update your `personal_site` file as follows:

```ruby
require 'rack'

class PersonalSite
  def self.call(env)
    ['200', {'Content-Type' => 'text/html'}, ['Welcome!']]
  end
end
```

That should do it. Let's run our tests again, and... Passing! Great.

Let's do a little bit of exploration.

### What's Going on Here?

First, put a pry into the `::call` method in our PersonalSite class:

```ruby
require 'rack'

class PersonalSite
  def self.call(env)
    require 'pry'; binding.pry
    ['200', {'Content-Type' => 'text/html'}, ['Welcome!']]
  end
end
```

And run your test.

Check to see what `env` is when you hit the pry. Spoiler: it's a hash!

It looks like there are quite a few key/value pairs in that `env` hash. Some of the interesting ones include:

* `"PATH_INFO"`: storing the path that the client is trying to visit. For now it should be `/`, which matches what we put into our test.
* `"REQUEST_METHOD"`: storing the HTTP verb that is used in the request. Currently this should be a GET. We'll learn others early in the module.
* `"QUERY_STRING"`: tells us if a user sent us any additional information in the URL. Can be used to pass data from the client to our application.

That's all interesting enough. Now let's pry into the test to see what we get there.

Remove the pry from our PersonalSite class and add one to our test as follows:

```ruby
require './test/test_helper'

class HomepageTest < CapybaraTestCase
  def test_user_can_see_the_homepage
    visit '/'

    require 'pry'; binding.pry
    assert page.has_content?("Welcome!")
    assert_equal 200, page.status_code
  end
end
```

Run your test again, and check to see at this point in the test what `page` is holding. You should be able to run the following commands:

```
> page
# => #<Capybara::Session>
> page.body
# => "Welcome!"
> page.status_code
# => 200
> page.response_headers
# => {"Content-Type" => "text/html", "Content-Length" => "8"}
```

So, in our app, we tried to send a response with a body, status code and headers, and it looks like that's exactly what got sent back. It seems like Rack does some work for us to calculate the number of characters in our response ("Welcome!" has 8), but other than that, this is pretty much what we told our app to do. Nice!

When we assert that the `page.has_content?("Welcome!")`, we are checking that `Welcome!` is in the body of our response. When we assert that the status code is 200, we're checking to see that's what our server sent back.

If you want a full list of methods that you can call on page, you can run `page.methods`.

Go ahead and remove the `pry` from our test suite and run it one more time to make sure we haven't broken anything.

## config.ru

So we kind of have a site running. We definitely have something that passes our test, but we haven't actually seen it in a browser yet. We came here to make web pages! Let's make some web pages!

Well... not so fast? We have to do a little bit of setup before we can see this in our browser.

Create a `config.ru` file by running `touch config.ru`

What does this file do? It tells Rack about our application. If our `test_helper` is the entry point for our application when our test suite is trying to access it, `config.ru` is the entry point when we're trying to access our app from the web.

Add the following to your newly created `config.ru` file.

```ruby
require './app/controllers/personal_site'

run PersonalSite
```

Save that and run `$rackup` from your terminal. You should see your browser spin up and print something similar to this:

```
Puma starting in single mode...
* Version 3.9.1 (ruby 2.4.1-p111), codename: Private Caller
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:9292
Use Ctrl-C to stop
```

You might have a different server (mine is Puma), and that might impact exactly what gets printed, but you should have something specifying a port number tacked onto `localhost`. Copy that (in my case `localhost:9292`), open up your browser, and pop it into the navbar. Once you hit return, you should see a page that says `Welcome!`

Website!

Go back to the terminal window where your server is running and check out the new information that you see. Mine shows this:

```
::1 - - [09/Aug/2017:23:34:15 -0600] "GET / HTTP/1.1" 200 - 0.0012
::1 - - [09/Aug/2017:23:34:15 -0600] "GET /favicon.ico HTTP/1.1" 200 - 0.0010
```

It looks like two requests were made, both of them GET requests. The first for `/`, and the second for `/favicon.ico`, both of which returned 200 responses. What's that favicon request? Something that browsers send to get those little icons that show up on some sites when you open tabs. We haven't set one explicitly, but it seems like maybe Rack is doing us some more favors in the background.

## Next Steps

This is great! We have a super simple website that we're serving locally and seeing in our browser. That in and of itself is super cool. What's next?

1) Our response is currently hard coded into our app. Let's see if we can use what we know about File IO to create some HTML pages elsewhere serve them up as a response.
1) Right now, `Welcome!` is the response to every request made to our server. Go ahead and try it. Visit `localhost:9292/blog` and see what happens. Still `Welcome!`. Let's figure out how to display different pages to our users based on the specific URI that they visit. It would also be nice to be able to link between the two of them.
1) Additionally, we'd like to be able to add some styling to our page. Let's add links to CSS in the pages we've created and be sure that we can serve that up to our users.
1) Can we deploy our site to the web so other people can see it? You bet we can.


