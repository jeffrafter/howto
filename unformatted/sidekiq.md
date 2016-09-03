# Sidekiq

You'll need the gem in your Gemfile:

    gem 'sidekiq'

Next you'll need to install:

    bundle install
    
By default sidekiq assumes you are running redis on your localhost on port 6379. This should work well for development. If you have additional configuration you can set up the connection using an environment variable `REDIS_PROVIDER`. The value is the name of another environment variable used to get the configuration.

If it is your own configuration you can point `REDIS_PROVIDER` to your own custom `REDIS_URL` and then define your redis URL using that variable.
    
You can create a more advanced configuration using `config/initializers/sidekiq.rb` but you usually won't need to.    

## Procfile

Make sure you are referencing Sidekiq in your `Procfile`

    worker: bundle exec sidekiq


## On Heroku


First you need to get the `redistogo` addon:

    heroku addons:docs redistogo
    
Redis To Go has a free version (up to 5MB) which should handle everything you need for basic Sidekiq. As of version 3.0 you need to set the `REDIS_PROVIDER` environment variable to the provided URL. You can do this in the heroku config:
        
    heroku config:set REDIS_PROVIDER=REDISTOGO_URL
    
From there Heroku will do the right thing with the worker definition in your Procfile.


## Image Resizing sample

To resize images in the background you should use ImageMagick. There is a simple gem layer over this called MiniMagick:

    gem 'mini_magick'
    
We'll store images on AWS S3, so you'll need the AWS SDK:
    
    gem 'aws-sdk'
    
Make sure your AWS config is initialized in `config/initializers/aws.rb`:

    require 'aws-sdk'

    AWS.config(
      access_key_id: ENV['AWS_ACCESS_KEY_ID'],
      secret_access_key: ENV['AWS_SECRET_ACCESS_KEY'])

    
Create a new worker file in `app/workers/image_worker.rb`:

    class ImageWorker
      include Sidekiq::Worker

      sidekiq_options :retry => false

      attr_reader :key

      def perform(key)
        @key = key
        return unless obj.exists?
        original = s3.buckets[bucket].objects[key_for('original')]
        return if original.exists?
        obj.copy_to(original, cache_options)
        process
      ensure
        tempfile.close
        tempfile.unlink
      end

      protected

      def process
        # Override if subclass
      end

      def bucket
        Rails.application.secrets.aws_bucket
      end

      def s3
        @s3 ||= AWS::S3.new
      end

      def obj
        @obj ||= s3.buckets[bucket].objects[key]
      end

      def ext
        @ext ||= File.extname(key)
      end

      def etag
        @etag ||= obj.etag.gsub(/"/, '')
      end

      def tempfile
        @tempfile ||= Tempfile.new([obj.etag, ext], Rails.root.join("tmp")).tap do |file|
          file.binmode
          obj.read do |chunk|
            file.write(chunk)
          end
          file.close
        end
      end

      def key_for(style)
        "#{File.dirname(key)}/#{etag}/#{File.basename(key, ext)}-#{style}#{ext}"
      end

      def convert(style)
        name = "#{etag}-#{style}"
        output = Tempfile.new([name, ext], Rails.root.join("tmp"))
        image = MiniMagick::Image.open(tempfile.path)
        yield image
        image.write output.path
        upload(key_for(style), output)
      ensure
        output.close
        output.unlink
      end

      def upload(key, file)
        dest = s3.buckets[bucket].objects[key]
        return if dest.exists?
        dest.write(Pathname.new(file.path), cache_options)
      end

      def cache_options
        {
          acl: :public_read,
          content_type: obj.content_type.to_s,
          cache_control: "max-age=#{1.year.to_i}",
          expires: 1.year.from_now.httpdate
        }
      end
    end


## Sidekiq web

In order to support the built in web interface you need to add Sinatra to your `Gemfile`:

    gem 'sinatra', '>= 1.3.0', :require => nil
    
On Rails 5 you'll need the latest:

    gem 'sinatra', github: 'sinatra'

Then in your `config/routes.rb` you'll need to require it (at the top of the file):

    require 'sidekiq/web'

And mount it (at the end your routes):

    mount Sidekiq::Web => '/sidekiq', :constraints => AuthConstraint.new

Notice that we used a custom constraint for authentication. Using this you can limit the route visibility to logged in users. To make this work you'll need to create the contraint at `lib/auth_constraint.rb`:

    # Constraint used in routing for hosted routed apps
    #
    # https://github.com/mperham/sidekiq/wiki/Monitoring
    #
    class AuthConstraint
      def matches?(request)
        return false unless request.session[:user_id]
        user = User.find request.session[:user_id]
        user && user.admin?
      end
    end

See also: https://github.com/mperham/sidekiq/wiki/Monitoring#rails-http-basic-auth-from-routes

```ruby
Sidekiq::Web.use Rack::Auth::Basic do |username, password|
  username == ENV["SIDEKIQ_USERNAME"] && password == ENV["SIDEKIQ_PASSWORD"]
end if Rails.env.production?
```

## Optimization

* https://github.com/toy/image_optim
* https://github.com/mooktakim/image_optim_bin
* https://www.petekeen.net/introduction-to-heroku-buildpacks
* https://github.com/bobbus/image-optim-buildpack
