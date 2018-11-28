---
layout: post
title:  "TensorFlow Lite, C++ and Android NDK"
author: Zimeng Lyu
date:   2018-11-27 10:35:26 -0500
categories: android tensorflow machinelearning mobile english
---

# Preamble

This guide came necesity, to understand how Tensorflow Lite integrates with native C++ code and the Android NDK. The steps I present here come from many sources and snippits of information. Prior to writing this guide, I could not find a comprehensive and easy to understand tutorial on how to assemble everything together. So I decided to write one.

_Note: I strongly recommend you use Mac OS to build and configure your Tensorflow Lite libraries. I found Windows 10 build steps unreliable and difficult to work with at the time of writing this guide._

# Lets Begin

We'll start by creating an Android Studio project in Android Studio. If you already have an Android Studio project you can skip these steps, but make sure C++ support is enabled (default toolchain).

_Note: If you don't have Android Studio installed you can [Get Android Studio][android-studio]_

An Android Studio project is the foundation for all Android platform applications. The project directory contains all of your graphical interfaces, source code, images and resources. It also includes the refernces and build instructions for your library code. You may know some of them: CMakeLists.txt, build.gradle, android.mk etc. While this guide does not get into details about these resources, you should learn how they work in your projects.

First, open Android Studio and click "Start a new Android Studio project". This will initiate the new project wizzard.

![](/images/tflite-android/1.png)

Next, give your project an application name, company domain (important when deploying your app to the Play Store and also influences your Java code namespaces), project location, and package name. 

Be sure to check "Include C++ support," as this will enable many of the features required to interact with the Android NDK, Java's JNI, and C++ source code. More on [working with native code and Android Studio][add-native-code].

![](/images/tflite-android/2.png)

We will use Phone and Tablet devices for this guide, but other target devices should also work.

Optionally, you can select instant run. This will speed up APK deployment times while debugging your simulator and/or devices. Read more about [instant run][instant-run].

![](/images/tflite-android/4.png)

For a tutorial project we'll choose an Empty Activity. The Basic Activity includes a bit more skeleton code you may find useful. You can read more about activities [here][intro-to-activities].

![](/images/tflite-android/5.png)

On the next screen, we'll just continue with the defaults the project wizzard provides us.

![](/images/tflite-android/6.png)

Determining what C++ Standard to use or when to upgrade can be tricky. The idea here is to use the C++ Standard thats most compatible your libraries, dependencies and existing source code. For this tutorial we'll use the Toolchain Default. But future versions of your projects may use more modern C++... 

Optionally you can enable Exceptions Support and Runtime Type Information. These allow native code exceptions to reach the Java Runtime and type information on dynamic types in your native code (e.g. auto types).

![](/images/tflite-android/7.png)

After finishing the Android Studio project wizzard you should see the Android Studio main window.

![](/images/tflite-android/8.png)

# Building TensorFlow Lite [libtensorflowlite.so, libtflite.so]

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

[android-studio]: https://developer.android.com/studio/
[add-native-code]: https://developer.android.com/studio/projects/add-native-code
[instant-run]: https://developer.android.com/studio/run/#instant-run
[intro-to-activities]: https://developer.android.com/guide/components/activities/intro-activities
[bazel-url]: https://bazel.build
[homebrew-url]: https://brew.sh
