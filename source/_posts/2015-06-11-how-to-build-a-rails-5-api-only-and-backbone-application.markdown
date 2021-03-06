---
layout: post
title: How to build a Rails 5 API only and Backbone application
comments: true
author:
  name: Jorge Bejar
  email: jorge@wyeworks.com
  twitter_handle: jmbejar
  github_handle:  jmbejar
  image:  /images/team/jorge-bejar.jpg
  description: Software Engineer at Wyeworks. Ruby on Rails developer.
published: true
---

A few weeks ago, [an announcement was made](http://wyeworks.com/blog/2015/4/20/rails-api-is-going-to-be-included-in-rails-5/) referring to the imminent inclusion of Rails API into Rails core. Until now, Rails API has been a separated project and people have been using it through the `rails-api` [gem](https://github.com/rails-api/rails-api).

Santiago Pastorino and I have been working on bringing Rails API into Rails for a while. After some further discussion, bug fixes and last-minute changes, the [corresponding pull request](https://github.com/rails/rails/pull/19832) was finally merged. We're happy to confirm that all Rails API capabilities will be available once Rails 5 is released!

Rails API goal is to facilitate the implementation of API only Rails projects, where only a subset of Rails features are available. A Rails API application counts with lightweight controllers, a reduced middleware stack and customized generators. All these features were conceived looking for a better experience at the moment of building an API only application.

For more detailed information about the Rails API project, you can take a look at [this Santiago Pastorino's article](http://wyeworks.com/blog/2012/4/20/rails-for-api-applications-rails-api-released/) about the project.

<!-- more -->

## How to implement a Rails API backend for a simple Backbone application.

Let's create an API only application! The rest of this article is a step-by-step example of how to build a Rails API backend for a simple TODO list application implemented with Backbone.

Since we want to focus on the backend implementation and its integration with the client side application, we decided to borrow the [Backbone TODO application from the TodoMVC project](https://github.com/tastejs/todomvc/tree/gh-pages/examples/backbone).

### Generating the Rails API only application

Once Rails 5 is released, creating an API only application will be accomplished by running:

<pre>rails new &lt;application-name&gt; --api</pre>

However, this feature was just incorporated into the master branch at the time of writing, so we need to generate the application directly from the most recent version of the Rails source code. The easiest way to have a copy of this code is by cloning the Rails Github project in our computer:

<pre>git clone git://github.com/rails/rails.git</pre>

Now we must run the `rails new` command in the folder where the repo was cloned. In order to have our generated project pointing to our local copy of the Rails source code, we need to run this command in the following manner:

<pre>bundle exec railties/exe/rails new &lt;parent-folder-path&gt;/my_api_app --api --edge</pre>

It's a good idea to specify a path for the generated project, so we avoid creating the Rails API application inside the Rails source code folder. That explains the `<parent-folder-path>` placeholder in the example below.

Now, we can explore what was generated in the new project's folder. You will notice that almost everything looks exactly the same than a regular Rails application, and that's certainly true. However, let's highlight what is different.

There are some changes in the generated Gemfile:

```ruby
source 'https://rubygems.org'

gem 'rails', github: "rails/rails"
gem 'sprockets-rails', github: "rails/sprockets-rails"
gem 'arel', github: "rails/arel"

# Use sqlite3 as the database for Active Record
gem 'sqlite3'
# Use ActiveModel has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Unicorn as the app server
# gem 'unicorn'

# Use Capistrano for deployment
# gem 'capistrano-rails', group: :development

# Use ActiveModelSerializers to serialize JSON responses
gem 'active_model_serializers', '~> 0.10.0.rc1'

# Use Rack CORS for handling Cross-Origin Resource Sharing (CORS), making cross-origin AJAX possible
# gem 'rack-cors'

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug'
end

group :development, :test do
  ....
end
```

We can notice that stuff related with asset management and template rendering is not longer present (`jquery-rails` and `turbolinks` among others). In addition, the `active_model_serializers` is included by default because it will be responsible for serializing the JSON responses returned by our API application.

Let's now check out the config/application.rb file:

```ruby

....
....

module TodoRailsApiBackend
  class Application < Rails::Application

    ....
    ....

    # Only loads a smaller set of middleware suitable for API only apps.
    # Middleware like session, flash, cookies can be added back manually.
    # Skip views, helpers and assets when generating a new resource.
    config.api_only = true
  end
end
```

The `api_only` config option makes posible to have our Rails application working excluding those middlewares and controller modules that are not needed in an API only application.

Last but not least, our main ApplicationController is defined slightly different:

```ruby
class ApplicationController < ActionController::API
end
```

Please note that `ApplicationController` inherits from `ActionController::API`. Remember that Rails standard applications have their controllers inheriting from `ActionController::Base` instead.

In case you're interested on turning an existent Rails application into an API only application, the differences mentioned below are the list of changes that you need to do manually in order to achieve that.

### Scaffolding the Todo resource

The main purpose of our API application is to serve as a backend storage for our list of TODOs. For this reason, we need to generate a new resource in our Rails API project. After looking at the code of the Backbone TODO application, we know we will need to define a Todo model including some attributes: a string `title`, a boolean `completed` and an integer `order`.

Let's use the scaffold command:

<pre>bin/rails g scaffold todo title completed:boolean order:integer</pre>

Again, we need to make sure all rails commands run the latest Rails source code in our computer, so we must run the executables from the bin folder (otherwise, the scaffold would use the rails code from an installed rails gem in the system).

This command is the same explained in the Rails' guides and books about the framework. Rails API does not require any change or additional options in all subsequent commands. The `rails-api` option added to the `config/application.rb` is enough to alter how scaffolding and other things works in our API only project.

The generated TodoController looks like this:

```ruby
class TodosController < ApplicationController
  before_action :set_todo, only: [:show, :update, :destroy]

  # GET /todos
  def index
    @todos = Todo.all

    render json: @todos
  end

  # GET /todos/1
  def show
    render json: @todo
  end

  # POST /todos
  def create
    @todo = Todo.new(todo_params)

    if @todo.save
      render json: @todo, status: :created, location: @todo
    else
      render json: @todo.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /todos/1
  def update
    if @todo.update(todo_params)
      render json: @todo
    else
      render json: @todo.errors, status: :unprocessable_entity
    end
  end

  # DELETE /todos/1
  def destroy
    @todo.destroy
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_todo
      @todo = Todo.find(params[:id])
    end

    # Only allow a trusted parameter "white list" through.
    def todo_params
      params.require(:todo).permit(:title, :completed, :order)
    end
end
```

If we compare this controller with one generated for a regular Rails application, we would find some differences. First of all, the actions `new` and `edit` are not included. That's because these actions are used to render the html pages containing the forms where users fill the data to be added or modified in the system. Of course, it does not make sense in an API because this application is not longer responsible for rendering html pages.

Looking at the render statements within the actions we can see more differences. The controller only responds in JSON format. The usage of `respond_to`, which allows us to handle different format responses or differentiate ajax requests, is not longer available.

The  controller tests, generated by the scaffold in the file `test/controllers/todos_controller_test.rb`, are consistent with the actions and format of the responses expressed in the `TodoController`.

No template files are generated when scaffolding a new resource in our Rails API application. We don't have the `app/view` folder in our project.

The `config/routes.rb` file includes now the following line:

```ruby
resources :todos
```

This is exactly the same result after running the scaffold command in a regular Rails application. However, this line defines only the routes that are necessary in our API. In other words, `new` and `edit` routes are excluded now. We can confirm that by running `bin/rake routes`:

<pre>
Prefix Verb   URI Pattern          Controller#Action
 todos GET    /todos(.:format)     todos#index
       POST   /todos(.:format)     todos#create
  todo GET    /todos/:id(.:format) todos#show
       PATCH  /todos/:id(.:format) todos#update
       PUT    /todos/:id(.:format) todos#update
       DELETE /todos/:id(.:format) todos#destroy
</pre>

Do not forget to run `bin/rake db:migrate`, so the database is ready when the time comes to test our application.

### Defining how TODO items should be serialized

Our API only application will respond incoming requests in JSON format, and here is where Active Model Serializers plays an important role. It requires to define the `TodoSerializer` class and provide the list of attributes from our `Todo` model to include in the responses.

Active Model Serializers offers a command to generate this serializer:

<pre>bin/rails g serializer todo title completed order</pre>

The result is the file `app/serializers/todo_serializer.rb`:

```ruby
class TodoSerializer < ActiveModel::Serializer
  attributes :id, :title, :completed, :order
end
```

At this point, we have implemented with almost zero effort a working backend application. We can run `bin/rails s` to start the web server and test our API with the help of `curl` command.

Let's create a new Todo

<pre>curl -H "Content-Type:application/json; charset=utf-8" -d '{"title":"something to do","order":1,"completed":false}' http://localhost:3000/todos</pre>

and it should return:

<pre>{"id":1,"title":"something to do","completed":false,"order":1}</pre>

Now, we can try to get the list of TODOs:

<pre>curl http://localhost:3000/todos</pre>

and the result should be:

<pre>[{"id":1,"title":"something todo","completed":false,"order":1}]</pre>

### Putting both components to work together

It's time to integrate the Backbone application with our backend implementation!

The original implementation of this TODO list application uses the browser local storage. However, we want to indicate that our Rails API application is the new storage for the data.

Let's replace the following lines in `js/collections/todos.js`:

```js
  // Save all of the todo items under the `"todos"` namespace.
  localStorage: new Backbone.LocalStorage('todos-backbone'),
```

with a definition for the URL of our API endpoint for TODO items:

```js
  // Set the rails-api backend endpoint for this specific model
  url: 'http://localhost:3000/todos',
```

After this change, we are almost ready. Our last outstanding task is to configure the cross origin policy to make the communication between both components possible. This is necessary when the backend and the client components are in different domains. Since we are doing local testing, we will run both things in localhost but using different ports, so we will end up having CORS errors in the browser if we don't configure any cross origin policy.

In Rails API, the handling of CORS is not enabled by default, however you can find the `rack-cors` gem listed in the Gemfile, but commented out.

In order to fix the CORS problem, we only need to uncomment this line and run `bundle install` again in the Rails API project. Also, we need to take a look at the file `config/initializers/cors.rb`. This file has an example of how CORS can be configured in our project.

Let's do some changes in this file, so we can test both components, assuming that we will use port 9000 to run the client side application:

```ruby
# Avoid CORS issues when API is called from the frontend app
# Handle Cross-Origin Resource Sharing (CORS) in order to accept cross-origin AJAX requests

# Read more: https://github.com/cyu/rack-cors

 Rails.application.config.middleware.insert_before 0, "Rack::Cors" do
   allow do
     origins 'localhost:9000'

     resource '*',
       headers: :any,
       methods: [:get, :post, :put, :patch, :delete, :options, :head]
   end
 end
```

You can read more about the `rack-cors` gem and how to configure the policies [here](https://github.com/cyu/rack-cors).

Once we have configured the cross origin policy in our backend, we are ready to test our client side application. We should turn on the backend server (`bin/rails s`) but also run a separated web server for the Backbone application.

We can simply run a test server using Ruby to test locally the frontend. Run the following command in the Backbone application's folder:

<pre>ruby -run -e httpd . -p 9000</pre>

Give it a try by browsing to [localhost:9000](http://localhost:9000). You will notice that TODO items are being stored by our Rails API backend now instead of the browser local storage. Therefore, both components are connected and communicated with each other.

## Conclusion

The aim of this article was to show how easy is to implement an API only backend for a simple Backbone application using the new Rails API feature. Rails API just landed into Rails source code, however you can count with this feature because the next Rails major release is around the corner.

Some stuff can still be improved. For instance, the file containing the `TodoSerializer` class should be generated as part of the scaffold command when the `active_model_serializers` gem is included in the project. We have fixed this specific problem and [it is already merged on master](https://github.com/rails-api/active_model_serializers/commit/4752e6723a6e0e8c4038ed4f36b87a954ad21097) and ready for the next release of `active_model_serializers`.

We also have other options in terms of serialization. In fact, we have prepared Rails API to play well with [JBuilder](https://github.com/rails/jbuilder). This library is an alternative within the Rails ecosystem that allows to define JSON responses using templates instead of defining a serializer class as Active Model Serializers does.

Although Rails API is just being incorporated into Rails and further improvements can certainly be done, I hope you feel more confortable and productive implementing your APIs with this new functionality to be shipped in the next Rails version.

Enjoy exploring Rails API!

## Resources

You can find the backend and frontend applications presented in this article in Github:

- [Rails API backend](https://github.com/jmbejar/todo-rails-api-backbone-backend)
- [Backbone frontend](https://github.com/jmbejar/todo-rails-api-backbone-frontend)

The Backbone application was based on the Backbone example included in the [TodoMVC project](http://todomvc.com/).
