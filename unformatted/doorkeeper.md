Setting up an api with doorkeeper
=================================

Doorkeeper lets you to turn your Rails application into an OAuth2 provider. This allows you to serve API clients on an application by application basis. It also allows your mobile application to talk with your server in a standardized way.

If your application has no front-end and is only an API you may want to consider using [Rails-api](https://github.com/rails-api/rails-api).

## Installing

In your Rails application add the following to your `Gemfile`:

    gem 'doorkeeper'
    
> Note: at the time of this writing you may want to point to the `master` branch as the newest versions of Doorkeeper have lots of changes.
    
Once you have added the gem, you'll need to run Bundler:

    bundle
    
Now that you have Doorkeeper you need to install the basics (this assumes you are using ActiveRecord to access your database):  

    bundle exec rails g doorkeeper:install
    bundle exec rails g doorkeeper:migration
    
Now run the migrations:    

    bundle exec rake db:migrate
    bundle exec rake RALS_ENV=test db:migrate
    
## Configure

After installing Doorkeeper you'll want to adjust `config/initializers/doorkeeper.rb`.

Make sure that the ORM is set to `:active_record`:

    # Change the ORM that doorkeeper will use.
    # Currently supported options are :active_record, :mongoid2, :mongoid3, :mongo_mapper
    orm :active_record

For your own API clients (your mobile application for example) you'll want to support credential based logins (email and password). In Oauth2 this are specified in the "Resource Owner Password Credentials" flow
  - [http://tools.ietf.org/html/rfc6749#section-1.3.3](http://tools.ietf.org/html/rfc6749#section-1.3.3)

    # Support credential based logins
    resource_owner_from_credentials do |routes|
      email = "#{params[:username]}".downcase
      return if email.blank?
      user = User.where('email = ?', email).first
      user if user && user.authenticate(params[:password])
    end
    
If you are supporting a traditional web-based OAuth2 flow, access grants are based on the currently logged in user. This comes from your traditional session which may be provided by Warden, Devise or your own authentication (for example Authkit):

    # This block will be called to check whether the resource owner is authenticated or not.
    resource_owner_authenticator do
      return @current_user if defined?(@current_user)
      @current_user ||= User.where(id: session[:user_id]).first if session[:user_id]
      redirect_to(login_path) unless @current_user
    end

Or if you are using user sessions:

    # This block will be called to check whether the resource owner is authenticated or not.
    resource_owner_authenticator do
      return @current_user_session.user if defined?(@current_user_session)
      @current_user_session ||= UserSession.active.where(id: session[:user_session_id]).first if session[:user_session_id]
      redirect_to(login_path) unless @current_user_session
      @current_user_session && @current_user_session.usr
    end

This is required because Doorkeeper does not have access to your `current_user` method.

You need to enabled the `password` grant flow for your mobile application to work:

    grant_flows %w(password)


## Base Controller

You probably don't want your API actions to behave in the same way your normal application does. Because of this, it is helpful to create a base `ApiController` to descend from. 

It is also helpful to version your API. There are lots of ways to do this effectively, but the simplest is to segregate your API versions by namespace. To do this create a new controller path:

    mkdir -p app/controllers/api/v1

Next create an `api_controller.rb`:

    # app/controller/api/v1/api_controller.rb
    module Api
      module V1
        class ApiController < ActionController::Base
          respond_to :json
          
          protected
          
          # Protected methods
          
        end
      end
    end

If you are using Rails-api you can descend instead from `ActionController::API`. 

>In controllers, any method that is not an action should be private or protected. Because we want descendent classes to be able to use the methods we should add the following in the `protected` area.

When using Doorkeeper it is helpful to define a `current_resource_owner` method which allows you treat the current OAuth user as you would a normal session user:

    def current_resource_owner
      return @current_resource_owner if defined?(@current_resource_owner)
      @current_resource_owner = User.find(doorkeeper_token.resource_owner_id) if doorkeeper_token
    end

When API clients interact with our API they can include the application `uid` in a request header. Typically this is only used by known clients outside of the traditional OAuth workflow. We'll add a convenience method (which will simplify our specs):

    def application_id
      request.headers["HTTP_X_APPLICATION_ID"]
    end

In cases where we are specifying the application via a request header we want to be able to require it:

    def require_oauth_application
      render(status: 403, nothing: true) unless oauth_application
    end

    def oauth_application
      return @oauth_application if defined?(@oauth_application)
      @oauth_application = Doorkeeper::Application.find_by_uid(application_id)
    end

Note that there is very little protection around this value and it can be easily spoofed.

## Default application

When providing an API interface for your own mobile application you'll want to pre-approve the application for your own users. In fact, it is helpful to create a pre-approved application for each distinct mobile client (iOS and Android for example).

It is easiest to create the application in `rails console`:

    app = Doorkeeper::Application.where(
      name: 'YOUR APP NAME iOS',
      redirect_uri: 'https://api.example.com').first_or_create

Once created, you'll want to copy the `uid` and `secret` which were autogenerated:
  
    puts app.uid
    puts app.secret
        
You'll want to store these values securely in your settings (and make them available in your environment or via `Rails.application.secrets.*`. You could make a second set of credentials for your development and test environments for consistency. 

Next in your `db/seeds.rb` file, you'll want to create the application with those values (or do this manually if your database is already seeded).

    Doorkeeper::Application.where(
      name: 'YOUR APP NAME iOS',
      redirect_uri: 'https://api.example.com',
      uid: ENV['DOORKEEPER_IOS_UID'],
      secret: ENV['DOORKEEPER_IOS_SECRET']
    ).first_or_create

## Signing up via the API

In many cases there is no website for your visitors to see. Still, new users need to sign up and this can be provided via your API. In `app/controllers/api/v1/signup_controller.rb` add the following:

    module Api
      module V1
        class SignupController < ApiController
          before_filter :require_oauth_application, only: [:create]

          # POST /api/v1/signup
          def create
            @signup = Signup.new(signup_params)

            if @signup.save
              access_token = Doorkeeper::AccessToken.create(
                application_id: oauth_application.id,
                resource_owner_id: @signup.user.id,
                use_refresh_token: Doorkeeper.configuration.refresh_token_enabled?,
                expires_in: Doorkeeper.configuration.access_token_expires_in)

              render json: {
                access_token: access_token.token,
                refresh_token: access_token.refresh_token
              }
            else
              render json: { errors: @signup.errors }
            end
          end

          protected

          def signup_params
            params.require(:signup).permit(
              :email,
              :password,
              :first_name,
              :last_name,
              :phone_number,
              :time_zone)
          end
        end
      end
    end

This assumes you have a Signup form (or model) which simplifies the controller. When the server receives a signup POST it creates a new user and immediately grants an access token. The access token is locked to the application that was specified in the request header.

Warning: The request header can be easily spoofed (in many cases it can be easily intercepted or extracted from your mobile binary via decompilation). Because of this, there is no way to guarantee that the client making the signup request is your application. A spoofed request will still return a valid access token. This may mean users can access your API via alternate clients. While this may be beneficial in many cases, your API must be created with this expectation.

If you need more control over which clients can access your API you may need to require that all signup be completed on your website.

You'll need to add a route for your action:

    namespace :api do
      namespace :v1 do
        post "signup", to: "signup#create"
      end
    end

## Protecting controllers and actions

Once you have Doorkeeper setup, protecting actions is easy:

    # app/controllers/api/v1/users_controller.rb
    module Api
      module V1
        class UsersController < ApiController
          before_action :doorkeeper_authorize!

          def me
            respond_with current_resource_owner
          end
        end
      end
    end

Doorkeeper's `before_action` allows you to authorize oauth users based on access tokens and scopes. Once authorized, you can access the authenticated user using `current_resource_owner`. 

In this example we respond directly with the object which will convert it to JSON. You probably don't want to return the entire User object like this. Instead you would want to use 
[Active Model Serializers](https://github.com/rails-api/active_model_serializers) or
[Rabl](https://github.com/nesquena/rabl) or
[JBuilder](https://github.com/rails/jbuilder).

You'll need to add a route for your action:

    namespace :api do
      namespace :v1 do
        get "me", to: "users#me"
      end
    end


## Specs

## Using Curl

To simulate your API you need three basic actions:

* Signup
* Login
* Logout
* Me

### Signup

To signup you will need to include the OAuth application id and the signup params:

    curl -X POST 'http://localhost:3000/api/v1/signup' \
      --header 'X-Application-Id: YOUR APPLICATION ID' \
      -d 'signup[first_name]=Example' \
      -d 'signup[last_name]=User' \
      -d 'signup[email]=test@example.com' \
      -d 'signup[password]=password' \
      -d 'signup[password_confirmation]=password'

You should see something like:

    {"access_token":"3210b5a33fc6728e7531cffbeed1d4004898343fa655bd826efa1df57a088779","refresh_token":null}

If you run it again you should see an error:

    {"errors":{"email":["has already been taken"]}}

### Login
    
If you are connecting from mobile application you'll need to be able to obtain a grant using your credentials:

    curl -X POST 'http://localhost:3000/oauth/token' \
      --header 'X-Application-Id: YOUR APPLICATION ID' \
      -d 'username=test@example.com' \
      -d 'password=password' \
      -d 'grant_type=password'

This will give you something like:

    {"access_token":"d922e31c20c7014fa4359f67131d2322d3f3eabb7e3999837de0829cc7364f28","token_type":"bearer","expires_in":7200}

If you want to issue non-expiring tokens change the following in `config/initializers/doorkeeper.rb`:

    # Access token expiration time (default 2 hours).
    # If you want to disable expiration, set this to nil.
    access_token_expires_in nil

### Logout

In some cases, logging out simply means forgetting the token. However, you can also revoke the token which will invalidate it across any clients and for force them to relogin:

    curl -X POST 'http://localhost:3000/oauth/revoke' \
      --header 'Authorization: Bearer YOUR ACCESS TOKEN'
 

## AFNetworking


# REMEMBER

* You must rate limit token failures
* You must filter tokens from parameter logging




https://github.com/doorkeeper-gem/doorkeeper/blob/ac4da86032f0e949dfe98753c57b3de1858ca062/lib/doorkeeper/rails/helpers.rb


