Setup for New Ruby on Rails Project
=

*Disclaimer: The following guide has not been tested extensively. Please use it at your own risk!*

## Create Rails Project
```bash
rails new <project name> --database=postgresql -T
cd <project name>
rvm gemset create <gemset name>
rvm gemset use <gemset name>
touch .ruby-version
touch .ruby-gemset
bundle install
```
```
# .ruby-version
2.4.1
```
```
# .ruby-gemset
<gemset name>
```

## Gems Installation

Instructions:
1. You may skip any gem that is not required.
2. Please run 'bundle' after each gem added in gemfile, even if not stated explicitly.
3. Highly recommend to commit after each gem installation.
4. Below is my preferred order of installation. You can install in any order, but be mindful of the interactions between gems (e.g Devise and Simple Form)

#### [Slim Template](https://github.com/slim-template/slim-rails)
```ruby
gem "slim-rails"
```

#### [RSpecs](https://github.com/rspec/rspec-rails)
```ruby
# group :development, :test
gem 'rspec-rails', '~> 3.7'
```
```bash
# Terminal
bundle
rails generate rspec:install
rspec
```

#### [Shoulda Matchers](https://github.com/thoughtbot/shoulda-matchers)
```ruby
# group :development, :test
gem 'shoulda-matchers', '~> 3.1'
```
```ruby
# rails_helper.rb
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```
```bash
# Terminal
bundle
rspec
```

#### [Factory Bot Rails](https://github.com/thoughtbot/factory_bot_rails)
```ruby
# group :development, :test
gem 'factory_bot_rails'
```
```ruby
# rails_helper.rb
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```
```bash
# Terminal
bundle
rspec
```

#### [Rails-Controller-Testing](https://github.com/rails/rails-controller-testing)
```ruby
# For 'assigns' to work
gem 'rails-controller-testing'
```

#### [Bootstrap](https://github.com/twbs/bootstrap-rubygem)
```ruby
gem 'bootstrap', '~> 4.0.0'
```
```scss
# app/assets/stylesheets/application.scss
# Remove all the *= require and *= require_tree
@import "bootstrap";
@import "/*";
```

#### [jQuery](https://github.com/rails/jquery-rails)
```ruby
gem 'jquery-rails'
```
```js
# application.js
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

#### [Simple Form](https://github.com/plataformatec/simple_form)
```ruby
gem 'simple_form'
```
```bash
# Terminal
bundle
rails generate simple_form:install --bootstrap #bootstrap option is optional
```

#### [Devise](https://github.com/plataformatec/devise)
```ruby
gem 'devise'
```
```bash
# Terminal
bundle
rails generate devise:install

# Models generation
rails generate devise MODEL
rails db:migrate

# Views generation
rails generate devise:views # for single model
# or
rails generate devise:views <resources> # for multiple models
```
```ruby
# config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

##### [Devise RSpecs](https://github.com/plataformatec/devise)
```ruby
# rails_helper.rb
RSpec.configure do |config|
  config.include Devise::Test::ControllerHelpers, type: :controller
  config.include Devise::Test::ControllerHelpers, type: :view
end
```

#### [dotenv](https://github.com/bkeepers/dotenv)
```ruby
# group :development, :test
gem 'dotenv-rails'
```
```bash
# Terminal
bundle
touch .env
```
```
# .gitignore
.env
```

#### [OmniAuth - Facebook](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview)
```ruby
gem 'omniauth-facebook'
```
```bash
# Terminal
bundle
rails g migration AddOmniauthToUsers provider:string uid:string
rake db:migrate
```
```ruby
# config/initializers/devise.rb
config.omniauth :facebook, ENV["APP_ID"], ENV["APP_SECRET"]
```
```ruby
# .env
APP_ID = <your app id>
APP_SECRET = <your app secret>
```
```ruby
# app/models/user.rb
devise :omniauthable, :omniauth_providers => [:facebook]
```
```slim
Views
= link_to "Sign in with Facebook", user_facebook_omniauth_authorize_path
```
```ruby
# config/routes.rb
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
```
```ruby
# app/controllers/users/omniauth_callbacks_controller.rb (to be created)
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
```
```ruby
# app/models/user.rb
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
    # If you are using confirmable and the provider(s) you use validate emails, 
    # uncomment the line below to skip the confirmation emails.
    # user.skip_confirmation!
  end
end
```

#### [Mailer](http://culttt.com/2016/03/02/getting-started-with-action-mailer-in-ruby-on-rails/)
```bash
# Terminal
rails generate mailer UserMailer
```
```ruby
# application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout "mailer"
end
```
[Official guide](http://guides.rubyonrails.org/action_mailer_basics.html)

#### [MailCatcher](https://mailcatcher.me/)
```ruby
gem 'mailcatcher'
```
```ruby
# environments/development.rb
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }
```
```bash
# Terminal
mailcatcher
```
Go to http://localhost:1080/

#### [Filestack](https://github.com/filestack/filestack-rails)
```ruby
gem 'filestack-rails', '~> 3.1.0'
```
```bash
# Terminal
bundle
```
```html
<!-- After <%= csrf_meta_tags %> -->
<%= filestack_js_include_tag %>
<%= filestack_js_init_tag %>
<!-- Before other scripts -->
```
```ruby
# config/application.rb
config.filestack_rails.api_key = ENV['FILESTACK_KEY']
```
```ruby
# .env
FILESTACK_KEY = <your filestack key>
```
