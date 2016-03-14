## Ionic integrate Pushwoosh Example
This project build a demo for integrating Pushwoosh SDK into a ionic project. The main part is pushwoosh service. It include:

  * How to install pushwoosh plugin
  * Initialize the plugin
  * Registering device for push notifications
  * Listen receiving push notifications
  * Handle notification

#### Note:
  * IOS Simulator can neither subscribe nor receive push notifications.
  * IOS and Android data structure is different

### Install

  * Install pushwoosh plugin

  `ionic plugin add pushwoosh-cordova-plugin@4.1.0`

  * Whitelist *.pushwoosh.com domain in the `config.xml` file

  `<access origin="*.pushwoosh.com" />`

### Code Example

  1). Initialize plugin

```
  // initialize the plugin
  function initializePlugin(){
    console.log("init pushwoosh plugin...");
    if (ionic.Platform.isIOS()) {
      pushNotification.onDeviceReady({ pw_appid: PW_APP_ID});
    }

    if (ionic.Platform.isAndroid()) {
      pushNotification.onDeviceReady({ projectid: GOOGLE_PROJECT_NUMBER, appid: PW_APP_ID });
    }
  }
```

  2). Registering Device for push notifications

```
// register device in pushwoosh server
function registerDevice(){
  console.log("register device...")
  pushNotification.registerDevice(function(status) {
    var deviceToken = ionic.Platform.isIOS() ? status.deviceToken : status;
    console.log('registerDevice: ' + deviceToken);
  }, function(status) {
    var errorStr = JSON.stringify(status)
    console.warn('failed to register : ' + errorStr);
    // if the request is timeout then retry
    if(errorStr.search("The request timed out") != -1){
      registerDevice();
    }
  });
}

  ```

  3). Listen receiving push notifications

```
  // set push notification callback before we initialize the plugin
  document.addEventListener('push-notification', function(event) {
    //get the notification payload
    var notification = event.notification;
    //handle notification
    notificationHandler(notification);
    //clear the app badge
    resetApplicationIconBadgeNumber();
  });
```

  4). Handle notification

```
  function notificationHandler(notification){
    var formatNotification = formatPushData(notification);
    var messageType = formatNotification.type;

    // onStart is true means the app is not active
    // the user click in notification center
    if(notification.onStart){
      // app wake up from notification center,
      // do something what you like, for example:
      // $state.go("home");
    } else {
      // do something what you need
    }
  }

  function formatPushData(notification){
    /********************* ios push data structure ************************************
    event = {
      ...
      notification: {
        onStart: true,
        aps: {
          alert: "Nice to meet you",
          badge: 1,
          sound: "default"
        },
        custom: {
          type: "message"
        },
      }
    }
    ******************************************************************************/

    /**************** android push data structure*****************************************
    event = {
      notification: {
        alert: "[xx] match with you",
        collapse_key: "do_not_collapse",
        custom: "{"type":"dop"}",
        foreground: false,
      }
    }
    ******************************************************************************/
    var notification = ionic.Platform.isIOS() ? notification.userdata : JSON.parse(notification.userdata);
    return notification;
  }
```

### refs
* http://docs.pushwoosh.com/docs/cordova-phonegap
* https://rawgit.com/Pushwoosh/pushwoosh-phonegap-3.0-plugin/master/Documentation/files/PushNotification-js.html
