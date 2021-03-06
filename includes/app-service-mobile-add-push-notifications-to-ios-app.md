
**Objective-C**:

1. In **QSAppDelegate.m**, import the iOS SDK and **QSTodoService.h**:
        
        #import <MicrosoftAzureMobile/MicrosoftAzureMobile.h>
        #import "QSTodoService.h"

2. In `didFinishLaunchingWithOptions` in **QSAppDelegate.m**, insert the following lines right before `return YES;`:

        UIUserNotificationSettings* notificationSettings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound categories:nil];
        [[UIApplication sharedApplication] registerUserNotificationSettings:notificationSettings];
        [[UIApplication sharedApplication] registerForRemoteNotifications];

3. In **QSAppDelegate.m**, add the following handler methods. Your app is now updated to support push notifications. 

        // Registration with APNs is successful
        - (void)application:(UIApplication *)application
        didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        
            QSTodoService *todoService = [QSTodoService defaultService];
            MSClient *client = todoService.client;
        
            [client.push registerDeviceToken:deviceToken completion:^(NSError *error) {
                if (error != nil) {
                    NSLog(@"Error registering for notifications: %@", error);
                }
            }];
        }
        
        // Handle any failure to register
        - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:
        (NSError *)error {
            NSLog(@"Failed to register for remote notifications: %@", error);
        }
        
        // Use userInfo in the payload to display an alert.
        - (void)application:(UIApplication *)application
              didReceiveRemoteNotification:(NSDictionary *)userInfo {
            NSLog(@"%@", userInfo);
        
            NSDictionary *apsPayload = userInfo[@"aps"];
            NSString *alertString = apsPayload[@"alert"];
        
            // Create alert with notification content.
            UIAlertController *alertController = [UIAlertController
                                          alertControllerWithTitle:@"Notification"
                                          message:alertString
                                          preferredStyle:UIAlertControllerStyleAlert];
        
            UIAlertAction *cancelAction = [UIAlertAction
                                           actionWithTitle:NSLocalizedString(@"Cancel", @"Cancel")
                                           style:UIAlertActionStyleCancel
                                           handler:^(UIAlertAction *action)
                                           {
                                               NSLog(@"Cancel");
                                           }];
            
            UIAlertAction *okAction = [UIAlertAction
                                       actionWithTitle:NSLocalizedString(@"OK", @"OK")
                                       style:UIAlertActionStyleDefault
                                       handler:^(UIAlertAction *action)
                                       {
                                           NSLog(@"OK");
                                       }];
            
            [alertController addAction:cancelAction];
            [alertController addAction:okAction];
            
            // Get current view controller.
            UIViewController *currentViewController = [[[[UIApplication sharedApplication] delegate] window] rootViewController];
            while (currentViewController.presentedViewController)
            {
                currentViewController = currentViewController.presentedViewController;
            }
            
            // Display alert.
            [currentViewController presentViewController:alertController animated:YES completion:nil];
        
        }

**Swift**:

1. Add file **ClientManager.swift** with the following contents. Replace _%AppUrl%_ with the URL of the Azure Mobile App backend.
        
        class ClientManager {
            static let sharedClient = MSClient(applicationURLString: "%AppUrl%")
        }

2. In **ToDoTableViewController.swift**, replace the `let client` line that initializes an `MSClient` with this line:

        let client = ClientManager.sharedClient
 
3. In **AppDelegate.swift**, replace the body of `func application` as follows:

        func application(application: UIApplication,
           didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) -> Bool {
           application.registerUserNotificationSettings(
               UIUserNotificationSettings(forTypes: [.Alert, .Badge, .Sound],
                   categories: nil))
           application.registerForRemoteNotifications()
           return true
        }

2. In **AppDelegate.swift**, add the following handler methods. Your app is now updated to support push notifications.
        

        func application(application: UIApplication,
        didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
            ClientManager.sharedClient.push?.registerDeviceToken(deviceToken, completion: { (error) -> Void in
                NSLog("Error registering for notifications: %@", error!.description)
            })
        }

            
        func application(application: UIApplication,
        didFailToRegisterForRemoteNotificationsWithError error: NSError) {
            NSLog("Failed to register for remote notifications: \n%@", error.description)
        }

        func application(application: UIApplication,
        didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
            
            NSLog("%@", userInfo)
            
            let apsNotification = userInfo["aps"] as! NSDictionary
            let apsString       = apsNotification["alert"] as! String
            
            
            let alert = UIAlertController(title: "Alert", message:apsString, preferredStyle: .Alert)
            let okAction = UIAlertAction(title: "OK", style: .Default) { _ in
                NSLog("OK")
            }
            let cancelAction = UIAlertAction(title: "Cancel", style: .Default) { _ in
                NSLog("Cancel")
            }
            
            alert.addAction(okAction)
            alert.addAction(cancelAction)
            
            var currentViewController = UIApplication.sharedApplication().delegate?.window??.rootViewController
            while currentViewController?.presentedViewController != nil {
                currentViewController = currentViewController?.presentedViewController
            }
            
            currentViewController?.presentViewController(alert, animated: true){}
            
        }
    
