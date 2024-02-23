# Building APIs with Rails 🤖

## Introduction to APIs
APIs (Application Programming Interfaces) allow different software applications to communicate with each other. In the context of web development, APIs often refer to web services that send and receive data through the internet. Rails provides robust support for building APIs that can serve JSON, XML, or other formats of data to clients, including web browsers, mobile apps, and other servers.

## Getting Started with Rails API
Rails offers a streamlined way of creating APIs, thanks to its conventions and the plethora of tools it provides. If you're starting a new Rails API-only project, you can use the `--api` option to generate a new application that's pre-configured for API-only applications:

```bash
rails new my_api_app --api
```

This command creates a new Rails application with a configuration that's optimized for API development, removing unnecessary middleware and templates that are typically used for browser-based applications.

## Namespacing API Routes
Organizing your API endpoints into namespaces is a best practice. It helps in logically grouping the API-related controllers and routes, making the codebase easier to maintain and navigate. For example, you can namespace your routes under api, so all your API endpoints will be prefixed with `/api/`:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  namespace :api do
    resources :things
  end
end
```

This way all our API only routes will start with `/api/` and API only controllers will be grouped together. This setup directs requests to `/api/things` to the `ThingsController` under the `api` namespace.

```ruby
Prefix Verb     URI     Pattern                        Controller#Action
    api_things  GET     /api/things(.:format)          api/things#index
                POST    /api/things(.:format)          api/things#create
 new_api_thing  GET     /api/things/new(.:format)      api/things#new
edit_api_thing  GET     /api/things/:id/edit(.:format) api/things#edit
     api_thing  GET     /api/things/:id(.:format)      api/things#show
                PATCH   /api/things/:id(.:format)      api/things#update
                PUT     /api/things/:id(.:format)      api/things#update
                DELETE  /api/things/:id(.:format)      api/things#destroy
```

## Controllers for APIs
In a namespaced API route, controllers are also organized within matching modules. Here's how you can define an API controller:

```ruby
# app/controllers/api/things_controller.rb
module Api
  class ThingsController < ApiController
    # Your actions here, for example:
    def index
      things = User.all
      render json: things
    end
  end
end
```

### ApiController Parent Class
Creating an `ApiController` as a parent class for your API controllers, instead of directly inheriting from `ApplicationController`, is a common and recommended practice for several reasons. This approach allows you to customize the behavior for your API controllers separately from the rest of your application, which is especially useful in larger Rails applications that serve both HTML and API responses. Here are some benefits of using an `ApiController`:

1. **Specialized Configuration**

An ApiController can have configurations and middleware specific to API requests, such as skipping views, CSRF protection (which might not be needed for API calls that use token authentication), and setting a default request format to JSON.

2. **Authentication**

APIs often use token-based authentication (such as JWT) instead of session-based authentication, which is more common in web applications. By inheriting from ApiController, you can apply authentication filters specifically for API endpoints without affecting your main application's authentication mechanism.

3. **Versioning**

If your application provides a versioned API, different versions might require different base configurations or behaviors. Starting with an `ApiController` makes it easier to create version-specific base controllers like `Api::V1::BaseController` that inherit from `ApiController`.

4. **DRY Code**

Common functionalities required across multiple API controllers, such as error handling, parameter sanitization, and response rendering, can be defined in `ApiController`, keeping your individual API controllers DRY (Don't Repeat Yourself) and focused on their specific actions.

Here's an example of what an `ApiController` parent class might look like:

```ruby
# app/controllers/api/api_controller.rb

# ActionController::API includes necessary modules for API functionality but skips those only needed for browser-based applications (like view rendering).
class ApiController < ActionController::API
  before_action :authenticate_request
  respond_to :json

  private

  def authenticate_request
    # Token-based authentication logic here
  end
end
```

## Serializing JSON with JBuilder
While the `as_json` method provides a quick way to serialize objects into JSON, [JBuilder](https://github.com/rails/jbuilder) offers a more powerful and flexible approach for crafting JSON responses. Install JBuilder by adding it to your `Gemfile` and running `bundle install``. 

```ruby
# Gemfile
gem "jbuilder"
```

Here's an example of using `jbuilder` to construct a JSON response:

```ruby
# app/views/api/things/index.json.jbuilder

json.array! @things do |thing|
  json.extract! thing, :id, :created_at, :name
  json.url api_thing_url(thing, format: :json)
end
```

This JBuilder template generates a JSON array of things, including their id, created_at, name, and a URL to each thing's show endpoint.

## Testing APIs
Testing your API endpoints is crucial for ensuring they work as expected. Tools like Hoppscotch, Postman, and cURL are popular for manual API testing. For automated testing, Rails provides a robust framework for writing tests for your controllers and models. 

### RSpec
Here's an example of a simple request spec using RSpec:

```ruby
# spec/controllers/api/things_controller_spec.rb
require 'rails_helper'

RSpec.describe "Things API", type: :request do
  describe "GET /api/things" do
    it "returns all things" do
      get api_things_path
      expect(response).to have_http_status(200)
      expect(JSON.parse(response.body).size).to eq(User.count)
    end
  end
end
```

### MiniTest
Here's an example of a simple test using MiniTest:

```ruby
# test/controllers/api/things_controller_test.rb
require 'test_helper'

class Api::UsersControllerTest < ActionDispatch::IntegrationTest
  setup do
    @thing = things(:one) # Assuming you have fixtures set up
  end

  test "should get index" do
    get api_things_url, as: :json
    assert_response :success
    assert_not_nil assigns(:things) # Ensures @things is assigned in the controller action
    things_response = JSON.parse(response.body)
    assert_equal User.count, things_response.size
    assert things_response.first.has_key?('name') # Checks for a name key in the first thing object
  end
end

```

## Authentication
Securing your API is crucial. Token-based authentication is a common strategy for APIs. Rails has several gems and strategies for implementing authentication, such as [Devise](https://github.com/heartcombo/devise/wiki/How-To:-Simple-Token-Authentication-Example) and [JWT](https://github.com/jwt/ruby-jwt) (JSON Web Tokens). The provided resources offer guides and examples on implementing token-based authentication in your Rails API.


## Documenting Your Rails API with Swagger
In the world of APIs, your documentation is the first impression you make on your users. Good documentation ensures this first impression is both informative and engaging. Providing clear and interactive documentation is as crucial as the API's functionality itself. This is where [Swagger](https://swagger.io/), officially known as the OpenAPI Specification, becomes an indispensable tool. Swagger allows you to describe your API's structure in either YAML or JSON format, enabling you to create comprehensive, interactive documentation effortlessly. 

Here's an [example](https://petstore.swagger.io/). This interactive documentation not only aids your team in understanding the API's capabilities and endpoints but also offers external users a practical way to explore and test the API directly from their browsers.

Integrating Swagger into your Rails project can be streamlined with gems like [rswag](https://github.com/rswag/rswag), which ties together your API documentation and your RSpec tests. This means that as your API evolves and your tests are updated, your API documentation automatically keeps pace, ensuring accuracy and consistency without additional effort. 

## Conclusion
Building APIs with Rails is a powerful way to serve data to various clients in a structured and secure manner. By following Rails conventions, leveraging namespaces, and utilizing tools like JBuilder, MiniTest, and RSpec, you can efficiently create, document, and test your APIs. Remember to secure your API with appropriate authentication mechanisms and continuously test your endpoints for reliability and performance.

This lesson has provided an overview of building APIs with Rails, including routing, controller setup, JSON serialization, and testing. As you become more comfortable with these concepts, you'll find Rails to be an incredibly efficient and enjoyable framework for API development.

## Resources

Here are some resources for building out an API:

- [Jelani's guide to adding token-based authentication](https://gist.github.com/jelaniwoods/b637851dba743e3084f75f4d27daeee4).
- [as_json](https://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html#method-i-as_json) for simple endpoints.
- [JBuilder README](https://github.com/rails/jbuilder#jbuilder) for more complicated endpoints.
- [Graphiti](https://github.com/graphiti-api/graphiti) for flexible endpoints (e.g. for when you are offering API access to third parties).
- [Hoppscotch](https://hoppscotch.io/), [Postman](https://www.postman.com/), or [cURL](https://en.wikipedia.org/wiki/CURL) for testing your API calls.
- [Building Awesome Rails APIs](https://collectiveidea.com/blog/archives/2013/06/13/building-awesome-rails-apis-part-1).
- [Devise](https://github.com/heartcombo/devise/wiki/How-To:-Simple-Token-Authentication-Example)
- [JWT](https://github.com/jwt/ruby-jwt)
