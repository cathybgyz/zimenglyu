Github: [TFLiteExample][project-repo]

# 前言

我一直找不到一个关于在 Android NDK 用 C++ TensorFlow Lite 全面而且细致的教程，所以在探索和尝试之后写下这篇与大家分享。

_请注意：强烈建议大家用 Mac OS 环境编译，在写这篇文章之前我尝试过用 Win 10，但是很不好用。而且TensorFlow Github 的 demo 上面说 bazel 目前不支持 Windows_


# 步骤

我们首先要在 Android Studio 上面创建一个 Android Studio project。如果你已经在 Android Studio 上面建好了 project, 可以跳过这个步骤， 但是要确保添加了C++支持。


_注：如果你还没有安装 Android Studio， 可以在[这里下载][android-studio]_

Android Studio project 是所有Android平台应用程序的基础。项目目录包含所有图形界面，源代码，图像和其他资源。它还包括库代码的引用和构建说明。你应该了解基本的关于：CMakeLists.txt，build.gradle，android.mk等的工作方式，这里没有详细介绍。

首先，打开 Android Studio 并点击 "Start a new Android Studio project" 来打开新的项目页面。


![](/img/tflite-android/1.png)

接着，填写应用程序名称，公司域名（在将应用程序部署到Play商店时很重要，也会影响Java代码名称空间），项目位置和 package 名称。

要确保选择 "Include C++ support," 这样会开启很多与 Android NDK, Java's JNI, 和 C++源代码相关的功能。更多请参考[working with native code and Android Studio][add-native-code]。

![](/img/tflite-android/2.png)

我们这里介绍使用 “Phone and Tablet devices”，其他的设备应该也可以。

（可选项）另外，你可以选择 instant run，这个会在你的安卓虚拟机/设备debug的时候加速 APK 部署。更多关于[instant run][instant-run]。

![](/img/tflite-android/4.png)

在这个示例项目中，我们选择 “Empty Activity”。在 ”Basic Activity" 中包含一些代码框架。更多关于 [avtivities][intro-to-activities]。

![](/img/tflite-android/5.png)

在这一页面中，我们保持初始设置不变。

![](/img/tflite-android/6.png)

决定用哪个 C++ Standard 或者什么时候要升级对每个人都不太一样。重点是要用能兼容你的库、依赖关系和你已经有的源代码。所以本文就使用 Toolchain Default，但是你的项目的未来的版本可能会需要新的 C++ Standard。

（可选项）你可以选择Exceptions Support 和 Runtime Type Information，这个会使你的源代码的 exception 在 Java Runtime 中键入动态类型的信息（比如 auto）。

![](/img/tflite-android/7.png)

在完成上述项目设定后，你会看到 Android Studio 主页面。

![](/img/tflite-android/8.png)

# 建立 TensorFlow Lite [libtensorflowlite.so, libtflite.so]

我们创建完Android Studio project 后要建立 TensorFlow Lite 库。我们首先要做的是：[Get Bazel][bazel-url]， 步骤如下：

_注：如果你没有安装 Homebrew，我强烈推荐你使用，Homebrew 是 Mac OS 环境里最好的资源管理程序之一。[Get Homebrew][homebrew-url]_

安装 Homebrew (`brew`):

~~~
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

用 Homebrew 安装 Bazel 是最快的安装 Bazel 的方法。

~~~
brew tap bazelbuild/tap
brew tap-pin bazelbuild/tap
brew install bazelbuild/tap/bazel
~~~

你可以检查 `bazel` 是否可以正常运行. 这里我装了 `0.18.1-homebrew` 所以你的安装版本可能会和我写这篇文章的时候版本不同。

~~~
bazel version
~~~

输出:

~~~
ZimengMacbookPro:zimenglyu zimenglyu$ bazel version
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
Build label: 0.18.1-homebrew
Build target: bazel-out/darwin-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Fri Nov 2 11:16:42 2018 (1541157402)
Build timestamp: 1541157402
Build timestamp as int: 1541157402
~~~

[project-repo]: https://github.com/zimenglyu/TFLiteExample
[android-studio]: https://developer.android.com/studio/
[add-native-code]: https://developer.android.com/studio/projects/add-native-code
[instant-run]: https://developer.android.com/studio/run/#instant-run
[intro-to-activities]: https://developer.android.com/guide/components/activities/intro-activities
[bazel-url]: https://bazel.build
[homebrew-url]: https://brew.sh
