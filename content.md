# API Resources 🤖

Here are some resources for building out an API:

- [Jelani's guide to adding token-based authentication](https://gist.github.com/jelaniwoods/b637851dba743e3084f75f4d27daeee4).
- [as_json](https://api.rubyonrails.org/classes/ActiveModel/Serializers/JSON.html#method-i-as_json) for simple endpoints.
- [JBuilder README](https://github.com/rails/jbuilder#jbuilder) for more complicated endpoints.
- [Graphiti](https://github.com/graphiti-api/graphiti) for flexible endpoints (e.g. for when you are offering API access to third parties).
- [Hoppscotch](https://hoppscotch.io/), [Postman](https://www.postman.com/), or [cURL](https://en.wikipedia.org/wiki/CURL) for testing your API calls.
- [Building Awesome Rails APIs](https://collectiveidea.com/blog/archives/2013/06/13/building-awesome-rails-apis-part-1).

It's very common practice to namespace API only routes. Something like this:

```ruby
Rails.application.routes.draw do
  namespace :api do
    resources :things
  end
end
```

This way all our API only routes will start with `/api/` and API only controllers will be grouped together.

```ruby
Prefix Verb   URI Pattern                    Controller#Action
    api_things GET    /api/things(.:format)          api/things#index
               POST   /api/things(.:format)          api/things#create
 new_api_thing GET    /api/things/new(.:format)      api/things#new
edit_api_thing GET    /api/things/:id/edit(.:format) api/things#edit
     api_thing GET    /api/things/:id(.:format)      api/things#show
               PATCH  /api/things/:id(.:format)      api/things#update
               PUT    /api/things/:id(.:format)      api/things#update
           DELETE /api/things/:id(.:format)      api/things#destroy
```
