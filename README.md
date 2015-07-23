# j2objc-gradle
Gradle Plugin for [J2ObjC](https://github.com/google/j2objc), which is an open-source tool
from Google that translates Java source code to Objective-C for the iOS (iPhone/iPad) platform.
The plugin is not affiliated with Google but was developed by former Google Engineers and others.
J2ObjC enables Java source to be part of an iOS application's build, as no editing of the
generated files is necessary. The goal is to write an app's non-UI code (such as application
logic and data models) in Java, which is then shared by web apps (using GWT), Android apps,
and iOS apps.


### Usage
At HEAD, this plugin is in a state of significant flux as we refactor it into a first-class
Gradle plugin for our beta. You may wish to wait for the beta release as we may make backwards
incompatible changes before that point.

You should start with a clean java only project without any android dependencies. It suggested that
this project is named `'shared'`. It must be buildable using Gradle's standard `'java'` plugin.
It may start as an empty project and allows you to gradually shift over code from an existing
Android application. See the section below on [Folder Structure](#folder-structure).

This is how to configure the build.gradle in your java only project. Please follow the link to
find the latest version number of the plugin:

    // File: shared/build.gradle
    plugins {
        id 'java'
        id 'com.github.j2objccontrib.j2objcgradle' version '0.3.0-alpha'
    }

    // Plugin settings:
    j2objcConfig {
        xcodeProjectDir '../ios'  // Xcode workspace directory
        xcodeTarget 'IosApp'      // iOS app target name

        // Other Settings:
        // https://github.com/j2objc-contrib/j2objc-gradle/blob/master/src/main/groovy/com/github/j2objccontrib/j2objcgradle/J2objcConfig.groovy#L30

        finalConfigure()          // Must be last call to configuration
    }

Within the Android application's project `build.gradle`, make it dependent on the `shared` project:

    // File: android/build.gradle
    dependencies {
        compile project(':shared')
    }


### NOTE: Open .xcworkspace in Xcode

When using the j2objcXcodeTask, open the `.xcworkspace` file in Xcode. If the `.xcodeproj` file
is opened in Xcode then CocoaPods will fail. This will appear as an Xcode build time error:

    library not found for -lPods-*-j2objc-shared


### Folder Structure

This is the suggested folder structure. It also shows a number of generated files and
folders that aren't committed to your repository. Files are shown before folders, so it
is not in strict alphabetical order.

    workspace
    ├── .gitignore                     // Should exclude: local.properties, settings.gradle, build/, ...
    ├── build.gradle
    ├── local.properties               // sdk.dir=<Android SDK> and j2objc.home=<J2ObjC>, .gitignore exclude
    ├── settings.gradle                // include ':android', ':shared'
    ├── android
    │   ├── build.gradle               // dependencies { compile project(':shared') }
    │   └── src/...                    // src/main/java/... and more, only Android specific code
    ├── ios
    │   ├── iosApp.xcworkspace         // Xcode workspace
    │   ├── iosApp.xcodeproj           // Xcode project, which is modified by j2objcXcode / CocoaPods
    │   ├── Podfile                    // j2objcXcode modifies this file for use by CocoaPods, committed
    │   ├── iosApp/...                 // j2objcXcode configures dependency on j2objcOutputs/{libs|src}
    │   ├── iosAppTests/...            // j2objcXcode configures as above but with "debug" buildType
    │   └── Pods/...                   // generated by CocoaPods for Xcode, .gitignore exclude
    └── shared
        ├── build.gradle               // apply 'java' then 'j2objc' plugins
        ├── build                      // generated build directory, .gitignore exclude
        │   ├── ...                    // other build output
        │   ├── binaries/...           // Contains test binary: testJ2objcExecutable/debug/testJ2objc
        │   ├── j2objc-shared.podspec  // j2objcXcode generates these settings to modify Xcode
        │   └── j2objcOutputs/...      // j2objcAssemble copies libraries and headers here
        ├── lib                        // library dependencies, must have source code for translation
        │   └── lib-with-src.jar       // library with source can be translated
        └── src/...                    // src/main/java/... shared code for translation


### Tasks

These are the main tasks for the plugin:

    j2objcCycleFinder       - Find cycles that can cause memory leaks, see notes below
    j2objcTranslate         - Translates to Objective-C
    j2objcAssemble          - Builds debug/release libraries, packs targets in to fat libraries
    j2objcTest              - Runs all JUnit tests, as translated into Objective-C
    j2objcBuild             - Builds and tests all j2objc outputs.
    j2objcXcode             - Configure Xcode to link static library & header files, uses CocoaPods

Note that you can use the Gradle shorthand of `$ gradlew jA` to do the `j2objcAssemble` task.
The other shorthand expressions are `jCF, jTr, jA, jTe, jB and jX`.


### Faster Development Cycle

If you are developing in a tight modify-compile-test loop and using only debug binaries, you
may want to disable the release build temporarily by adding to your `local.properties` file:

    j2objc.releaseEnabled=false

This should cut the j2objc build time up to 50%.  You can also do this for `j2objc.debugEnabled`.


### FAQ

See [FAQ.md](FAQ.md).


### Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md#quick-start).


### License

This library is distributed under the Apache 2.0 license found in the
[LICENSE](./LICENSE) file.
