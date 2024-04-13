---
layout: post
title: Testing Sinatra session-dependent methods using Rack::Test
---
Have you ever estabilish a mock section for testing a Sinatra application? There is a great chance you will never do it. To be honest, I would think the same way and here I'm writing about it. So, you never know...

My project is built using Sinatra and, to be fair, it became bigger than I originally expected. I was using Rack::Test for testing then eventually I needed to test some functionalities that required a user to be logged in or logged out.

That was easy, I thought, I just need to create reproduce on the test environment what the application uses in production to assure the user is logged, let's use section!

Easier said than done, I came to realize that doing that on Sinatra with Rack::Test was not really intuitive, since, I could not access the application helper methods to accomplish that.

Below is a condensed version of the files I had in the project, only the necessary for understanding parts were kept, so we can focus on what it matters.

**app.rb**
  ```ruby
  require 'sinatra'
  require 'sinatra/activerecord'
  require_relative 'helpers/session_helpers'


  class App < Sinatra::Base
    helpers SessionHelpers

    enable :sessions

    get '/login' do
      erb :"users/login"
    end

    post '/login' do
      @user = User.find_by(email: params[:email])
      if !@user.nil? && @user.authenticate(params[:password])
        login(@user)
      else
        redirect "/login"
      end
    end

    get '/logout' do
      logout
      redirect "/login"
    end
  ```

**helpers/session_helpers.rb**
  ```ruby
  module SessionHelpers
    def login(user)
      session[:user_id] = user.id
    end

    def logout
      session.clear
    end
  end
  ```

**test/test_helper.rb**
  ```ruby
  ENV['APP_ENV'] = 'test'

  require_relative '../app'
  require 'test/unit'
  require 'rack/test'

  class AppTest < Test::Unit::TestCase
    include Rack::Test::Methods

    def app
      App
    end
  end
  ```

Using fixtures (Maybe a topic to cover next?!) I created the user FOO. Below is a basic exmaple of a test to check for the login/logout functionalities using that user credentials:
**test/views_test.rb**
  ```ruby
  require_relative 'test_helper'

  class ViewsTest < AppTest
    attr_reader :foo_user

    def setup
      @foo_user = User.find_by(name: "Foo")
    end

    def test_logged_and_unlogged_user_home_page
      login(foo_user)
      get "/"
      assert last_response.ok?
      assert last_response.body.include?(foo_user.name)
      logout
      get "/"
      assert last_response.ok?
      refute last_response.body.include?(foo_user.name)
    end
  end
  ```

Running this test will result on the errors:
- NoMethodError: undefined method `login'
- NoMethodError: undefined method `logout'

Passing the value directly to the session hash is also not an option, since the Rack::Test uses its own mock version of a session for testing and, even if not, the application session hash is not accessible through the test methods.
So the only way to accomplish that is creating new methods accessible on the test environment that can populate this mock session with the values we want.
In this case, I created this methods on the `test_helper.rb`. Below is the updated version of it:

**test/test_helper.rb**
  ```ruby
  ENV['APP_ENV'] = 'test'

  require_relative '../app'
  require 'test/unit'
  require 'rack/test'

  class AppTest < Test::Unit::TestCase
    include Rack::Test::Methods

    def app
      App
    end

    def login(user)
      env "rack.session", { :user_id => user.id} #Populates with values the mock session
    end

    def logout
      env "rack.session", { :user_id => nil} #Clears the values of the mock session
    end
  end
  ```
This solution allows the passing of the user information to the mock session with minimum modification of the original code application.
Of course, your application might require a customized version of this example for better addressing your needs, but I hope that can be a good starter.
Happy coding!

***References:***
- [*TESTING SINATRA WITH RACK::TEST*](https://sinatrarb.com/testing.html)
- [*Method: Rack::Test::Session#env*](https://www.rubydoc.info/github/brynary/rack-test/Rack%2FTest%2FSession:env)
