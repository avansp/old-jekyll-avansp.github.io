---
layout: post
title:  "CMake tricks"
date:   2015-03-12
tags: [en]
excerpt: "Debugging, auto generated variables, option grouping, C++11 compiler check"
---

## Debugging with CMake

Two simple commands that help me to debug with `CMake`:

* `MESSAGE(<mode> msg_string)`
   This command outputs a string during the configure process. I usually insert this command to check values of some variables, e.g.
{% highlight cmake %}
MESSAGE(STATUS "PROJECT_BINARY_DIR = ${PROJECT_BINARY_DIR}")
{% endhighlight %}

* `RETURN()`
   This command just stops the configuration execution, which helps me to see the intermediate values without finishing the whole `CMake` process.

## Auto-generated project variable names

There are certain variables that `CMake` generates when we declare a project a name. These are variables prefixed with `<PROJECT_NAME>_` name in the `CMake` documentation page. These variables are especially handy if you have subprojects under your main project. The most important ones are binary and source directories:

{% highlight cmake %}
PROJECT( AWESOME )
MESSAGE( STATUS "project name = ${PROJECT_NAME}" )
MESSAGE( STATUS "source dir = ${AWESOME_SOURCE_DIR}")
MESSAGE( STATUS "binary dir = ${AWESOME_BINARY_DIR}")
{% endhighlight %}

## Grouping options

Values that we allow user to modify are set by `OPTION(<variable_name> help_string [initial_value])` command. However, it would be nice if these options are grouped nicely to our project or subproject names. To do that, we just make sure that the variable names are prefixed with the same name. The `CMake` gui will automatically group them based on their prefix. E.g.,

{% highlight cmake %}
OPTION( AWESOME_BUILD_TEST "Build test applications for the awesome project", OFF )
OPTION( AWESOME_INIT_RAND "Set initial random number seeds", 200 )
{% endhighlight %}

will group these options under **AWESOME** heading in the `CMake` gui application.

## Checking C++11 compiler

I found this snippets from [Guy Rutenberg](http://www.guyrutenberg.com/2014/01/05/enabling-c11-c0x-in-cmake/) to check C++11 compiler on different platforms:
{% highlight cmake %}
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if( COMPILER_SUPPORTS_CXX11 )
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif( COMPILER_SUPPORTS_CXX0X )
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
	message( FATAL "The compiler ${CMAKE_CXX_COMPILER} does not support C++11." )
endif()
{% endhighlight %}
