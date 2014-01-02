# These are unformatted notes

## Settings

* Make an `application.yml` and `database.yml`. 
* Make examples of the application and database settings by removing the important stuff and adding `.example` to the ends.

## Initializers

Bunch of initializers:

* `after_transaction.rb` Not sure why this is not part of rails but it should be. Bundling things to happen after transactions is important for things like Sidekiq.
* `aws.rb` This was suggested by paperclip. This require really shouldn't be necessary.
* `bitly.rb` If you are working with bitly you need to initialize the api
* `cents_to_dollars.rb` Good little plugin to convert cents to dollars. Even when you have this you should always work with the *_in_cents fields as those are canonical (and numeric as opposed to formatted strings).
* `omniauth.rb` All of your setup for omniauth goes here.
* `paperclip.rb` Eventually this will be split into uploadkit. This is a set of defaults for naming and direct uploading to s3 (works with non-direct upload also).
* `sidekiq.rb` Setup for background jobs.
* `simple_admin.rb` Setup for admin work.

## Lib files

Some crazy lib files (these could be gems, but then gem loading would be EVEN SLOWER):

* `auth_constraint.rb` Really useful lib for auth filtering in routes. Primarily used by sidekiq and simple admin.
* `countries.rb` Just a central location of country names and continents. Hashed together. This is maybe out of date and maybe needs some useful i18n stuff.
* `downcase_routes.rb` this is really controversial... it makes it easier on some servers to always use lowercased routes. When you have tokens this can mess everything up. Not currently using it.
* `email_format_validator.rb` This is great if you want to test that emails are formatted like emails. It is not bulletproof...
* `force_ssl.rb` there are plugins for this, but this is a little more lightweight. You shouldn't need it if you are handling the redirect in nginx. Unless you are parodying... then you might, but then it isn't necessary! So please do investigate. 
* `full_name_splitter.rb` I hate the idea of this one... just ask given name and family name. Or first and last. But this attempts to split with a little bit of smarts.
* `google_client.rb` This is a wrapper for the google client api. It handles key expiration.


## Heroku SSL

References:

* https://devcenter.heroku.com/articles/ssl-endpoint#upload-certificates
* https://gist.github.com/RandomEtc/1222498
* http://lsayers.blogspot.com/2012/08/setting-up-ssl-on-heroku.html