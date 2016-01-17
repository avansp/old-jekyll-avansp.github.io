---
layout: post
title:  "Understanding XNAT pipeline"
date:   2016-01-03
tags: [en]
comments: true
---

Pipeline is a powerful feature of [XNAT 1.6](https://wiki.xnat.org/display/XNAT16/Home), because it allows you to run shell-like program by using only XML files. If you have a command line executable for your project/subject/session/images, then you can create an XML file using the [XNAT pipeline schema](https://wiki.xnat.org/display/XNAT16/XNAT+Pipeline+Development+Schema) to ask XNAT to run automatically whenever a user upload new images. However, *as mostly an open-source framework does*, XNAT does not provide clear documentations on how to create a pipeline. The only lengthly explanation is given by a video tutorial at XNAT 2012 Workshop presented by Mohana Ramaratnam (the inventor of XNAT pipeline):

<iframe src="https://player.vimeo.com/video/47245858" width="500" height="375" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

Other useful information:


## Setting up pipeline in custom location

By default, XNAT pipeline is located at `XNAT_HOME/pipeline` (we call it `PIPELINE_HOME`), but it's advisable to manage your own pipelines in different location to avoid being overwritten during upgrade[^1]. Here's my setting:
{% highlight text %}
/opt/xnat/tools/pipelines
/opt/xnat/tools/bin
/opt/xnat/tools/logs
/opt/xnat/tools/resources
{% endhighlight %}

The `pipelines` contains XML files for the XNAT pipelines, `bin` is for your command line executables, `logs` to store outputs and log files when your pipelines are executed and `resources` contains XML files for resource descriptors that link the pipeline engine with executables.

To understand how to create XML pipelines and resources, it's better to look around pre-defined pipelines in `PIPELINE_HOME` folder. Note that in the resource descriptors if there is no absolute path, the relative path is `PIPELINE_HOME/catalog`.

## Installing pipeline in the XNAT

First, we need to create a new menu item in the XNAT's experiment so that we can trigger our pipeline manually for testing. *Note that you can run pipeline at any level, but somehow XNAT only allows us to see the history and manage running pipeline at experiment level*. An experiment level is similar to imaging sessions such as MR Session, CT Session, etc.

* Login as admin
* Open `Administer->Data Types`
* Select one of the imaging session (e.g. xnat:mrSessionData)
* Select action `Edit`
* In the **Available Report Actions**, add new action menu: Name=`PipelineScreen_launch_pipeline`, Display Name=`Pipeline`, Image=`wrench.gif`, Popup=`always`, Sequence=`18`
* Submit

You should have now a new menu item `Pipeline` in the action box.

Before attaching a pipeline to a project, you must make the pipeline available:

* Open `Administer->Pipelines`
* Click `Add pipeline to Repository` link
* Insert the absolute path to the pipeline XML file.
* Leave the custom UI empty
* Click add

Then you can attach that pipeline to a project.

## XNAT pipeline template

This is just a basic XML file for a template that does nothing but to call `echo` command by using a predefined GenericCommand resource descriptor (located at `PIPELINE_HOME/catalog/pipeline-tools`)[^2].

{% gist avansp/1466a1a168842607ea9d %}

## Where is the output?

1. If you run the pipeline and it works, the output is in `/opt/xnat/tools/log/[SessionLabel].err`
2. If the pipeline status shows Queued for a long time, then you can mark it as failed and see the problem in `PIPELINE_HOME/pipeline/logs/pipeline_[DATE]_[TIME].log`. Find a message marked with **ERROR**.

> Tips: every time a pipeline is executed, there is a log file. Make sure you clean them in both log dirs regularly.

## Some useful parameter snippets

A parameter define a variable that you can retrieve by using an XPATH expression. If you want to define a parameter, then you can put it under `parameters` element, but if you want to ask user to specify or change the parameter value, then put them under `input-parameters` element. Note that if you change input parameters, you must re-enable the pipeline from repository again in order to allow XNAT to build a UI for your input fields.

#### Getting the correct URL for your XNAT server ####
{% highlight xml %}
<parameter>
    <name>resolvedHost</name>
    <values>
        <unique>^if (boolean(/Pipeline/parameters/parameter[name='aliasHost']/values/unique)) then /Pipeline/parameters/parameter[name='aliasHost']/values/unique/text() else /Pipeline/parameters/parameter[name='host']/values/unique/text()^</unique>
    </values>
    <description>Use aliasHost if it exists, or host if not</description>
</parameter>
{% endhighlight %}

#### Get info from schema link ####

Put this under `<values></values>` element). E.g.
{% highlight xml %}
<!-- all scan IDs from the current image session where the pipeline is running -->
<schemalink>xnat:imageSessionData/scans/scan/ID</schemalink>
<!-- image session ID -->
<schemalink>xnat:imageSessionData/ID</schemalink>
{% endhighlight %}
A complete list can be found in [XNAT.xsd](https://central.xnat.org/schemas/xnat/xnat.xsd) file. The schemalink can only be put in the input parameters area.

#### Get the root path of the archived data ####

In the input-parameters (under documentation element), we define `imageSessionID` parameter taken from XNAT schema link xnat:imageSessionData/ID
{% highlight xml %}
<input-parameters>
    <parameter>
        <name>imageSessionID</name>
        <values>
            <schemalink>xnat:imageSessionData/ID</schemalink>
        </values>
        <description>Image session ID</description>
    </parameter>
</input-parameters>
{% endhighlight %}

Then under parameters element, define the archive path as:
{% highlight xml %}
<parameter>
    <name>archivepath</name>
    <values>
        <unique>^fileUtils:GetArchiveDirRootPath(/Pipeline/parameters/parameter[name='resolvedHost']/values/unique/text(), /Pipeline/parameters/parameter[name='user']/values/unique/text(), /Pipeline/parameters/parameter[name='pwd]/values/unique/text()'], /Pipeline/parameters/parameter[name='imageSessionID']/values/unique)^</unique>
    </values>
    <description>Root to the local archive path</description>
</parameter>
{% endhighlight %}

Write the XPATH:
```
^/Pipeline/parameters/parameter[name='archivepath']/values/unique/text()^
```
it will give you something like `/opt/xnat/data/archive/[PROJECT_NAME]/arc001/`.

## The undocumented but very useful FileUtils

In the XNAT pipeline, there is a very useful library to retrieve file information for you data (`fileUtils`) from the remote host, but alas there is no documentation of what functions that you can call, unless if you open [its source code directly](https://bitbucket.org/nrg/pipeline_imagingtools_1_6dev/). Here's a list what I've found useful (*as of 06/01/2016*):

{% highlight java %}
String GetCachePath(String host, String user, String pwd, String project);
String GetArchiveDirRootPath(String host, String user, String pwd, String imageSessionId)
{% endhighlight %}

To use the `fileUtils` library, you must add attribute in the Pipeline element like this:
{% highlight xml %}
<Pipeline xmlns:fileUtils="org.nrg.imagingtools.utils.FileUtils">
{% endhighlight %}

Then use it as an XPATH expression, e.g.
{% highlight xml %}
<parameter>
    <name>cachepath</name>
    <values>
        <unique>^fileUtils:GetCachePath(/Pipeline/parameters/parameter[name='resolvedHost']/values/unique/text(), /Pipeline/parameters/parameter[name='user']/values/unique/text(), /Pipeline/parameters/parameter[name='pwd']/values/unique/text(),/Pipeline/parameters/parameter[name='project']/values/unique/text())^</unique>
    </values>
</parameter>
{% endhighlight %}

## The parameters you get

These are parameters that you will automatically have when you run a pipeline:
{% highlight xml %}
id           : accession ID (experiment ID)
host         : URI to your XNAT server
user         : username that runs the pipeline
u            : identical with user
pwd          : user password
label        : image session label
project      : project name
mailhost     : hostname of the mail server
userfullname : user full name
builddir     : path to build directory that you can use
xnatserver   : the title name of your XNAT server
adminemail   : administrator email address
useremail    : user email address
workflowid   : not sure its purpose
{% endhighlight %}

[^1]: You can store your pipelines in any location as long as the user that runs XNAT -- *usually tomcat* -- owns it.
[^2]: If you open `PIPELINE_HOME/catalog/pipeline-tools/GenericCommand.xml` you can find that the actual script is located at `PIPELINE_HOME/bin/GenericCommand.sh`
