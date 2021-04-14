---
layout: post
title:  "TensorFlow Lite, Android NDK"
author: Zimeng Lyu
date:   2018-11-27 10:35:26 -0500
categories: en ml android tensorflow
---

Github: [TFLiteExample][project-repo]

# Purpose

I could not find a comprehensive and easy to understand tutorial on getting TensorFlow Lite working with native code and the Android NDK. So I decided to write one.

_Note: I strongly recommend you use Mac OS to build and configure your Tensorflow Lite libraries. I found Windows 10 build steps unreliable and difficult to work with at the time of writing this guide._

# Lets Begin

We'll start by creating an Android Studio project in Android Studio. If you already have an Android Studio project you can skip these steps, but make sure C++ support is enabled (default toolchain).

_Note: If you don't have Android Studio installed you can [Get Android Studio][android-studio]_

An Android Studio project is the foundation for all Android platform applications. The project directory contains all of your graphical interfaces, source code, images and resources. It also includes the references and build instructions for your library code. You may know some of them: CMakeLists.txt, build.gradle, android.mk etc. While this guide does not get into details about these resources, you should learn how they work in your projects.

First, open Android Studio and click "Start a new Android Studio project". This will initiate the new project wizard.

![](/images/tflite-android/1.png)

Next, give your project an application name, company domain (important when deploying your app to the Play Store and also influences your Java code namespaces), project location, and package name. 

Be sure to check "Include C++ support," as this will enable many of the features required to interact with the Android NDK, Java's JNI, and C++ source code. More on [working with native code and Android Studio][add-native-code].

![](/images/tflite-android/2.png)

We will use Phone and Tablet devices for this guide, but other target devices should also work.

Optionally, you can select instant run. This will speed up APK deployment times while debugging your simulator and/or devices. Read more about [instant run][instant-run].

![](/images/tflite-android/4.png)

For a tutorial project we'll choose an Empty Activity. The Basic Activity includes a bit more skeleton code you may find useful. You can read more about activities [here][intro-to-activities].

![](/images/tflite-android/5.png)

On the next screen, we'll just continue with the defaults the project wizard provides us.

![](/images/tflite-android/6.png)

Determining what C++ Standard to use or when to upgrade can be tricky. The idea here is to use the C++ Standard thats most compatible your libraries, dependencies and existing source code. For this tutorial we'll use the Toolchain Default. But future versions of your projects may use more modern C++... 

Optionally you can enable Exceptions Support and Runtime Type Information. These allow native code exceptions to reach the Java Runtime and type information on dynamic types in your native code (e.g. auto types).

![](/images/tflite-android/7.png)

After finishing the Android Studio project wizard you should see the Android Studio main window.

![](/images/tflite-android/8.png)

# Building TensorFlow Lite [libtensorflowlite.so]

Now that we've setup our Android Studio project we can start building TensorFlow Lite. And the first thing we need to do is [Get Bazel][bazel-url]. Install steps below...

_Note: If you do not have Homebrew installed on your machine I strongly recommend you get it. It is one of the greatest package managers on Mac OS. [Get Homebrew][homebrew-url]_

Installing Homebew (`brew`):

~~~
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

Installing Bazel with Homebrew is the quickest way to get going:

~~~
brew tap bazelbuild/tap
brew tap-pin bazelbuild/tap
brew install bazelbuild/tap/bazel
~~~

You can then verify `bazel` is working correctly. I used `0.18.1-homebrew` so your version may differ or may be incompatible at the time this article was written.

~~~
bazel version
~~~

Output:

~~~
ZimengMacbookPro:zimenglyu zimenglyu$ bazel version
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
Build label: 0.18.1-homebrew
Build target: bazel-out/darwin-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Fri Nov 2 11:16:42 2018 (1541157402)
Build timestamp: 1541157402
Build timestamp as int: 1541157402
~~~

Next, we need to install the required Android NDK and SDK versions in Android Studio:

Go to the SDK Manager in Android Studio:

![](/images/tflite-android/10.png)

Then go to SDK Tools, and install the 4 items I highlighted here. SDK version `>=23`, Android SDK Build Tools API `>= 26.0.1` and NDK `>= 18` are required. Save the Android SDK Location and we will use it later.

![](/images/tflite-android/11.png)

If you can't see the Android SDK Build Tool version as shown above, your Android Studio has more than one version installed. Check the Show Pachage Details at bottom right and you would see all the Android SDK Build Tool versions you've installed.

![](/images/tflite-android/12.png)

Clone the official Tensorflow Github repository:

~~~
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
~~~

Now lets edit the `WORKSPACE` file found inside the root directory of Tensorflow repo. Add the following lines to the end of the `WORKSPACE` file. Make sure the path and package versions match the ones you've installed in Android Studio. Use the Android SDK path you saved earlier as the `android_sdk_repository` path here.

~~~
android_sdk_repository(
   name = "androidsdk",
   api_level = 28,
   build_tools_version = "28.0.3",
   path = "/Users/xxxx/Library/Android/sdk",
)

android_ndk_repository(
   name="androidndk",
   path="/Users/xxxx/Library/Android/sdk/ndk-bundle",
   api_level=18
)
~~~

Run `./configure` and answer yes when asked to automatically configure the `./WORKSPACE`. And match the SDK and NDK versions and paths with what you installed in Android Studio.

Next, go to `tensorflow/lite` directory and add the following to the `BUILD` file:

_NOTE: older versions of the Tensorflow repository locate the `lite` folder inside `tensorflow/contrib`. These `BUILD` settings will not work unless you adjust for the proper paths._

~~~
cc_binary(
    name = "libtensorflowLite.so",
    linkopts=[
        "-shared", 
        "-Wl,-soname=libtensorflowLite.so",
    ],
    linkshared = 1,
    copts = tflite_copts(),
    deps = [
        ":framework",
        "//tensorflow/lite/kernels:builtin_ops",
    ],
)
~~~

Now we can build the libtensorflowlite.so file by running the following command from the root directory of the Tensorflow repository:

~~~
bazel build //tensorflow/lite:libtensorflowLite.so --crosstool_top=//external:android/crosstool --cpu=armeabi-v8a --host_crosstool_top=@bazel_tools//tools/cpp:toolchain --cxxopt="-std=c++11"
~~~

When I ran this command, I encountered the following warning:

~~~
WARNING: Duplicate rc file: /Users/xxxx/Documents/GitHub/tensorflow/tools/bazel.rc is read multiple times, most recently imported from /Users/zimenglyu/Documents/GitHub/tensorflow/.bazelrc
~~~

If you get the same warning make sure you don't have multiple `.bazelrc` files (may be hidden). I found one in the root directory of my local Tensorflow repo. Removing it cleared this warning and allowed me to continue compiling the tflite library.

After successfully running the bazel command, you can find `libtensorflowlite.so` in `bazel-out/arm64-v8a-opt/bin/tensorflow/lite/` folder. The folder is aliased in the root directory of your Tensorflow repository.

Next, we'll edit the CMakeLists.txt in your Android Studio and add the following:

~~~
set(pathToTensorflowLite /Users/xxxx/Documents/cpplibs/libraries/tensorflow-lite/distribution)

add_library(libtensorflowLite SHARED IMPORTED)

set_target_properties(libtensorflowLite PROPERTIES IMPORTED_LOCATION
    ${pathToTensorflowLite}/lib/${ANDROID_ABI}/libtensorflowLite.so)

target_include_directories(native-lib PRIVATE
            ${pathToTensorflowLite}/include)

target_link_libraries( native-lib
                       libtensorflowLite
                       ${log-lib} )
~~~

The file structure of the example above is:

* distribution
  * lib
    * armeabi-v8a
      * libtensorflowLite.so
  * include
    * flatbuffers
    * tensorflow

clone the flatbuffers repository:

~~~
git clone https://github.com/google/flatbuffers.git
~~~

Put all the flatbuffer header files in the `distribution/include/flatbuffers` folder. And put the `tensorflow/tensorflow` folder in the `distribution/include` folder

Open the build.gradle(Module:App) and add the following lines to Andriod section:

~~~
sourceSets {
        main {
            // let gradle pack the shared library into apk
            jni.srcDirs = ["libs"]
        }
    }
~~~

[project-repo]: https://github.com/zimenglyu/TFLiteExample
[android-studio]: https://developer.android.com/studio/
[add-native-code]: https://developer.android.com/studio/projects/add-native-code
[instant-run]: https://developer.android.com/studio/run/#instant-run
[intro-to-activities]: https://developer.android.com/guide/components/activities/intro-activities
[bazel-url]: https://bazel.build
[homebrew-url]: https://brew.sh
