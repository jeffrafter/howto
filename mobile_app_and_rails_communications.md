# Mobile app and Rails Communications

If there is only one client (not multiple users of an API) then all you need is the ability to sign messages. If there is more than one user of the API you will need to maintain all of the signing secrets and an app id to look them up. 

These values are stored directly in the code of the iOS application, For example, connecting to the parse API looks like this:


    [Parse setApplicationId:@"Mp39HeDYI8b2rMp39HeDYI8b2Mp39HeDYI8b2"
                  clientKey:@"y27vlU54XO4DTTlRf7ARetbyC8AYaSA4KD8CXQEQ"];
              

This is placed in the `AppDelegate` method `didFinishLaunchingWithOptions`.

The basic flow is:

* Login/Signup request is made
  * The request is signed, and the signature is verified
  * Login credentials are checked
  * A user token is created and returned
  * User token is stored in Keychain (not in NSUserDefaults)

Logged in users can communicate in two ways:  

* Subsequent logged in requests are made
  * The requests are signed, but in the payload the user token is added
  * The requests are signed using the user token and the user id or email is passed. This allows a user lookup followed by a secure compare.




# On the Rails side

You need a controller that acts as your primary interface point:

    # Note that this is not descended from ApplicationController
    class ApiController < ActionController::Base
    end
    
You'll need to setup a secret

* secret   
    
Things you need:

* Signatures
* Verify Signed Requests
* Verify Tokens
* User id token?
* token_user
* device_token
* current_device
* current_user
* user_required
* token_user_required


# On the iOS Side

ConnectionManager

* Makes request
* Calls delegates
* Indicates network traffic via delegate

RawRequest

* Has a NSMutableURLRequest
* Has an NSUrlConnection
* When the request is created the params are decorated with additional info:
  * User agent
  * Device Id (from the MAC address not the UDID)
* When the request is started the request manager is notified
* Handles failing/completing/aborting

RequestHandler

* Has the ability to append user based token if there is one



APIConnectionManager
APIRequest
APIHandler
APISession
APIError


# RESOURCES

http://keighl.com/post/secure-api-request-from-ios-to-rails/