# Swift - Getting Started

Open XCode and start a new project. When starting choose a *Single View Application*:

![Start a new project](https://rpl.cat/uploads/POd2YaOKDuqwh6wpO0-5OxvFoBujmtGWxiL01rLGUnM/public.png)

Choose an appropriate (and unique) bundle identifier and application name. For example:

![Choose your project options](https://rpl.cat/uploads/EEI83eJzqzF8l2NUIn8cs2PDEVlC2YJWgOJ7iPIF28I/public.png)

Once your options are set, choose a folder for the project to live. XCode will automatically setup a git repository and make your first commit.

## Setup a .gitignore

```
# Created by http://www.gitignore.io

### OSX ###
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon


# Thumbnails
._*

# Files that might appear on external disk
.Spotlight-V100
.Trashes

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk


### Swift ###
# Xcode
#
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata
*.xccheckout
*.moved-aside
DerivedData
*.hmap
*.ipa
*.xcuserstate

# CocoaPods
#
# We recommend against adding the Pods directory to your .gitignore. However
# you should judge for yourself, the pros and cons are mentioned at:
# http://guides.cocoapods.org/using/using-cocoapods.html#should-i-ignore-the-pods-directory-in-source-control
#
# Pods/

# You may only want to exclude Carthage/Checkouts if you are on Circle
Carthage/*

### Xcode ###
build/
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata
*.xccheckout
*.moved-aside
DerivedData
*.xcuserstate
```

## RootViewController

Having a single root view controller makes controlling the direction of specific behavior easier. Create a new class called `RootViewController` which is a `UIViewController`. This should be a singleton.

To make it a singleton add the following (within your `RootViewController`):

```swift
static let sharedInstance = RootViewController()
```

Within your `AppDelegate` add the following:

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    window = UIWindow(frame: UIScreen.mainScreen().bounds)
    window!.backgroundColor = .whiteColor()
    window!.rootViewController = RootViewController.sharedInstance
    window!.makeKeyAndVisible()
    RootViewController.sharedInstance.showViewControllerNeededForState()
    
    return true
}

```

Notice that this delegates the flow control to `showViewControllerNeededForState`. This can be as simple as:

```swift
func showViewControllerForState() {
    if (state == 1) {
        showConfirm()
    } else {
        showMain()
    }
}
```

Here the `state` is a basic variable. But this might be a more complex check to see if a user has logged in or has completed an optional setup. 

## Storyboards for all of your controllers

In general, large storyboards can become unweildy and slow. Splitting them up helps. Create a new user interface file and call it "Confirm":

![New user interface file](https://rpl.cat/uploads/DYrmcUouznl6gOZD65IdC2N7slEkkkA7cbk-klR4g0g/public.png)

Also create a new view controller:

![New view controller](https://rpl.cat/uploads/HXbeX0Rj4wvPS4R4jXROXw3p_mTFBfRljvQI9199jrU/public.png)

![Name the view controller](https://rpl.cat/uploads/DIIG_EU2r_vzuuOWyKr0YtRMIcUMMDPAdGlevJVG9Go/public.png)

Open the new storyboard and click on the view controller. Then in the Utilities panel set the identifier and class type:

![Set the identifier and type](https://rpl.cat/uploads/xpL9OC8JFsLvKJnMooi8x0M8LLE1FMHxK43Qbx4dFIc/public.png)

Having the storyboard connected to your view controller makes things easy. But it would be even easier if you could initialize the storyboard directly. Add the following to your `ConfirmViewController`:

```swift
static let storyboardIdentifier = "ConfirmViewControllerStoryboard"

static func initFromStoryboard() -> ConfirmViewController? {
    let storyboard = UIStoryboard(name: "Confirm", bundle: nil)

    return storyboard.instantiateViewControllerWithIdentifier(storyboardIdentifier) as? ConfirmViewController
}
```

Now, return to your `RootViewController` and add the function that shows the appropriate storyboard:

```swift
func showConfirm() {
    let confirmViewController = ConfirmViewController.initFromStoryboard()!
    let confirmView = confirmViewController.view
    addChildViewController(confirmViewController)
    view.addSubview(confirmView)
    confirmView.frame = view.frame;
    confirmViewController.didMoveToParentViewController(self)
}
```

You should be able to do the same for the `Main.storyboard` and the ViewController associated with it (which you can rename).

## Adding a delegate to your controller

Because the `RootViewController` is a singleton with a `sharedInstance` you can call methods directly on it. However, in some cases you may want to use the delegate pattern instead. 

In your `ConfirmViewController` define a new protocol:

```swift
protocol ConfirmViewControllerDelegate: class {
    func confirmViewControllerSucceeded(viewController: ConfirmViewController)
}
```

Then based on an action you can call it:

```swift
@IBAction func confirmTouchUpInside(sender: AnyObject) {
    self.delegate?.confirmViewControllerSucceeded(self)
}
```

Your `RootViewController` will need to implement the protocol:

```swift
class RootViewController: UIViewController, ConfirmViewControllerDelegate {
    
    var state = 1
    
    // ... more code here
    
    func confirmViewControllerSucceeded(viewController: ConfirmViewController) {
      print("Success");
      state = 2
      showViewControllerForState()
    }
}
```

## Carthage

Carthage manages dependencies.

```bash
brew install carthage
```

Create a `Cartfile` in that folder that contains your "*.xcodeproj" that looks like:

```
github "Alamofire/Alamofire" == 2.0
```

Install:

```bash
carthage update --platform iOS
```

Once you've built the dependencies you'll want to add them into your application's **Embedded Binaries** by dragging them into the correct location:

![XCode](https://rpl.cat/uploads/JkjAx6M6oqLmqQLlkPAKXo4t7wOW3bWCQHUxu8qme2Q/public.png)

![Copy items](https://rpl.cat/uploads/9zlNE9I0nF5qWaK_L-HBrYAHO4E5ZL7X5S8gmFauXRo/public.png)


> See also https://www.raywenderlich.com/109330/carthage-tutorial-getting-started

## Reachability

In the AppDelegate:

```swift
 // MARK: Reachability

    private func initializeReachability() {
        do {
            reachability = try Reachability.reachabilityForInternetConnection()
        } catch {
            print("Unable to create Reachability")
            return
        }

        reachability?.whenReachable = whenReachable
        reachability?.whenUnreachable = whenUnreachable

        do {
            try reachability?.startNotifier()
        } catch {
            print("Unable to start notifier")
        }
    }

    private func whenReachable(reachability: Reachability) {
        dispatch_async(dispatch_get_main_queue()) {
            RootViewController.sharedInstance.showLoginOrAppAsNeeded()
        }
    }

    private func whenUnreachable(reachability: Reachability) {
        dispatch_async(dispatch_get_main_queue()) {
            RootViewController.sharedInstance.showNetworkUnavailable()
        }
    }
```    

## Registering for notifications

In the AppDelegate:

```swift
    // MARK: Register notifications

    static func registerNotifications() {
        registerUINotifications()
        registerAllRemoteNotifications()
    }

    private static func registerUINotifications () {
        let types: UIUserNotificationType = [.Badge, .Alert, .Sound]
        let settings = UIUserNotificationSettings(forTypes: types, categories: nil)
        UIApplication.sharedApplication().registerUserNotificationSettings(settings)
    }

    private static func registerAllRemoteNotifications () {
        UIApplication.sharedApplication().registerForRemoteNotifications()
    }
```

## Managing child view controllers

```
func showViewController(newVC: UIViewController, animated: Bool) {        
        let newView = newVC.view
        addChildViewController(newVC)
        view.addSubview(newVC.view)
        // Set the frame of the new view, this is using Snapkit
        newView.snp_remakeConstraints { $0.edges.equalToSuperview() }
        newVC.didMoveToParentViewController(self)

    }
```    
