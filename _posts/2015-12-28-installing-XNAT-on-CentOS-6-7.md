---
layout: post
title:  "Installing XNAT 1.6.5 server on CentOS 6.7"
date:   2015-12-28
tags: [en]
comments: true
---

One of the CAP servers that I maintain is [XNAT server](http://www.xnat.org). The server is running RedHat, so I've been playing XNAT on CentOS 6.7 VM (see my previous [Basic CentOS 6 VM Setup]({% post_url 2015-08-30-create-basic-centos-vm %}) post). There are three steps to make a XNAT server up and running:

1. Installing Java SDK 1.7 (the core engine)
2. Installing PostgreSQL 9.4 (database)
3. Installing Apache Tomcat 7 (web application server)
4. Installing SMTP server (email notification)
5. Installing XNAT 1.6.5 (the application)

<div class="divider"></div>

# Installing Java SDK 1.7

Go to [Oracle's JDK 1.7](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html) website and download:

```
jdk-7u80-linux-x64.rpm
```

Install:
{% highlight console %}
# rpm -ivh jdk-7u80-linux-x64.rpm
# javac -version
javac 1.7.0_80
{% endhighlight %}

The java directory will be in `/usr/java/jdk1.7.0_80`. It is good to create environment variable **JAVA_HOME** pointing that directory for all users. Create `/etc/profile.d/java.sh` and write:
{% highlight bash %}
export JAVA_HOME=/usr/java/jdk1.7.0_80
export JRE_HOME=${JAVA_HOME}/jre
export PATH=${PATH}:${JAVA_HOME}/bin:${JRE_HOME}/bin
{% endhighlight %}
The settings will be loaded during boot. You can set it directly with `source /etc/profile.d/java.sh` command.

<div class="divider"></div>

# Installing Apache Tomcat 7

Download Tomcat 7 from: [http://tomcat.apache.org/download-70.cgi](http://tomcat.apache.org/download-70.cgi) (last version was 7.67).

Unpack it to `/opt/tomcat`.

Create user **tomcat**, fix the file ownership and permission and test the web server:
{% highlight console %}
# useradd -d /opt/tomcat -M -r tomcat
# chmod tomcat:tomcat -R /opt/tomcat
# sudo -u tomcat -i
$ cd /opt/tomcat/bin
$ ./startup.sh
Using CATALINA_BASE:   /opt/tomcat
Using CATALINA_HOME:   /opt/tomcat
Using CATALINA_TMPDIR: /opt/tomcat/temp
Using JRE_HOME:        /usr/java/jdk1.7.0_80/jre
Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
Tomcat started.
{% endhighlight %}

The URL for the web server is: `http://localhost:8080/`

![Tomcat 7](/images/tomcat7-screenshot.png)

> **NOTE**: if you try to open the URL from non localhost address and didn't show any result, you may have to open the firewall of port 8080 with iptables.

We need to configure Tomcat for XNAT and make it run as a service run by user tomcat. First, let us shut it down:
{% highlight console %}
$ cd /opt/tomcat/bin
$ ./shutdown.sh
{% endhighlight %}

Using user **tomcat**, create `/opt/tomcat/bin/setenv.sh` file:
{% highlight bash %}
CATALINA_OPTS="$CATALINA_OPTS -Xms512m -Xmx1024m -XX:MaxPermSize=256m"
{% endhighlight %}
([*see here why*](https://wiki.xnat.org/display/XNAT16/Configuring+Tomcat))

Create init file `/etc/init.d/tomcat`
{% gist lesstif/f96b2ea2b975b8b16d0e %}

Setup the service

```console
# chkconfig --add tomcat
# chkconfig --level 2345 tomcat on
# service tomcat start
```

You can check the web server again using browser.

<div class="divider"></div>

# Installing PostgreSQL 9.4

The latest version of postgresql is not maintained by the default yum repository. We need to make sure that yum does not install from their repository. Edit `/etc/yum.repos.d/CentOS-Base.repo`.

On **[base]** and **[updates]** sections, add the following Line

```
exclude=postgresql*
```

Install PostgreSQL:
{% highlight console %}
# yum localinstall http://yum.postgresql.org/9.4/redhat/rhel-6-x86_64/pgdg-centos94-9.4-1.noarch.rpm
# yum install postgresql94-server
{% endhighlight %}

Initialize database:
{% highlight console %}
# service postgresql-9.4 initdb
# chkconfig --level 2345 postgresql-9.4 on
# service postgresql-9.4 start
{% endhighlight %}

Check:
{% highlight console %}
# sudo -u postgres -i
$ psql
psql (9.4.5)
Type "help" for help

postgres=#
{% endhighlight %}

Fix the authorization mechanism. This is required by XNAT to allow java ODBC driver to connect to the database. Using user **postgres**, edit `/var/lib/pgsql/9.4/data/pg_hba.conf`.

Change local's method from `peer` into `trust`, and `indent` into `md5`.

Restart postgres:

```
# service postgresql-9.4 restart
```

Create a user for XNAT database.
{% highlight console %}
# createuser -U postgres -S -D -R -P xnatuser
{% endhighlight %}

Crete database for XNAT
{% highlight console %}
# createdb -U postgres -O xnatuser xnat
{% endhighlight %}

<div class="divider"></div>

# Installing SMTP server

This is not a required part of XNAT installation but I've found it necessary because XNAT makes use emailing very often, sometimes in an important notification about a process. So I'd recommend to set it up correctly before building the XNAT webapp.

{% highlight console %}
# yum install sendmail sendmail-cf m4
# hostname >> /etc/mail/relay-domain
# m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf
# chkconfig --add sendmail
# chkconfig --level 2345 sendmail on
# service sendmail start
{% endhighlight %}

The SMTP server uses port 25. To check whether this port is open, you can use telnet:
{% highlight console %}
# telnet localhost 25
{% endhighlight %}

Test sending an email, first create email.txt file that contains:
{% highlight text %}
Subject: test email

Hello SMTP.
{% endhighlight %}

Send it
{% highlight console %}
# sendmail -v your@email.address.com << email.txt
{% endhighlight %}

If all went well, then you can use mail `server=localhost`, `protocol=SMTP` and `port=25`.


<div class="divider"></div>

# Installing XNAT 1.6.5

Download XNAT 1.6.5 from [http://xnat.org/download/](http://xnat.org/download/)

Untar it to `/opt/xnat` and change ownership/permission to user **tomcat**.
{% highlight console %}
# cd /opt
# tar -xvpzf xnat-1.6.5.tar.gz
# chown tomcat:tomcat -R /opt/xnat
{% endhighlight %}

Copy `build.properties.sample` file into `build.properties` and set all its properties accordingly. This is my settings:

{% gist avansp/b08f3bcdafcf1932860e %}

Note that in the `build.properties` settings, there are locations for file in `/opt/xnat-data/`, so you need to create these directories that are accessible by user **tomcat**.

Run the build script.

{% highlight console %}
# sudo -u tomcat -i
$ cd /opt/xnat
$ ./bin/setup.sh
{% endhighlight %}

When everything is fine, create file `/etc/profile.d/xnat.sh`:
{% highlight bash %}
export XNAT_HOME=/opt/xnat
export PATH=${PATH}:/${XNAT_HOME}/bin
{% endhighlight %}

Setup the database:

{% highlight console %}
# sudo -u tomcat -i
$ cd /opt/xnat/deployments/xnat
$ psql -d xnat -f sql/xnat.sql -U xnatuser
$ StoreXML -l security/security.xml -allowDataDeletion true
$ StoreXML -dir ./work/field_groups -u admin -p admin -allowDataDeletion true
{% endhighlight %}

Deploy XNAT
{% highlight console %}
$ cd /opt/xnat
$ ./bin/update.sh -Ddeploy=true
{% endhighlight %}

Depending on the url setting in the build.properties file, you should have this screenshot:

![XNAT 1.6.5](/images/xnat-screenshot.png)
