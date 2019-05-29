# react-native-zendesk-zopim-chat

Simple module that allows displaying Zopim Chat from Zendesk for React Native With Support for push notifications. Module works for both Android and iOS

`department` is set to `ZDCPreChatDataRequiredEditable`.

## Known issues

I could not find how to make the import for iOS work properly since I'm using Cocoapods for Zendesk, if you have a suggestion that would be great.

## Getting started

### Further Reading (Important)

Follow the instructions to install the SDK for 
[iOS](https://developer.zendesk.com/embeddables/docs/ios-chat-sdk/introduction) 
and 
[Android](https://developer.zendesk.com/embeddables/docs/android-chat-sdk/introduction).

## Pre-Reqisites

To use the push notifications features you must first install firebase. I recommend React Native Firebase. [](https://rnfirebase.io)
[iOS](https://rnfirebase.io/docs/v5.x.x/installation/ios) 
and 
[Android](https://rnfirebase.io/docs/v5.x.x/installation/android)

Then go through the installation for enabling notifications with the rnfirebase package 
[iOS](https://rnfirebase.io/docs/v5.x.x/messaging/ios) 
and 
[Android](https://rnfirebase.io/docs/v5.x.x/messaging/android)

### Installation

Run in terminal
```groovy
    npm i react-native-zendesk-zopim-chat
```

### Automatic Integration

### Using React Native Link

Run 
```groovy
react-native link react-native-zendesk-zopim-chat
```
 after which you should be able to use this library on iOS and Android directly


### Manual Integration

#### iOS
1. `npm install react-native-zendesk-zopim-chat --save`
2. In Xcode, drag and drop `node_modules/react-native-zendesk-zopim-chat/RNZendeskChat.m` and `node_modules/react-native-zendesk-zopim-chat/RNZendeskChat.h` into your project.
3. Configure `ZDCChat` in `AppDelegate.m`:

```
#import <ZDCChat/ZDCChat.h>

[ZDCChat initializeWithAccountKey:@"YOUR_ZENDESK_ACCOUNT_KEY"];
```

#### Android

1. Define the `react-native-zendesk-zopim-chat` project in `android/settings.gradle`:

```groovy
...
include ':react-native-zendesk-zopim-chat'
project(':react-native-zendesk-zopim-chat').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-zendesk-zopim-chat/android')
```

2. Add the `react-native-zendesk-zopim-chat` as an dependency of your app in `android/app/build.gradle`:

```groovy
...
dependencies {
  ...
  implementation project(':react-native-firebase')
  implementation project(':react-native-zendesk-zopim-chat') // Add this line
}
```

If you've defined *[project-wide
properties](https://developer.android.com/studio/build/gradle-tips.html)*
(**recommended**) in your root `build.gradle`, this library will detect
the presence of the following properties: (Project level gradle file) in ` android/build.gradle `
```groovy
buildscript {
  ...
     repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.3.2'
        classpath 'com.google.gms:google-services:4.2.0'
        classpath 'com.google.firebase:firebase-plugins:1.1.5'
    }

    /**
    + Project-wide Gradle configuration properties 
    */
    ext {
        compileSdkVersion   = 28
        targetSdkVersion    = 28
        buildToolsVersion   = "28.0.3"
        supportLibVersion   = "28.0.0"
        googlePlayServicesVersion = "16.1.0" // or set latest version HERE
    }
    ...
}
allprojects {
  ...
    repositories {
        mavenLocal()
        jcenter()
        google()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        // Crashlytics repository
        maven { url 'https://maven.google.com'}
        maven { url 'https://zendesk.jfrog.io/zendesk/repo' }
    }
  ...
}
 
```

If you do **not** have *project-wide properties* defined and have a
different play-services version than the one included in this library,
use the following instead (switch 16.0.1 for the desired version):

```groovy
...
dependencies {
  ...
    // Google and Firebase Dependencies
    implementation project(':react-native-firebase')
    implementation "com.google.android.gms:play-services-base:16.0.1"
    implementation 'com.google.firebase:firebase-core:16.0.8'
    implementation 'com.google.firebase:firebase-messaging:17.5.0'

    implementation (project(':react-native-zendesk-zopim-chat')){
      exclude group: 'com.android.support'
      exclude module: 'appcompat-v7'
      exclude module: 'support-v4'
    }
    ...
}
```
3. Open up `android/app/main/java/[...]/MainApplication.java`
  - Add `import com.zaidiapps.zopimchat.*;` to the imports at the top of the file
  - Add `new RNZopimChatPackage()` to the list returned by the `getPackages()` method :

```java
// Firebase packages
import io.invertase.firebase.RNFirebasePackage;
import io.invertase.firebase.messaging.RNFirebaseMessagingPackage;
import io.invertase.firebase.notifications.RNFirebaseNotificationsPackage; 
// Zopim Chat package
import com.zaidiapps.zopimchat.zendesk.RNZopimChatPackage;
...
    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new RNZopimChatPackage()
        );
    }
```

4. Configure `ZopimChat` in `android/app/main/java/[...]/MainApplication.java`

```java
...
  static RNZendeskChatModule rnZendeskChatModule;

  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, /* native exopackage */ false);
    // Init the chat module from taskrabbit
    ReactApplicationContext reactContext = new ReactApplicationContext(this);
    rnZendeskChatModule = new RNZendeskChatModule(reactContext);
    rnZendeskChatModule.init("YOUR_ZENDESK_ACCOUNT_KEY");
  }
  ...
```

## Usage

### In your code 

#### App Container 
```javascript
import React from 'react'
import { AsyncStorage } from 'react-native'
import ZendeskChat from 'react-native-zendesk-chat'
import firebase from 'react-native-firebase'
import Router from './Router'

class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      fcm_token: '',
      alert: {
        title: '', message: '', confirmText: '', showLoading: false,
      },
    }
  }

  render() {
    const { alert } = this.state
    return (
      <Router screenProps={
          { alert }
        }
      />
    )
  }


  _showLocalNotification = (notif) => {
    let notification = new firebase.notifications.Notification()
    notification = notification.setNotificationId(new Date().valueOf().toString())
      .setNotificationId(notif.notificationId)
      .setTitle(notif.title)
      .setSubtitle(notif.subtitle)
      .setBody(notif.body)
      .setData(notif.data)
      .setSound('notif_sound.mp3') // Found in android/app/main/res/raw/

    notification.ios.badge = 10
    notification.android.setAutoCancel(true)
    notification.android.setColorized(true)
    notification.android.setOngoing(true)
    notification.android.setPriority(firebase.notifications.Android.Priority.High)
    notification.android.setSmallIcon('ic_stat_notif_icon')
    notification.android.setLargeIcon('ic_launcher')
    notification.android.setVibrate([500])
    notification.android.setChannelId('main_channel')

    firebase.notifications().displayNotification(notification)
  }

  // This method handles notifications based on the state the app is in upon receiving a notif i.e whether closed or open
  _dealWithNotification = (notif, source) => {
    let alert_title = ''
    let alert_message = ''
    let alert_button_text = ''
    let destination = ''
    let notif_type = ''
    let notif_data = {}
    if ('data' in notif) {
      const { data } = notif
      notif_data = data
      alert_title = 'alert_title' in data ? data.alert_title : 'Urgent Message'
      alert_message = 'alert_message' in data ? data.alert_message : 'Urgent Message'
      destination = 'destination' in data ? data.destination : null // The destination to navigate to (Zendesk component screen below)
      notif_type = 'notif_type' in data ? data.notif_type : null // Check if the type is zendesk or other as shown in the docs below

  ```
[Android](https://developer.zendesk.com/embeddables/docs/android-chat-sdk/push_notifications)
[iOS](https://developer.zendesk.com/embeddables/docs/ios-chat-sdk/push_notifications)
  
  ```javascript
      
    }

    if (source === 'app_open') { // Notification opened from app open foreground/background state
      if (notif_type === 'alert') {
        this.setState(alert)
      }
    } else if (source === 'app_closed') { // Notification opened from app open closed state.
      AsyncStorage.multiSet([ // Cache the notif to use it on the prefered screen if need be
        ['pending_notif', notif_type],
        ['notif_data', JSON.stringify(notif_data)],
      ])
    }
  }


  // The app listener handles notifications that came in while the app is in the foreground or in open state in the background.
  _registerAppListener() {
    this.notificationListener = firebase.notifications().onNotification((notification) => {
      this._showLocalNotification(notification)
    })
    this.notificationOpenedListener = firebase.notifications().onNotificationOpened((notificationOpen) => {
      const notif = notificationOpen.notification
      this._dealWithNotification(notif, 'app_open')
    });

    this.onTokenRefreshListener = firebase.messaging().onTokenRefresh((token) => {
      this.setState({ fcm_token: token }, () => {
        ZendeskChat.setPushToken(token)
      })
    });

    this.messageListener = firebase.messaging().onMessage((message) => {
      _showLocalNotification(message)
    });
  }


  componentDidMount = async () => {
    const channel = new firebase.notifications.Android.Channel('main_channel', 'App Main Channel', firebase.notifications.Android.Importance.Max)
      .setDescription('My App Mobile Main Notification Channel ')
    // Create the channel
    firebase.notifications().android.createChannel(channel)
    this._registerAppListener()
    // Here we retrieve the notification that woke the App from state CLOSED and handle it
    firebase.notifications().getInitialNotification()
      .then((notificationOpen) => {
        if (notificationOpen) {
        // Get information about the notification that was opened
          const notif = notificationOpen.notification;
          this._dealWithNotification(notif, 'app_closed')
        }
      })
    // As of Android M permissions must be requested explicitly
    if (!await firebase.messaging().hasPermission()) {
      try {
        await firebase.messaging().requestPermission();
      } catch (e) {
        // alert('Failed to grant permission')
      }
    }
    firebase.messaging().getToken().then((token) => {
      this.setState({ fcm_token: token })
      ZendeskChat.setPushToken(token)
    })
  }

  ...
```

#### Other Component

```javascript
...
import ZendeskChat from 'react-native-zendesk-zopim-chat';
...

ZendeskChat.startChat({
  name: user.full_name,
  email: user.email,
  phone: user.mobile_phone,
  tags: ['tag1', 'tag2'],
  department: "Your department"
});

```

## TODO

* Add examples