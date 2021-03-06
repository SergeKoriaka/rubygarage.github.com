---
layout: slide
title:  Rails Engine
---

# RailsEngines
Rails Engines is basically a whole Rails app that lives in the container of another one. A Rails application is
actually just a "supercharged" engine, with the Rails::Application class inheriting a lot of its behavior from
Rails::Engine.
[Rails Engine documentation][1]

---

## Generating an engine
```bash
$ rails plugin new shopping_cart --dummy-path=spec/dummy --skip-test-unit --mountable
create
create  README.rdoc
create  Rakefile
create  shopping_cart.gemspec
create  MIT-LICENSE
create  .gitignore
create  Gemfile
create  app
create  app/controllers/shopping_cart/application_controller.rb
create  app/helpers/shopping_cart/application_helper.rb
create  app/mailers
create  app/models
create  app/views/layouts/shopping_cart/application.html.erb
create  app/assets/images/shopping_cart
create  app/assets/images/shopping_cart/.keep
create  config/routes.rb
create  lib/shopping_cart.rb
create  lib/tasks/shopping_cart_tasks.rake
create  lib/shopping_cart/version.rb
create  lib/shopping_cart/engine.rb
create  app/assets/stylesheets/shopping_cart/application.css
create  app/assets/javascripts/shopping_cart/application.js
create  bin
create  bin/rails
vendor_app  spec/dummy
run  bundle install
```

---

## Files structure
shopping_cart.gemspec
```ruby
$:.push File.expand_path("../lib", __FILE__)
# Maintain your gem's version:
require "shopping_cart/version"
# Describe your gem and declare its dependencies:
Gem::Specification.new do |s|
s.name        = "shopping_cart"
s.version     = ShoppingCart::VERSION
s.authors     = ["Vladimir Vorobyov"]
s.email       = ["sparrowpublic@gmail.com"]
s.homepage    = "TODO"
s.summary     = "TODO: Summary of ShoppingCart."
s.description = "TODO: Description of ShoppingCart."
s.license     = "MIT"
s.files = Dir["{app,config,db,lib}/`/*", "MIT-LICENSE", "Rakefile", "README.rdoc"]
s.test_files = Dir["test/`/*"]
s.add_dependency "rails", "~> 4.2.0"
s.add_development_dependency "sqlite3"
end
```
lib/shopping_cart.rb
```ruby
require "shopping_cart/engine"
module ShoppingCart
end
```
lib/shopping_cart/engine.rb
```ruby
module ShoppingCart
class Engine
app/controllers/shopping_cart/application_controller.rb
```ruby
module ShoppingCart
class ApplicationController
config/routes.rb
```ruby
ShoppingCart::Engine.routes.draw do
end
```
Then ensure that this file is loaded at the top of your config/application.rb (or in your Gemfile) and it will
automatically load models, controllers and helpers inside app, load routes at config/routes.rb, load locales at
config/locales/\*, and load tasks at lib/tasks/\*.

---

## Mounting the Engine
Specifying the engine inside the Gemfile would be done by specifying it as a normal, everyday gem Gemfile
```ruby
gem 'shopping_cart', path: "/path/to/shopping_cart"
```
By placing the gem in the Gemfile it will be loaded when Rails is loaded. It will first require lib/shopping_cart.rb
from the engine, then lib/shopping_cart/engine.rb, which is the file that defines the major pieces of functionality
for the engine.
To make the engine's functionality accessible from within an application, it needs to be mounted in that
application's config/routes.rb file.
config/routes.rb
```ruby
mount ShoppingCart::Engine, at: "/cart"
```

---

## Testing with RSpec, Capybara, and FactoryGirl
Add these lines to the gemspec file
```ruby
s.add_development_dependency 'rspec-rails'
s.add_development_dependency 'capybara'
s.add_development_dependency 'factory_girl_rails'
```
Add this line to your gemspec file
```ruby
s.test_files = Dir["spec/`/*"]
```
Modify Rakefile to look like this
```ruby
#!/usr/bin/env rake
begin
require 'bundler/setup'
rescue LoadError
puts 'You must `gem install bundler` and `bundle install` to run rake tasks'
end
APP_RAKEFILE = File.expand_path("../spec/dummy/Rakefile", __FILE__)
load 'rails/tasks/engine.rake'
Bundler::GemHelper.install_tasks
Dir[File.join(File.dirname(__FILE__), 'tasks/`/*.rake')].each {|f| load f }
require 'rspec/core'
require 'rspec/core/rake_task'
desc "Run all specs in spec directory (excluding plugin specs)"
RSpec::Core::RakeTask.new(:spec => 'app:db:test:prepare')
task :default => :spec
```
Create a spec/spec_helper.rb file
```ruby
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path("../dummy/config/environment.rb",  __FILE__)
require 'rspec/rails'
require 'rspec/autorun'
require 'factory_girl_rails'
Rails.backtrace_cleaner.remove_silencers!
# Load support files
Dir["#{File.dirname(__FILE__)}/support/`/*.rb"].each { |f| require f }
RSpec.configure do |config|
config.mock_with :rspec
config.use_transactional_fixtures = true
config.infer_base_class_for_anonymous_controllers = false
config.order = "random"
end
```
Add this config to your engine file
```ruby
module ShoppingCart
class Engine  false
g.fixture_replacement :factory_girl, :dir => 'spec/factories'
g.assets false
g.helper false
end
end
end
```

---

## Configuration
Rails::Engine allows you to wrap a specific Rails application or subset of functionality and share it with other
applications or within a larger packaged application. Since Rails 3.0, every Rails::Application is just an engine,
which allows for simple feature and application sharing.
Any Rails::Engine is also a Rails::Railtie, so the same methods (like rake_tasks and generators) and configuration
options that are available in railties can also be used in engines.
Besides the [Railtie][2] configuration which is shared across the application, in a Rails::Engine you can access
autoload_paths, eager_load_paths and autoload_once_paths, which, differently from a Railtie, are scoped to the
current engine.
lib/shopping_cart/engine.rb
```ruby
class MyEngine

---

## Rack and Middleware stack
### Endpoint
An engine can also be a rack application. It can be useful if you have a rack application that you would like to wrap
with Engine and provide with some of the Engine's features.
```ruby
module MyEngine
class Engine
Now you can mount your engine in application's routes
```ruby
Rails.application.routes.draw do
mount MyEngine::Engine => "/engine"
end
```
### Middleware stack
As an engine can now be a rack endpoint, it can also have a middleware stack.
```ruby
module MyEngine
class Engine

---

## Routes and Mount
### Routes
If you don't specify an endpoint, routes will be used as the default endpoint. You can use them just like you use an
application's routes
```ruby
MyEngine::Engine.routes.draw do
get "/" => "posts#index"
end
```
### Mount
Now you can mount your engine in application's routes
```ruby
Rails.application.routes.draw do
mount MyEngine::Engine => "/blog"
get "/blog/omg" => "main#omg"
end
```

---

## Isolated Engine
```ruby
module MyEngine
class Engine
```ruby
module MyEngine
class FooController
### Routes
config/routes.rb
```ruby
Rails.application.routes.draw do
mount MyEngine::Engine => "/my_engine", as: "my_engine"
get "/foo" => "foo#index"
end
```
You can use the my_engine helper inside your application
```ruby
class FooController  /my_engine/
end
end
```
There is also a main_app helper that gives you access to application's routes inside Engine
```ruby
module MyEngine
class BarController
def index
main_app.foo_path # => /foo
end
end
end
```
### Helpers
For few specific helpers
```ruby
class ApplicationController
For all of the engine's helpers
```ruby
class ApplicationController
### Migrations &amp; seed data
To use engine's migrations
```bash
rake ENGINE_NAME:install:migrations
```
To use engine's seed
```bash
MyEngine::Engine.load_seed
```

---

## Full or mountable
Both options will generate an engine. The difference is that --mountable will create the engine in an isolated
namespace, whereas --full will create an engine that shares the namespace of the main app.
The differences will be manifested in 3 ways.
### The engine class file will call isolate_namespace
#### Full engine
lib/my_full_engine/engine.rb
```ruby
module MyFullEngine
class Engine
#### Mounted engine
lib/my_mountable_engine/engine.rb
```ruby
module MyMountableEngine
class Engine
### The engine's config/routes.rb file will be namespaced
#### Full engine
```ruby
Rails.application.routes.draw do
end
```
#### Mounted engine
```ruby
MyMountableEngine::Engine.routes.draw do
end
```
The parent application could bundle it's functionality under a single route such as
```ruby
mount MyMountableEngine::Engine => "/engine", :as => "namespaced"
```
### The file structure for controllers, helpers, views, and assets will be namespaced
```bash
create app/controllers/my_mountable_engine/application_controller.rb
create app/helpers/my_mountable_engine/application_helper.rb
create app/mailers
create app/models
create app/views/layouts/my_mountable_engine/application.html.erb
create app/assets/images/my_mountable_engine
create app/assets/stylesheets/my_mountable_engine/application.css
create app/assets/javascripts/my_mountable_engine/application.js
create config/routes.rb
create lib/my_mountable_engine.rb
create lib/tasks/my_mountable_engine.rake
create lib/my_mountable_engine/version.rb
create lib/my_mountable_engine/engine.rb
```
[1]: http://guides.rubyonrails.org/engines.html
[2]: https://github.com/rails/rails/blob/master/railties/lib/rails/railtie.rb
