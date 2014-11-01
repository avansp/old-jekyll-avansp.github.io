---
layout: post
title: Building ITK 4.6.1 on Mac OS X 10.9.5
categories: build
comments: true
---

Always find this task annoying. You need to build, build and build when new releases are released. Not to mention the new habit of Apple to deviate from common sense: change a compiler, the way its SDK is located, etc. This post is just a reminder to myself when I need to re-build my [ITK][itk].

My current settings are:
{% highlight bash %}
OSX: 10.9.5
Xcode: SDK 10.9 (have 10.10, but my OS is still 9)
CMAKE: 3.1.0-rc1
ITK: 4.7
{% endhighlight %}

From a *fresh start*, meaning new directory to build. Run CMAKE and generate. You can use any generator, but I chose `CodeBlocks - Unix Makefile`, because [Qt][qt] can read `CMake` project structure if we use `CodeBlocks` generator.

Set the correct C & C++ compiler (*I don't want to mix up with GNU*)
{% highlight bash %}
CMAKE_CXX_COMPILER = /usr/bin/clang++
CMAKE_C_COMPILER = /usr/bin/clang
{% endhighlight %}

Spotted [here](http://public.kitware.com/pipermail/community/2014-February/001312.html), set the correct compiler flags:
{% highlight bash %}
CMAKE_CXX_FLAGS = -stdlib=libstdc++ -std=c++11
CMAKE_OSX_SYSROOT = /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk
CMAKE_OSX_DEPLOYMENT_TARGET = 10.9
CMAKE_OSX_ARCHITECTURES = x86_64
{% endhighlight %}

Other settings are

* `CMAKE_INSTALL_PREFIX`. Make sure it does not install directly to `/usr/local/` folder by default.

* `ITKVideoBridgeOpenCV` module enabled to link [ITK][itk] with [OpenCV][openCV].

Okay, give it a go!

**It didn't work**

So I tweaked, tried, and nothing worked, until I hacked directly `CMakeCache.txt` file.

It appears that even though I told CMake that I used LLVM's `clang` compiler (Apple changed that since 10.9 from GNU), CMake still uses `<type_traits>` inclusion instead of `<tr1/type_traits>`.

I forced CMake, by changing these lines in the `CMakeCache.txt` file manually:
{% highlight bash %}
//Have include tr1/type_traits
ITK_HAS_STLTR1_TR1_TYPE_TRAITS:INTERNAL=
//Have include type_traits
ITK_HAS_STLTR1_TYPE_TRAITS:INTERNAL=1
{% endhighlight %}

into:
{% highlight bash %}
//Have include tr1/type_traits
ITK_HAS_STLTR1_TR1_TYPE_TRAITS:INTERNAL=1
//Have include type_traits
ITK_HAS_STLTR1_TYPE_TRAITS:INTERNAL=
{% endhighlight %}

It seems to work... still compiling. No complaints about type traits.

[itk]: http://www.itk.org/
[qt]: http://qt-project.org/
[openCV]: http://opencv.org/
