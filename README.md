# Login

This sample project shows how to present a login dialog using the Lock widget interface. Once you log in, you're taken to a very basic profile screen, with some data about your user.

##### Required Dependencies

```swift
  pod 'Lock', '~> 1.28'
  pod 'Lock/Safari'
```

##### Perform Social Authentication with SafariAuthenticator

First, go to your Client Dashboard and make sure that Allowed Callback URLs contains the following:

```
{YOUR_APP_BUNDLE_IDENTIFIER}:///ios/{YOUR_APP_BUNDLE_IDENTIFIER}/callback
``` 
In your application's Info.plist file, register your iOS Bundle Identifier as a custom scheme. To do so, open the Info.plist as source code, and add this chunk of code under the main <dict> entry:

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>None</string>
        <key>CFBundleURLName</key>
        <string>auth0</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>{YOUR_APP_BUNDLE_IDENTIFIER}</string>
        </array>
    </dict>
</array>
```

#### Important Snippets

##### 1. Initialise Safari Authenticator in AppDelegate
```swift
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
      // Override point for customization after application launch.
      A0LockLogger.logAll()
      let lock = A0Lock.shared()
      let safari = A0SafariAuthenticator(lock: lock, connectionName: "google-oauth2", useUniversalLink: false)

      lock.registerAuthenticators([safari])
      lock.applicationLaunched(options: launchOptions)
      return true
  }    
  func application(_ application: UIApplication, open url: URL, sourceApplication: String?, annotation: Any) -> Bool {
      return A0Lock.shared().handle(url as URL, sourceApplication: sourceApplication)
  }
  func application(_ application: UIApplication,
                            continue userActivity: NSUserActivity,
                            restorationHandler: @escaping ([Any]?) -> Void) -> Bool
  {
      return A0Lock.shared().continue(userActivity, restorationHandler: restorationHandler)
  }
```

##### 2. Present the login widget

In `HomeViewController.swift`:

```swift
@IBAction func showLoginController(sender: UIButton) {
    let controller = A0Lock.sharedLock().newLockViewController()
    controller.closable = true
    controller.onAuthenticationBlock = { profile, token in
        guard let userProfile = profile else {
            self.showMissingProfileAlert()
            return
        }
        self.retrievedProfile = userProfile
        controller.dismissViewControllerAnimated(true, completion: nil)
        self.performSegueWithIdentifier("ShowProfile", sender: nil)
    }
    A0Lock.sharedLock().presentLockController(controller, fromController: self)
}
```

##### 3. Pass the profile object

In this sample, the `profile` object is passed to the next screen through a variable, in the `prepareForSegue` method, as follows:

    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        guard let profileController = segue.destinationViewController as? ProfileViewController else {
            return
        }
        profileController.profile = self.retrievedProfile
    }
This is a very simple approach to do so. You'll find later on, in further sample projects, that this strategy is replaced by using a `SessionManager` class, which avoids updating inconsistencies among view controllers.

##### 4. Show basic profile data

In `ProfileViewController.swift`:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    self.welcomeLabel.text = "Welcome, \(self.profile.name)"
    NSURLSession.sharedSession().dataTaskWithURL(self.profile.picture, completionHandler: { data, response, error in
        dispatch_async(dispatch_get_main_queue()) {
            guard let data = data where error == nil else { return }
            self.avatarImageView.image = UIImage(data: data)
        }
    }).resume()
}
```
