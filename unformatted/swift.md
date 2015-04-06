http://www.objc.io/books/
http://nshipster.com
http://robots.thoughtbot.com/efficient-json-in-swift-with-functional-concepts-and-generics








Swift:

http://matthewpalmer.net/blog/2014/06/21/example-ios-keychain-swift-save-query/

http://cdn1.raywenderlich.com/wp-content/uploads/2014/06/RW-Swift-Cheatsheet-0_5.pdf

http://stackoverflow.com/questions/24113313/adding-item-to-keychain-using-swift
http://rshelby.com/2014/08/using-swift-to-save-and-query-ios-keychain-in-xcode-beta-4/
https://github.com/jverdi/UICKeyChainStore/blob/master/Lib/UICKeyChainStore.m

https://medium.com/@aommiez/afnetwork-integrate-swfit-80514b545b40

```
    func signup() {
        // POST signup
        let manager = AFHTTPRequestOperationManager()
        manager.requestSerializer.setValue(self.application, forHTTPHeaderField: "X-Application-Id")
        var parameters = [
            "signup": [
                "full_name": "Test User",
                "email": "test2@example.com",
                "password": "password",
                "password_confirmation": "password_confirmation",
                "birthdate": NSDate(),
                "weight_in_pounds": 150
            ]
        ]
        var success = { (operation: AFHTTPRequestOperation!, responseObject: AnyObject!) -> Void in
            println("JSON: " + responseObject.description)
            let responseDict = responseObject as Dictionary<String, AnyObject>
            let token : String? = (responseDict["token"] as AnyObject?) as? String
            let refreshToken : String? = (responseDict["refresh_token"] as AnyObject?) as? String
            let derp : String? = (responseDict["derp"] as AnyObject?) as? String

            println("Token: \(token)")
            println("Refresh Token: \(refreshToken)")
            println("Derp: \(derp)")
        }
        var failure = { (operation: AFHTTPRequestOperation!,error: NSError!) -> Void in
            println("Error: " + error.localizedDescription)
        }
        manager.POST("\(self.url())/signup", parameters: parameters, success: success, failure: failure)
    }
```