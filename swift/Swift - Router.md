# Swift - Router


## Router

A simple router:

```swift
//
//  Router.swift
//  TextMeWhen
//
//  Created by Jeffrey Rafter on 8/10/16.
//  Copyright © 2016 TextMeWhen. All rights reserved.
//

import Foundation
import Alamofire

typealias JSON = [String:AnyObject]

enum ResultType<T> {
    case Success(T)
    case Error(String?)
}

enum Router: URLRequestConvertible {

    case SendPhoneConfirmation(phoneNumber: String)

    private static let host: String = "http://localhost:9292"
    
    private var method: Alamofire.Method {
        switch self {
        case .SendPhoneConfirmation:
            return .POST
        }
    }
    
    private var path: String {
        switch self {
        case .SendPhoneConfirmation:
            return "/v1/confirm"
        }
    }
    
    var URLRequest: NSMutableURLRequest {
        let url = NSURL(string: "\(Router.host)")!
        let mutableURLRequest = NSMutableURLRequest(URL: url.URLByAppendingPathComponent(path))
        mutableURLRequest.HTTPMethod = method.rawValue
        
        switch self {
        case .SendPhoneConfirmation(let phoneNumber):
            let parameters = [ "phone_number" : phoneNumber ]
            return Alamofire.ParameterEncoding.JSON.encode(mutableURLRequest, parameters: parameters).0
        }
    }
}
```



 A more complex, namespaced version:
 
```swift
//
//  Router.swift
//  TextMeWhen
//
//  Created by Jeffrey Rafter on 8/10/16.
//  Copyright © 2016 TextMeWhen. All rights reserved.
//

import Foundation
import Alamofire

typealias JSON = [String:AnyObject]

enum ResultType<T> {
    case Success(T)
    case Error(String?)
}

struct Router {}

extension Router {
    enum Routes: URLRequestConvertible {
        case SendPhoneConfirmation(phoneNumber: String)
    }
}

extension Router.Routes {

    private static let host: String = "http://localhost:9292"
    
    private var method: Alamofire.Method {
        switch self {
        case .SendPhoneConfirmation:
            return .POST
        }
    }
    
    private var path: String {
        switch self {
        case .SendPhoneConfirmation:
            return "/v1/confirm"
        }
    }
    
    var URLRequest: NSMutableURLRequest {
        let url = NSURL(string: "\(Router.Routes.host)")!
        let mutableURLRequest = NSMutableURLRequest(URL: url.URLByAppendingPathComponent(path))
        mutableURLRequest.HTTPMethod = method.rawValue
        
        switch self {
        case .SendPhoneConfirmation(let phoneNumber):
            let parameters = [ "phone_number" : phoneNumber ]
            return Alamofire.ParameterEncoding.JSON.encode(mutableURLRequest, parameters: parameters).0
        }
    }
}

```

## Auth

Create a new Swift file `Clients/Auth/Auth.swift`

import Foundation
import Alamofire
import Intercom

typealias AccessToken = String

enum UserDefaultsKey {
    static let token = "user.token"
}

enum AuthType {
    case email(String,  String)
    case facebook
    case google
}

enum OAuthProvider: String {
    case facebook
    case google
}

// Auth is a wrapper around the networking layer trying to normalize the API
// for different login options. All calls to login will result in either an 
// error or an access token. The **only** state associated with Auth is
// a Teespring access token stored in NSUserDefaults when login is successful.
// This token is removed on logout
struct Auth {
    
    static var token: String? {
        get { return NSUserDefaults.standardUserDefaults().valueForKey(UserDefaultsKey.token) as? String }
        set { NSUserDefaults.standardUserDefaults().setValue(newValue, forKey: UserDefaultsKey.token) }
    }
    
    static func login(type: AuthType, completion: (ResultType<AccessToken>) -> Void) {
        switch type {
        case .email(let email, let password):
            Analytics.trackEvent("email.login.initiated")
            Auth.Email.login(email, password: password, completion: completion)
        case .facebook:
            Analytics.trackEvent("facebook.login.initiated")
            Auth.Facebook.login(completion)
        case.google:
            Analytics.trackEvent("google.login.initiated")
            Auth.Google.login(completion)
        }
    }
    
    static func logout() {
        Auth.token = nil
    }
    
    static func fetchAuthedUser(completion: ResultType<(User)> -> Void) {
        Alamofire.request(Teespring.Router.GetMe).validate().responseJSON { response in
            
            guard let result = response.result.value as? JSON else {
                return
            }
            
            guard response.result.isSuccess && result["error"] as? String == nil else {
                let errorMessage = (result["error"] as? String) ?? "Server error"
                completion(.Error(errorMessage))
                return
            }
            
            guard let users = result["users"] as? [[String : AnyObject]],
                id = users.first?["id"] as? Int,
                email = users.first?["email"] as? String else {
                    completion(.Error("Server error"))
                    return
            }
            
            User.currentUser.email = email
            User.currentUser.id = id

            completion(.Success(User.currentUser))
        }
    }
    
}


## That Phone extension

```swift
import Foundation
import Alamofire

extension Auth {
    struct Phone {
        static func login(phoneNumber: String, completion: (ResultType<ConfirmationToken>) -> Void) {
            Alamofire.request(Router.SendPhoneConfirmation(phoneNumber: phoneNumber)).validate().responseJSON { response in
                let result = Phone.handleResponse(response)
                
                switch result {
                case .Success(let token):
                    Auth.token = token
                    
                    print("Phone login succeeded")
                case .Error(let message):
                    print("Phone login failed: \(message)")
                }
                completion(result)
            }
        }
        
        static func handleResponse(response: Response<AnyObject, NSError>) -> ResultType<ConfirmationToken> {
            guard let result = response.result.value as? JSON else {
                return .Error("Server error")
            }
        
            guard response.result.isSuccess && result["error"] as? String == nil else {
                let errorMessage = (result["error"] as? String) ?? "Server error"
                return .Error(errorMessage)
            }
        
            guard let token = result["token"] as? String else {
                return .Error("Invalid access token")
            }
        
            return .Success(token)
        }
    }
}
```


## Struct with func

```swift
import Foundation
import Alamofire

extension Auth {
    struct Email {
        static func login(email: String, password: String, completion: (ResultType<AccessToken>) -> Void) {
            Alamofire.request(Teespring.Router.GetAccessToken(email: email, password: password)).validate().responseJSON { response in
                let result = AuthResponse.parseTokenResponse(response)
                switch result {
                case .Success:
                    Analytics.trackEvent("email.login.succeeded")
                case .Error:
                    Analytics.trackEvent("email.login.failed")
                }
                completion(result)
            }
        }
    }
}
```

## Caling it

Call the method:

```swift
@IBAction func loginWithEmail(sender: TSButton) {
    guard let email = emailField.text, password = passwordField.text else { return }
    showActivityIndicator()
    Auth.login(.email(email, password), completion: handleLogin)
}
```

Handle the result using a generic `ResultType` functionally:

```swift
    private func handleLogin(loginResult: ResultType<AccessToken>) {
        hideActivityIndicator()
        switch loginResult {
        case .Success:
            self.view.endEditing(true)
            self.delegate?.loginViewControllerUserLoginSucceeded(self)
        case .Error(let message):
            guard let message = message else { return }
            
            let action = UIAlertAction(title: "Dismiss", style: .Default, handler: nil)
            let alert = UIAlertController(title: message, message: nil, preferredStyle: .Alert)
            alert.addAction(action)
            
            self.presentViewController(alert, animated: true, completion: nil)
        }
    }
```

## App transport security exception

In Info.plist

![Add the XML](https://rpl.cat/uploads/5oofhd2ZPPW2CP7yHCZHJk_eqI4uadp0TNzJOwhFG2Y/public.png)


```xml
<key>NSAppTransportSecurity</key>
<dict>
   <key>NSExceptionDomains</key>
   <dict>
       <key>localhost</key>
       <dict>
        <key>NSTemporaryExceptionAllowsInsecureHTTPSLoads</key>
           <false/>           
           <key>NSIncludesSubdomains</key>
           <true/>
           <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
           <true/>
           <key>NSTemporaryExceptionMinimumTLSVersion</key>
           <string>1.0</string>
           <key>NSTemporaryExceptionRequiresForwardSecrecy</key>
           <false/>
       </dict>
   </dict>
</dict>
```
