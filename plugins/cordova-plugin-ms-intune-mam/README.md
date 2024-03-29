

#﻿Microsoft Intune App SDK Cordova Plugin

##Overview

The Intune App SDK Cordova plugin enables [Intune mobile app management features](https://docs.microsoft.com/en-us/intune/deploy-use/protect-app-data-using-mobile-app-management-policies-with-microsoft-intune) in iOS and Android apps built with Cordova. The plugin allows developers to easily build in app restriction and data protection features into their Cordova-based app.

You will find that you can enable SDK features without changing your app’s behavior. Once you've built the plugin into your iOS or Android mobile app, the IT admin will be able to deploy policy via Microsoft Intune supporting a variety of features that enable data protection. We have built the plugin so that most the steps are automatically performed in the Cordova build process. As a result, you should be able to get your app enabled for management quickly. To get started, follow the steps below based on your target platform.

Before you install and use Microsoft Intune App SDK Cordova Plugin you **must**:

* Review the [Intune App SDK Cordova Plugin License Terms](https://github.com/msintuneappsdk/cordova-plugin-ms-intune-mam/blob/master/Intune_App_SDK_Cordova_plugin_RTM_license.pdf).
* Print and retain a copy of the license terms for your records. By downloading and using Intune App SDK Cordova plugin you agree to such license terms.  If you do not accept them, do not use the software.

See the [official documentation](https://docs.microsoft.com/en-us/intune/develop/intune-app-sdk) for a more detailed description of the Intune App SDK.

## Supported Scenarios

###Platforms
* Android
* iOS


###EMM scenarios

* Intune MAM on Intune MDM enrolled devices
* Intune MAM on third-party EMM enrolled devices
* Intune MAM on unenrolled, unmanaged devices

Cordova apps built with the Intune App SDK plugin can now receive Intune mobile application management (MAM) policies on both Intune mobile device management (MDM) enrolled devices and unenrolled devices.

If the device is not MDM-enrolled, the app will prompt the user to authenticate with an Intune-licensed account on launch. The app will receive Intune MAM policy set by the IT administrator of that account upon successful authentication.

## Prerequisites
### Android
* The latest Microsoft Intune Company Portal app must always be installed on the device.

### iOS
* The latest Microsoft Intune Company Portal app must be installed on the device for MDM features. It is **not** needed for MAM without device enrollment features.

* Version 0.8.0+ of the [Azure Active Directory Authentication Libraries (ADAL) plugin for Cordova](https://github.com/AzureAD/azure-activedirectory-library-for-cordova) is required.
  * **Note:** Due to an Apache Cordova bug the filed [here](https://issues.apache.org/jira/browse/CB-6227?jql=text%20~%20%22plugin%20dependency%22), apps that already have the plugin dependency will not automatically upgrade the plugin to the requested version.

## Quick start

1. Update your version of ADAL:

	```
	cordova plugin remove cordova-plugin-ms-adal
	cordova plugin add cordova-plugin-ms-adal@0.8.x
	```

2. Add the Intune App SDK for Cordova plugin:

	```
	cordova plugin add cordova-plugin-ms-intune-mam
	```

## How to build the plugin into your iOS app

You'll need to complete some steps for your app to be Intune MAM enabled. For convenience, these steps are performed automatically in the Cordova build process as a pre-build hook. As a result, the automated steps will modify your `*.pbxproj` , `*-Info.plist`, and `*.entitlements` files that are associated with a project configuration. If your project doesn't contain an entitlements file, the plugin will create one automatically.

This setup only supports a single target and will perform the configuration on the first target found if there are multiple targets. If the process fails, check that:

1. Your app's Xcode project is `[name].xcodeproj` where `[name]` is the value defined in `config.xml`
2. Your project uses the [standard Cordova app directory structure](https://cordova.apache.org/docs/en/latest/reference/cordova-cli/index.html#directory-structure).

## How to build the plugin into your Android app

1. Import this plugin with the latest Cordova tools. The plugin will be automatically invoked as an `after_compile` step.

2. The plugin will create a MAM enabled version of a built apk (Android API 14+) at the end of the build process. The build output will contain a `[Project]-intunewrapped-[Build_Configuration].apk` (e.g. `helloWorld-intunewrapped-debug.apk`).

The plugin only supports gradle builds.

Due to a Cordova bug filed [here](https://issues.apache.org/jira/browse/CB-9434) that causes certain Cordova hooks to be ignored on `cordova run`, to run the wrapped app from the command line, you must do the following:

```
$ cordova build
$ cordova run --nobuild
```


### Signing your Android app
To add signing information to the wrapped apk, modify `build.json` as you normally would for adding signing information. For example:
```
{
  "android": {
    "release": {
      "keystore": "your.keystore",
      "storePassword": "yourpassword",
      "alias": "youralias",
      "password" : "yourpassword",
      "keystoreType": ""
    }
  }
}
```

###Build for Android test only

1. Add `android:testOnly="true"` to the application node of the `AndroidManifest.xml` file.


2. **Cordova 6.x.x:** In `[PROJECT_ROOT]/platforms/android/cordova/lib/Adb.js`, change line 60 from

	```
	var args = ['-s', target, 'install'];
	```
	to
	```
	var args = ['-s', target, 'install', '-t'];
	```

The effect of this is to run `adb install` with the "-t" flag since `testOnly` apps are not installable without it.

### Debugging from Visual Studio
After launching the app for the first time you should see a dialog notifying you that the app is managed by Intune. Hit "Don't show again" and click the debug/run button again from VS for breakpoints to be hit.

## Known Limitations
### Android
* Multi-Dex support is incomplete.
* App must be target Android 4.0 (Android API 14) or above.

### iOS
* Whenever you modify the list of UTI’s under the **CFBundleDocumentTypes** node of the **Info.plist** file, you must clear the Intune UTI’s in the Imported UTI’s section of the same plist file (**UTImportedTypeDeclarations** node) before building again. All of the Intune UTI’s will start with the prefix `com.microsoft.intune.mam`.

* If you want to remove the Intune plugin from your Cordova project, you must also remove the iOS platform and re-add it to undo some of the Intune configuration in the .xcodeproj and .plist files.
