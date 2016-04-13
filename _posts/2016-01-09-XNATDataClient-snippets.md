---
layout: post
title:  "Snippets of XNATDataClient"
date:   2016-01-09
tags: [en]
comments: true
---

[XNATDataClient](https://wiki.xnat.org/display/XNAT16/XnatDataClient) or **XDC** is a command line tool from XNAT that you can use to **GET**, **POST**, **PUT** or **DELETE** data that are stored in the XNAT server. The tool is pretty undocumented. Here's my notes from my experiments with XDC.

## XNAT REST API

XDC makes use of the [XNAT REST API](https://wiki.xnat.org/display/XNAT16/Using+the+XNAT+REST+API) URI format in the *remote* or *r* argument. By the way, if you want to experiment with the **GET** method of XDC, you can try it directly with your browser, like these:

#### Get project list with 4 different output types:

[https://central.xnat.org/data/projects?format=html](https://central.xnat.org/data/projects?format=html)
[https://central.xnat.org/data/projects?format=xml](https://central.xnat.org/data/projects?format=xml)
[https://central.xnat.org/data/projects?format=json](https://central.xnat.org/data/projects?format=json)
[https://central.xnat.org/data/projects?format=csv](https://central.xnat.org/data/projects?format=csv)

*Note the default format is HTML.*

### Search XNAT data (XML output)

[https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/searches/@xnat:mrsessiondata](https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/searches/@xnat:mrsessiondata)

### Get patient & image session list

[https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/subjects](https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/subjects)
[https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/subjects/CENTRAL_S00113/experiments/](https://central.xnat.org/data/projects/CENTRAL_OASIS_LONG/subjects/CENTRAL_S00113/experiments/)

## Create an empty patient

> XNAT does not give any warnings if the new patient already exists.

Command line:
{% highlight console %}
$ XnatDataClient -u {USERNAME} -p {PASSWORD} -m PUT -r '{URL_TO_XNAT_SERVER}/data/projects/{PROJECT_NAME}/subjects/{PATIENT_NAME}'
{% endhighlight %}

Pipeline:
{% highlight xml %}
<step id="CREATE_SUBJECT" description="Create a new subject">
    <resource name="XnatDataClient" location="xnat_tools">
        <argument id="user">
            <value>^/Pipeline/parameters/parameter[name='user']/values/unique/text()^</value>
        </argument>
        <argument id="password">
            <value>^/Pipeline/parameters/parameter[name='pwd']/values/unique/text()^</value>
        </argument>
        <argument id="method">
            <value>PUT</value>
        </argument>
        <argument id="remote">
            <value>^concat(/Pipeline/parameters/parameter[name='resolvedHost']/values/unique/text(),'/data/projects/',/Pipeline/parameters/parameter[name='projectID']/values/unique/text(),'/subjects/',/Pipeline/parameters/parameter[name='newPatientID']/values/unique/text())^</value>
        </argument>
    </resource>
</step>
{% endhighlight %}

## Upload data

Upload a ZIP file using curl and cookie
{% highlight bash %}
#!/bin/bash
user=$1
password=$2
host=$3
project=$4
file=$5

cookie=`curl -s -k -u $user:$password -X POST $host/data/JSESSION`

curl -k --cookie JSESSIONID=$cookie --form image_archive=@${file} --form project=$project --form inbody=true --form overwrite=delete $host/data/services/import?format=html
{% endhighlight %}


## Useful links

* [Examples of XDC in XNAT pipelines by John Flavin](https://bitbucket.org/nrg_customizations/)
* [XNAT REST API directory](https://wiki.xnat.org/display/XNAT16/XNAT+REST+API+Directory)
* [XNAT data model](https://wiki.xnat.org/display/XNAT16/Understanding+the+XNAT+Data+Model)
* [XNAT schema data model](https://central.xnat.org/schemas/xnat/xnat.xsd)
* [XNAT REST XML shortcuts](https://wiki.xnat.org/display/XNAT16/XNAT+REST+XML+Path+Shortcuts)
* [Connecting XNAT & DICOM from BIGR group](http://xnat.bigr.nl/index.php/Xnat:Dicom)
