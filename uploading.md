# Uploading Files in Rails

When you upload files in Rails. The main things you want to consider are:

* Upload size and timeouts
* Reliability of storage
* Post processing
* Importing
* Progress in the browser

In Rails, the generally accepted ways that people upload files are using Paperclip or CarrierWave (historically, attachment_fu was popular). 

### Upload size and timeouts

Uploading large files takes a long time. While the upload is being transferred a connection is kept open and this can lead to resource starvation. If all of your users suddenly upload a large file simultaneously will you run out of Nginx connections? If the files are large, will Nginx timeout before the upload is completed (see client_max_body_size)? 

Once the file is uploaded, it is handed to Rails where rails parses the params, including the uploaded file. This takes still longer. If you need to process the uploaded file before returning a response you can end up with a very long request/response. 

In general, when you are first launching your site you don't need to worry about this. Small images for avatars won't bog down the server and users don't upload them very often. If you are working with large media, high res photos, or you are expecting lots of uploads then you should look at offloading the process.

#### Offload processing to Nginx

First you can **move the upload processing and progress to Nginx**. These modules are not standard and must be compiled in. In the chef section later, you'll see we include these already, just in case. With these in place, Nginx will handle the upload and dump the result to a /tmp file. It will parse the paramters (in C, not Ruby) and pass the parsed parameters Rails. This alone can result in 10X performance on uploads.

To use this...

#### Direct upload to S3

The second option is to not upload files to your server at all. Instead you can upload directly to S3. Most people use s3_direct_upload for this, but I use uploadkit because it is simpler (though less flexible). The general principle allows you server to generate a secure upload URL to your S3 bucket. This upload URL is signed and expires. Once the upload is complete your server may need to process the result. There are two strategies here:

1. Use AJAX to upload the file, on completion post back to the server
2. Override the uploaded filename and poll S3 for completion

I generally opt for the first. The workflow is:

1. User navigates to the upload page, server includes a form for direct upload to S3.
2. User uploads the file (handled via AJAX, with progress events).
3. Upload completes, in the success handler a POST is made to the server with the completed file key (or the whole URL).
4. Server creates the record, returns an immediate response with the record in "importing" state and destination URL (the URL of the object when importing and processing is completed).
5. Background job imports the uploaded file from S3, processes the styles, and changes the state of the record.
6. Browser polls S3 for the completed file, shows importing/processing image until the file is complete.


Paperclip initializer

```
Paperclip::Attachment.default_options.merge!(
  restricted_characters: nil,
  escape_url:            false,
  storage:               :s3,
  default_url:           ':s3_default_url',
  url:                   ':s3_domain_url',
  path:                  '/system/:class/:style/:hash/:filename',
  hash_data:             ':class/:attachment/:id/:style/:uploaded_at',
  hash_secret:           '60532e36ecd55729de0ffda1df42c82dbbd73b3ea7c622cbf45ef9f75ec1663bb53a36f4b8845e5f35abe552cfee9fd188ba07cb31f628aa1249355d5177d808',
  s3_permissions:        :private,
  s3_protocol:           'https',
  s3_credentials:        {
    bucket:            ENV['aws_bucket'],
    access_key_id:     ENV['aws_access_key_id'],
    secret_access_key: ENV['aws_secret_access_key']
  }
)

# When importing, the updated_at key changes when the remote_url is
# actually imported, which changes the hash. If you use the time that
# the remote_url is set instead (attachment_uploaded_at) then you can still
# have a unique key that changes as the content changes.
Paperclip.interpolates :uploaded_at do |attachment, style_name|
  return attachment.instance.attachment_uploaded_at.to_i.to_s
end

Paperclip.interpolates :s3_default_url do |attachment, style_name|
  style_name = "thumb"
  classname = plural_cache.underscore_and_pluralize(attachment.instance.class.to_s)
  missing_path = "/images/#{classname}/#{style_name}/missing.png"
  return missing_path if attachment.nil? || attachment.instance.blank? || attachment.instance.remote_url.blank?
  uri = URI.parse(attachment.instance.remote_url) rescue nil
  return missing_path unless uri
  original_filename = File.basename(URI.decode(uri.path))
  basename = original_filename.gsub(/#{Regexp.escape(File.extname(original_filename))}$/, "")
  extension = ((style = attachment.styles[style_name.to_s.to_sym]) && style[:format]) || File.extname(original_filename).gsub(/^\.+/, "")
  filename = [basename, extension].reject(&:blank?).join(".")
  hash = attachment.hash_key(style_name)
  path = "system/#{classname}/#{style_name}/#{hash}/#{filename}"
  if attachment.s3_permissions(style_name) == "public-read"
    UploadHelper.aws_bucket_url + "/" + path
  else
    # We still get an authenticated url for the path, it is just based on the destination rather than the current
    UploadHelper.aws_url_for(path)
  end
end
```

Upload Helper:

    # Allow direct uploads to Amazon S3. Uploads are segmented by the
    # Everything is uploaded into an separate paths based on the current user
    # so that files are not overwritten when multiple people upload different
    # files with the same name.
    #
    # In order to use this helper you must have the following environment
    # variables set:
    #
    #   aws_access_key_id
    #   aws_secret_access_key
    #   aws_bucket
    #
    # Configure the AWS bucket with the following CORS policy
    #
    #   <CORSConfiguration>
    #     <CORSRule>
    #       <AllowedOrigin>http://0.0.0.0:3000</AllowedOrigin>
    #       <AllowedOrigin><%= ENV['domain'] %></AllowedOrigin>
    #       <AllowedMethod>GET</AllowedMethod>
    #       <AllowedMethod>POST</AllowedMethod>
    #       <AllowedMethod>PUT</AllowedMethod>
    #       <MaxAgeSeconds>3600</MaxAgeSeconds>
    #       <AllowedHeader>*</AllowedHeader>
    #     </CORSRule>
    #   </CORSConfiguration>
    #
    module UploadHelper

      # Use this as part of the XHR data params for the upload form
      # Options include:
      #
      #   expires_at: defaults to 1 hour from now
      #   max_file_size: defaults to 5GB (not currently used)
      #   acl: defaults to authenticated-read
      #   starts_with: defaults to "uploads/#{h(current_user.to_param)}/#{SecureRandom.hex}"
      #
      def aws_upload_params(options={})
        expires_at = options[:expires_at] || 1.hours.from_now
        max_file_size = options[:max_file_size] || 5.gigabyte
        acl = options[:acl] || 'authenticated-read'
        hash = "#{SecureRandom.hex}"
        starts_with = options[:starts_with] || "uploads/#{hash}"
        bucket = ENV['aws_bucket']
        # This used to include , but it threw Server IO errors
        policy = Base64.encode64(
          "{'expiration': '#{expires_at.utc.strftime('%Y-%m-%dT%H:%M:%S.000Z')}',
            'conditions': [
              ['starts-with', '$key', '#{starts_with}'],
              ['starts-with', '$hash', '#{hash}'],
              ['starts-with', '$utf8', ''],
              ['starts-with', '$x-requested-with', ''],
              ['eq', '$success_action_status', '201'],
              ['content-length-range', 0, #{max_file_size}],
              {'bucket': '#{bucket}'},
              {'acl': '#{acl}'},
              {'success_action_status': '201'}
            ]
          }").gsub(/\n|\r/, '')

        signature = Base64.encode64(
                      OpenSSL::HMAC.digest(
                        OpenSSL::Digest::Digest.new('sha1'),
                        ENV['aws_secret_access_key'], policy)).gsub("\n","")

        return {
          "key" => "#{starts_with}/${filename}",
          "hash" => "#{hash}",
          "utf8" => "",
          "x-requested-with" => "",
          "AWSAccessKeyId" => "#{ENV['aws_access_key_id']}",
          "acl" => "#{acl}",
          "policy" => "#{policy}",
          "signature" => "#{signature}",
          "success_action_status" => "201"
        }
      end

      # Use aws_upload_tags when embedding the upload params in a form.
      # For example:
      #
      #   <form action="<%= aws_upload_url %>" method="post" enctype="multipart/form-data" id="avatar_form">
      #     <input type="file" name="file" id="avatar_attachment">
      #     <%= aws_upload_tags %>
      #   </form>
      #
      def aws_upload_tags
        aws_upload_params.each do |name, value|
          concat text_field_tag(name, value)
        end
        nil
      end

      # The destination host for the upload always uses https even though it can be slower.
      # Safe and sure wins the race
      def aws_bucket_url
        UploadHelper.aws_bucket_url
      end

      def self.aws_bucket_url
        "https://#{ENV['aws_bucket']}.#{ENV['aws_region'] || 's3'}.amazonaws.com"
      end

      # Authenticated url for S3 objects with expiration. By default the URL will expire in
      # 1 day. The path should not include the bucket name.
      def aws_url_for(path, expires=nil)
        UploadHelper.aws_url_for(path, expires)
      end

      def self.aws_url_for(path, expires=nil)
        path = "/#{path}" unless path =~ /\A\//
        path = URI.encode(path)
        expires ||= 1.day.from_now
        string_to_sign = "GET\n\n\n#{expires.to_i}\n/#{ENV['aws_bucket']}#{path}"
        signature = Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest::Digest.new('sha1'), ENV['aws_secret_access_key'], string_to_sign)).gsub("\n","")
        query_string = URI.encode_www_form("AWSAccessKeyId" => ENV['aws_access_key_id'], "Signature" => signature, "Expires" => expires.to_i)
        "#{aws_bucket_url}#{path}?#{query_string}"
      end
    end

Avatar model:

    class Avatar < ActiveRecord::Base
      has_attached_file :attachment,
        styles: {thumb: "200x200#"},
        s3_permissions: 'public-read',
        s3_headers: {"Cache-Control" => "max-age=#{1.year.to_i}", "Expires" => 1.year.from_now.httpdate}

      before_validation :prepare_import
      validates :attachment, attachment_presence: true, unless: :attachment_importing?
      after_save :async_import

      def as_json(options = {})
        super({
          methods: [:url]
        }.merge(options))
      end

      def url
        # self.attachment.expiring_url(3600, :thumb)
        self.attachment.url(:thumb)
      end

      def import!
        return false unless self.attachment_importing?
        self.attachment_importing = false
        if self.remote_url.present?
          uri = URI.parse(self.remote_url)
          self.attachment = uri
          self.attachment_file_name = File.basename(URI.decode(uri.path))
        end
        self.save
      end

      protected

      def prepare_import
        return unless self.remote_url.present? && self.remote_url_changed?
        self.attachment_importing = true
      end

      def async_import
        AvatarImportWorker.perform_async(self.id) if self.attachment_importing?
      end
    end

    # == Schema Information
    #
    # Table name: avatars
    #
    #  id                      :integer          not null, primary key
    #  top                     :integer          default(0)
    #  left                    :integer          default(0)
    #  width                   :integer          default(0)
    #  height                  :integer          default(0)
    #  created_at              :datetime
    #  updated_at              :datetime
    #  attachment_file_name    :string(255)
    #  attachment_content_type :string(255)
    #  attachment_file_size    :integer
    #  attachment_updated_at   :datetime
    #  attachment_importing    :boolean          default(FALSE)
    #  attachment_hash         :string(255)
    #  remote_url              :text


In the edit form:

    <!-- Upload form stubs -->
    <div style="display:none">
      <form action="<%= aws_bucket_url %>" method="post" enctype="multipart/form-data" id="avatar_form">
        <input type="file" name="file" id="avatar_attachment">
        <%= aws_upload_tags %>
      </form>
    </div>
    
Background worker:

```
class AvatarImportWorker
  include Sidekiq::Worker

  sidekiq_options queue: "default"
  sidekiq_options retry: false

  def perform(avatar_id)
    avatar = Avatar.find(avatar_id)
    avatar.import!
  end
end
```

In the upload controller:

```
class UploadController < ApplicationController
  before_filter :aws_key_to_remote_url, only: [:avatar]

  ...
  
  def avatar
    @avatar = current_user.build_avatar(avatar_params)

    # Each new upload needs to be unique in case the same file is uploaded twice with different content
    @avatar.attachment_uploaded_at = Time.now

    if current_user.save
      respond_to do |format|
        format.json { render json: @avatar }
      end
    else
      respond_to do |format|
        format.json { render json: { status: 'error', errors: @avatar.errors }.to_json, status: 422 }
      end
    end
  end
  
  ...

  protected
  
  def avatar_params
    params.require(:avatar).permit(:attachment, :remote_url)
  end

  def aws_key_to_remote_url
    params[:avatar] ||= {}
    key = params[:avatar][:key]
    params[:avatar][:remote_url] ||= view_context.aws_url_for(key) if key.present?
  end
   
``` 

In the Javascript:

```
jQuery(function($) {
  var avatarXhr;
  var avatarTimeout;
  var avatarImage;

  function postAvatar(key) {
    if (avatarXhr) avatarXhr.abort();
    if (avatarImage) delete(avatarImage);
    if (avatarTimeout) {
      clearTimeout(avatarTimeout);
      avatarTimeout = null;
    }
    console.log("Posting: "+key);
    avatarXhr = $.ajax({
      type: 'post',
      url: '/upload/avatar',
      data:  {
        avatar: {key: key},
        authenticity_token: $("meta[name='csrf-token']").attr('content'),
        utf8: "✓"
      },
      success: function(response) {
        console.log(response);
        var $img = $('img.avatar')
        $img.each(function() {
          loadImage($(this), decodeURI(response.url));
        });
      }
    });
  }

  function loadImage($img, url, retryCount) {
    var missingUrl = '/images/avatars/original/missing.png';
    var importingUrl = '/images/loading.gif';
    if (!retryCount) {
      $img.removeClass('missing');
      $img.parent().addClass("importing");
      retryCount = 0;
    }
    avatarImage = new Image();
    avatarImage.src = url;
    avatarImage.onload = function() {
      if ($img) {
        $img.attr("src", url);
        $img.parent().removeClass('importing');
      }
    };
    avatarImage.onerror = function() {
      if (retryCount >= 10) {
        $img.parent().removeClass('importing');
        $img.attr("src", missingUrl);
        console.log('There was a problem loading the image');
        return false;
      }
      avatarTimeout = setTimeout(function() {
        loadImage($img, url, ++retryCount);
      }, 3000);
    };
  }

  $('.user-photo, .user-photo a').bind('click', function(e) {
    e.preventDefault();
    e.stopPropagation();
    $('#avatar_attachment').trigger("click");
    return false;
  });

  $('#avatar_attachment').uploader({
    start: function() {
      var uploader = this;
    },
    progress: function(event) {
      console.log("Progress!", event)
    },
    success: function(response) {
      console.log(response);
      var $response = $(response);
      var key = $(response).find("Key").text();
      postAvatar(key);
    },
    error: function(response) {
      console.log("Error!", response);
    }
  });
});

```

### Reliability of storage
### Post processing
### Importing
### Progress in the browser

## Resources

* [Uploading multiple files with the Nginx Upload Module](http://blog.joshsoftware.com/2010/10/20/uploading-multiple-files-with-nginx-upload-module-and-upload-progress-bar/)
* [Nginx Upload Module With Paerclip](http://matthewhutchinson.net/2010/1/6/nginx-upload-module-with-paperclip-on-rails/page/2)
* [Direct to S3 Browser Uploads](http://www.spacevatican.org/2013/7/7/direct-to-s3-browser-uploads/)
* http://blog.vrplumber.com/b/2013/06/07/nginx-upload-module/
* 