# Android integration guide into React Native CLI project

The following guide covers basic integration process into your React Native Expo project
where required part of an integration and customization of Banuba Video Editor SDK is implemented in **android** directory
of your React Native Expo project using native Android development process.

### Add SDK dependencies
Add Banuba repositories in [project gradle](../android/build.gradle#L55) file to get Video Editor SDK and AR Cloud dependencies.

```groovy
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/Banuba/banuba-ve-sdk")
            credentials {
                username = "Banuba"
                password = ""
            }
        }
        maven {
            name = "ARCloudPackages"
            url = uri("https://github.com/Banuba/banuba-ar")
            credentials {
                username = "Banuba"
                password = ""
            }
        }
```

Add Video Editor SDK dependencies in [app gradle](../android/app/build.gradle#L313) file.
```groovy
    def banubaSdkVersion = '1.26.3'
    implementation "com.banuba.sdk:ffmpeg:4.4"
    implementation "com.banuba.sdk:camera-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:camera-ui-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:core-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:core-ui-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-flow-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-timeline-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-ui-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-gallery-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-effects-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:effect-player-adapter:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ar-cloud:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-audio-browser-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:banuba-token-storage-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-export-sdk:${banubaSdkVersion}"
    implementation "com.banuba.sdk:ve-playback-sdk:${banubaSdkVersion}"
```

### Add SDK integration helper
Add [VideoEditorIntegrationHelper.kt](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorIntegrationHelper.kt) file
to initialize SDK dependencies. This class also allows you to customize many Video Editor SDK features i.e. min/max video durations, export flow, order of effects and others.

### Add React Package
First, add [VideoEditorModule](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorModule.kt) for communicating
between React Native and Video Editor SDK.

Next, create ```VideoEditorReactPackage``` and add ```VideoEditorModule``` to the list of modules.
```kotlin
 class VideoEditorReactPackage : ReactPackage {

    override fun createNativeModules(reactContext: ReactApplicationContext): MutableList<NativeModule> {
        val modules = mutableListOf<NativeModule>()
        modules.add(VideoEditorModule(reactContext))
        return modules
    }
    ...
}

```
Finally, add ```VideoEditorReactPackage```  to list of packages in [Application class](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/MainApplication.java#L31)
```java
        @Override
        protected List<ReactPackage> getPackages() {
            List<ReactPackage> packages = new PackageList(this).getPackages();
            packages.add(new VideoEditorReactPackage());
            ...
            return packages;
        }
```

### Update AndroidManifest
Add ```VideoCreationActivity``` in your [AndroidManifest.xml](../android/app/src/main/AndroidManifest.xml#L49) file.
The Activity is used to bring together a number of screens in a certain flow.

Next, allow Network by adding permissions
```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
```
and ```android:usesCleartextTraffic="true"``` in [AndroidManifest.xml](../android/app/src/main/AndroidManifest.xml).

Network traffic is used for downloading AR effects from AR Cloud and stickers from [Giphy](https://giphy.com/).

Please set up correctly [network security config](https://developer.android.com/training/articles/security-config) and use of ```android:usesCleartextTraffic```
by following [guide](https://developer.android.com/guide/topics/manifest/application-element).

### Add resources
Video Editor SDK uses a lot of resources required for running.  
Please make sure all these resources are provided in your project.

1. [bnb-resources](../android/app/src/main/assets/bnb-resources) to use built-in Banuba AR and Lut effects.
   Using Banuba AR ```assets/bnb-resources/effects``` requires [Face AR product](https://docs.banuba.com/face-ar-sdk-v1). Please contact Banuba Sales managers to get more AR effects.  

2. [color](../android/app/src/main/res/color),
   [drawable](../android/app/src/main/res/drawable),
   [drawable-hdpi](../android/app/src/main/res/drawable-hdpi),
   [drawable-ldpi](../android/app/src/main/res/drawable-ldpi),
   [drawable-mdpi](../android/app/src/main/res/drawable-mdpi),
   [drawable-xhdpi](../android/app/src/main/res/drawable-xhdpi),
   [drawable-xxhdpi](../android/app/src/main/res/drawable-xxhdpi),
   [drawable-xxxhdpi](../android/app/src/main/res/drawable-xxxhdpi) are visual assets used in views and added in the sample for simplicity. You can use your specific assets.  

3. [values](../android/app/src/main/res/values) to use colors and themes. Theme ```VideoCreationTheme``` and its styles use resources in **drawable** and **color** directories.  

### Configure media export
Video Editor SDK allows to export a number of media files to meet your requirements.
Implement ```ExportParamsProvider``` and provide ```List<ExportParams>``` where every ```ExportParams``` is a media file i.e. video or audio -
[See example](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorIntegrationHelper.kt#L248).

Use ```ExportParams.Builder.fileName()``` method to provide custom media file name.

### Start SDK
First, initialize Video Editor SDK using license token in ```VideoEditorModule``` on Android.
```kotlin
val videoEditor = BanubaVideoEditor.initialize(LICENSE_TOKEN)
```
Please note, instance ```videoEditor``` can be **null** if the license token is incorrect.  
[See example](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorModule.kt#L97)

Next, to start Video Editor SDK from React Native use ```startAndroidVideoEditor()``` method defined in [App.js](../App.js#45).
It will open Video Editor SDK from camera screen.
```javascript
   function initVideoEditor() {
         VideoEditorModule.initVideoEditor(LICENSE_TOKEN);
   }
   
   async function startAndroidVideoEditor() {
         initVideoEditor();
         return await VideoEditorModule.openVideoEditor();
   }
       
   <Button title = "Open Video Editor" onPress={async () => {
                if (Platform.OS === 'android') {
                    startAndroidVideoEditor().then(videoUri => {
                        // Handle received exported video
                    }).catch(e => {
                        // Handle error 
                    })
                }
            }
   />
   ```

Since Video Editor SDK is launched within ```VideoCreationActivity``` exported video is returned from the Activity
into [onActivityResult](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorModule.kt#45).

Export returns [videoUri](../App.js#L111) path as a ```String``` value where exported video stored to ReactNative.

[Promises](https://reactnative.dev/docs/native-modules-android#promises) is used to make a bridge between Android and ReactNative.

### Enable custom Audio Browser experience
Video Editor SDK allows to implement your experience of providing audio tracks for your users - custom Audio Browser.  
To check out the simplest experience on Flutter you can set ```true``` to [CONFIG_ENABLE_CUSTOM_AUDIO_BROWSER](../android/app/src/main/java/com/vesdkreactnativecliintegrationsample/VideoEditorIntegrationHelper.kt#L62)  
:exclamation:<ins>Video Editor SDK can play only files stored on device.</ins>

## What is next?
We have covered a basic process of integration Banuba Video Editor SDK into your React Native CLI project.</br>
More integration details and customization options you will find in [Banuba Video Editor SDK Android Integration Sample](https://github.com/Banuba/ve-sdk-android-integration-sample).