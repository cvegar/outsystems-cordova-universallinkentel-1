Ionic Deeplinks Plugin
======

This plugin makes it easy to respond to deeplinks through custom URL schemes
and Universal/App Links on iOS and Android.

For example, you can have your app open through a link to https://yoursite.com/product/cool-beans and then navigate
to display the Cool Beans in your app (cool beans!).

Additionally, on Android iOS, your app can be opened through a custom URL scheme, like `coolbeans://product/cool-beans`.

Since Custom URL scheme behavior has changed quite a bit in iOS 9.2 for the case where the app isn't installed, you'll want to start using [Universal Links](#ios-configuration) as it's clear custom URL schemes are on the way out.

*Note: this plugin may clash with existing Custom URL Scheme and Universal Links Plugins. Please let
us know if you encounter compatibility issues. Also, try removing them and using this one on its own.*

##ORIGINAL plugin
[ionic-plugin-deeplinks](https://github.com/ionic-team/ionic-plugin-deeplinks)

## VALID Universal link DOMAIN defined for Entel App Personas
- app.entel.cl

## VALID Universal Link paths defined for Entel App Personas
- /open
- /Entel/*
- /ul/*

## VALID production ready app IDs 
- cl.entel.appswlsdesa
- N4YLN8K7AT.cl.entel.appentel

## VALID debug-test app IDs 
- cl.entel.appswlsdesa.debug

## Configurations deployed on app.entel.cl
Entel uses old IIS for public site. This forces a 'hardcoded' solution to serve the required files that are validated by Android/iOS when users install the app: they are served by F5 on https://app.entel.cl/.well-known/

#### assetlinks.json (Android)
Android handles valids paths from settings in AndroidManifest.xml and, because of that, assetlinks.json is ready and do not need any changes. If more paths are needed, PLEASE update apple-app-site-association file on server.
To enable this app in another domain (example: testapp.entel.cl) only copy this file in the respective /.well-known folder for that subdomain.  

#### apple-app-site-association (iOS)
iOS handles valid paths from this config file. Adding new paths to Universal Links requires an update/deploy to this file before trying to use in app. PLEASE, update AndroidManifest.xml with this new paths.
To enable this app in another domain (example: testapp.entel.cl) only copy this file in the respective /.well-known folder for that subdomain.  

## Installation
server-configs folder in this repository contains files required to serve only by https scheme in /.well-known folder on ALL domains. Failing to deploy in ALL domains, invalidates universal link FOR ALL DOMAINS. 


## Handling Deeplinks in JavaScript

#### Non-Ionic/angular

Ionic Native works with non-Ionic/Angular projects and can be accessed at `window.IonicNative` if imported.

If you don't want to use Ionic Native, the plugin is available on `window.IonicDeeplink` with a similar API minus the observable callback:

```javascript
window.addEventListener('deviceready', function() {
  IonicDeeplink.route({
    '/product/:productId': {
      target: 'product',
      parent: 'products'
    }
  }, function(match) {
  }, function(nomatch) {
  });
})
```

#### Ionic/Angular 2

*note: make sure to call IonicDeeplink from a platform.ready or `deviceready` event*


```javascript
import { Platform, NavController } from 'ionic-angular';
import { Deeplinks } from '@ionic-native/deeplinks';

export class MyApp {
  constructor(
    protected platform: Platform
    , protected navController: NavController
    , protected deeplinks: Deeplinks
    ) {
    this.platform.ready().then(() => {
      this.deeplinks.route({
        '/about-us': HomePage,
        '/products/:productId': HelpPage
      }).subscribe((match) => {
        // match.$route - the route we matched, which is the matched entry from the arguments to route()
        // match.$args - the args passed in the link
        // match.$link - the full link data
        console.log('Successfully matched route', match);
      },
      (nomatch) => {
        // nomatch.$link - the full link data
        console.error('Got a deeplink that didn\'t match', nomatch);
      });
    });
  }
}

// Note: routeWithNavController returns an observable from Ionic Native so it *must* be subscribed to first in order to trigger.
```

If you're using Ionic 2, there is a convenience method to route automatically:

```javascript
import { Platform, NavController } from 'ionic-angular';
import { Deeplinks } from '@ionic-native/deeplinks';

export class MyApp {
  constructor(
    protected platform: Platform
    , protected navController: NavController
    , protected deeplinks: Deeplinks
    ) {
    this.platform.ready().then(() => {
      this.deeplinks.routeWithNavController(this.navController, {
        '/about-us': HomePage,
        '/products/:productId': HelpPage
      }).subscribe((match) => {
        // match.$route - the route we matched, which is the matched entry from the arguments to route()
        // match.$args - the args passed in the link
        // match.$link - the full link data
        console.log('Successfully matched route', match);
      },
      (nomatch) => {
        // nomatch.$link - the full link data
        console.error('Got a deeplink that didn\'t match', nomatch);
      });
    });
  }
}

// Note: routeWithNavController returns an observable from Ionic Native so it *must* be subscribed to first in order to trigger.
```

#### Ionic/Angular 1

For Ionic 1 and Angular 1 apps using Ionic Native, there are many ways we can handle deeplinks. However,
we need to make sure we set up a history stack for the user, we can't navigate directly to our page
because Ionic 1's navigation system won't properly build the navigation stack (to show a back button, for example).

This is all fine because deeplinks should provide the user with a designed experience for what the back button
should do, as we are putting them deep into the app and need to provide a natural way back to the main flow:


```javascript
angular.module('myApp', ['ionic', 'ionic.native'])

.run(['$ionicPlatform', '$cordovaDeeplinks', '$state', '$timeout', function($ionicPlatform, $cordovaDeeplinks, $state, $timeout) {
  $ionicPlatform.ready(function() {
    // Note: route's first argument can take any kind of object as its data,
    // and will send along the matching object if the route matches the deeplink
    $cordovaDeeplinks.route({
      '/product/:productId': {
        target: 'product',
        parent: 'products'
      }
    }).subscribe(function(match) {
      // One of our routes matched, we will quickly navigate to our parent
      // view to give the user a natural back button flow
      $timeout(function() {
        $state.go(match.$route.parent, match.$args);

        // Finally, we will navigate to the deeplink page. Now the user has
        // the 'product' view visibile, and the back button goes back to the
        // 'products' view.
        $timeout(function() {
          $state.go(match.$route.target, match.$args);
        }, 800);
      }, 100); // Timeouts can be tweaked to customize the feel of the deeplink
    }, function(nomatch) {
      console.warn('No match', nomatch);
    });
  });
}])
```



## iOS Configuration

As of iOS 9.2, Universal Links *must* be enabled in order to deep link to your app. Custom URL schemes are no longer supported.

Follow the official [Universal Links](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html) guide on the Apple Developer docs
to set up your domain to allow Universal Links.

## Android Configuration

Android supports Custom URL Scheme links, and as of Android 6.0 supports a similar feature to iOS' Universal Links called App Links.

Follow the App Links documentation on [Declaring Website Associations](https://developer.android.com/training/app-links/index.html#web-assoc) to enable your domain to
deeplink to your Android app.

To prevent Android from creating multiple app instances when opening deeplinks, you can add the following preference in Cordova config.xml file:

```xml
 <preference name="AndroidLaunchMode" value="singleTask" />
```
