# Shopify API interaction

You'll want to talk to the API and the best way to do this is using the Shopify API gem. Add the following to your `Gemfile`

    gem 'shopify_api'
    gem 'rest-client'

Then `bundle`.

## API keys

In order to get started you are going to need to setup your application as a _Private Application_ on your shop. To do this, create a new private application: 

    https://YOURSHOP.myshopify.com/admin/apps/private/new

Once created, go to the application and find the API keys, add them to your `.env`:

    SHOPIFY_API_KEY="YOUR_API_KEY"
    SHOPIFY_PASSWORD="YOUR_PASSWORD"
    SHOPIFY_SHOP="YOUR_SHOP_NAME"


Once you have the API keys in your `.env` you should import them into your `config/secrets.yml`:

    shopify_api_key: <%= ENV["SHOPIFY_API_KEY"] %>
    shopify_password: <%= ENV["SHOPIFY_PASSWORD"] %>
    shopify_shop: <%= ENV["SHOPIFY_SHOP"] %>


## Initializer

Once you've setup your API keys you'll want to preload them in an initializer. Create `/config/initializers/shopify.rb`:

    shopify_shop_url = "https://#{Rails.application.secrets.shopify_api_key}:#{Rails.application.secrets.shopify_password}@#{Rails.application.secrets.shopify_shop}.myshopify.com/admin"
    ShopifyAPI::Base.site = shopify_shop_url


## Metafields


product.add_metafield(ShopifyAPI::Metafield.new({
   :description => 'Author of book',
   :namespace => 'book',
   :key => 'author',
   :value => 'Kurt Vonnegut',
   :value_type => 'string'
}))