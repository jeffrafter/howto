# Shopify embedded app

Start with a basic rails application. Then do the following:

* Setup the secrets for the embedded app
* Add the gems
* Setup omniauth
* 


## Secrets

Edit your `config/secrets.yml`

    defaults: &defaults
      shopify_app_api_key: <%= ENV["SHOPIFY_APP_API_KEY"] %>
      shopify_app_api_secret: <%= ENV["SHOPIFY_APP_API_SECRET"] %>

Add the environment variables to the `.env`:

    SHOPIFY_APP_API_KEY=12345abcdef....
    SHOPIFY_APP_API_SECRET=12345abcdef....
    
    
## Gems

Add the following to your `Gemfile`:

    gem 'shopify_api'
    gem 'omniauth'
    gem 'omniauth-shopify-oauth2', git: 'git://github.com/jeffrafter/omniauth-shopify-oauth2'
    
Then `bundle`.

## Setup omniauth

Omniauth is middleware which is setup in an initializer. Create `config/initializers/omniauth.rb`:

    # See http://docs.shopify.com/api/tutorials/oauth for all scopes
    SHOPIFY_SCOPES = [
      'read_content',
      'write_content',
      'read_products',
      'write_products',
      'read_customers',
      'write_customers',
      'read_orders'
    ]

    Rails.application.config.middleware.use OmniAuth::Builder do
      provider :shopify,
        Rails.application.secrets.shopify_app_api_key,
        Rails.application.secrets.shopify_app_api_secret,
        :scope => SHOPIFY_SCOPES.join(', '),
        :setup => lambda {|env|
          params = Rack::Utils.parse_query(env['QUERY_STRING'])
          site_url = "https://#{params['shop']}"
          env['omniauth.strategy'].options[:client_options][:site] = site_url
        }
    end


## Shopify Models

    rails g model shop_session shop_id:integer uid:string access_token:string ip:string user_agent:string access_at:datetime logout_at:datetime revoked_at:datetime


    class CreateShopSessions < ActiveRecord::Migration
      def change
        create_table :shop_sessions do |t|
          t.string :url
          t.string :uid
          t.string :access_token
          t.string :ip
          t.string :user_agent
          t.datetime :access_at
          t.datetime :logout_at
          t.datetime :revoked_at

          t.timestamps null: false
        end

        add_index :shop_sessions, [:uid, :access_token]
        add_index :shop_sessions, [:revoked_at, :logout_at, :access_at]
      end
    end
    
    
 Add a site method for the API
 
     def site
       "https://#{self.url}/admin"
     end
     
     def token
       self.access_token
     end

## Shopify Controller

In order to handle the interactions in Shopify you will want a specific controller. Create `app/controllers/shopify_controller.rb`:


https://gist.github.com/jeffrafter/5f3184de69d6737d2898


# Shopify Layout

https://gist.github.com/jeffrafter/a4ac26761d6a5e0552ce

# Views


New:

    Shopify new

    <div class="page-header" id="login-header">
      <h1>Install This App</h1>
      <h1><small>This app requires you to login to start using it.</small></h1>
    </div>

    <div style="width:340px; margin:0 auto;">
      <%= form_tag :shopify_login, method: :get, class: 'form-search' do %>
        <%= text_field_tag 'shop', nil, placeholder: 'Shop URL', class: 'btn-large' %>
        <%= submit_tag 'Install', class: 'btn btn-primary btn-large' %>
      <% end %>
    </div>

    <p align="center" style="margin-top:30px">
      <%= image_tag 'shopify.png' %>
    </p>

Welcome:


    Shopify welcome!
    <br>
    <br>
    [<%= shop_session.shop %>]
    <br>
    <br>
    <strong><%= flash[:notice] %></strong>







# Routes

    Rails.application.routes.draw do
      root "shopify#welcome"

      get '/shopify/welcome', to: 'shopify#welcome', as: :shopify_welcome
      get '/shopify/products', to: 'shopify#products', as: :shopify_products

      get '/shopify/login', to: 'shopify#new', as: :shopify_login
      post '/shopify/login', to: 'shopify#create'
      get '/shopify/logout', to: 'shopify#destroy', as: :shopify_logout
      get '/auth/shopify/callback', to: 'shopify#create', as: :shopify_callback
    end






Now go to this url:

https://snowe-shopify.herokuapp.com/shopify/login?shop=embedded-test


https://docs.shopify.com/api/authentication/oauth




















http://embedded-test.myshopify.com/admin/admin/api_clients/new?shop=embedded-test

https://embedded-test.myshopify.com/admin/api_clients/new?shop=embedded-test&api_client[return_url]=http://snowe-shopify.herokuapp.com/shopify/login



https://embedded-test.myshopify.com/admin/oauth/authorize?client_id=9498a7cd6d0e1a56547c5c618bd8c004&scope=read_content,write_content,read_products,write_products, read_customers,write_customers,read_orders&redirect_uri=https://snowe-shopify.herokuapp.com/welcome

read_content,write_content,read_products,write_products, read_customers,write_customers,read_orders




https://snowe-shopify.herokuapp.com/auth/shopify/callback?code=5bb8f92ff8c7da37eaba890020ed2f45&hmac=6dfa0f1e0b56bb3c4d8390c96634d693a699be896ba7c6e211e14f630d2d502d&shop=embedded-test.myshopify.com&signature=b80e78f1c664bca5dc2c2178905709cb&timestamp=1428965125


# Safari and IE embedded IFrame Cookies

http://www.mendoweb.be/blog/internet-explorer-safari-third-party-cookie-problem/


# Validating the HMAC

The problem is that in many requests the HMAC is blank. For example lets say you go to

    https://<your-site>/shopify/welcome
    
This will hit your controller and render the welcome page (without an HMAC). When this page renders it will render the JavaScript which runs the shopify app embed which will reframe your page within the Shopify Admin:

    https://<your-shop>.myshopify.com/admin/apps/<your-app-no>/shopify/welcome
    
This page will re-render your welcome page but this time with the shop and the hmac params included:

    https://<your-site>/shopify/welcome?hmac=12345&signature=12345&timestamp=12345&shop=<your-shop>.myshopify.com
    
    
 