---
layout: post
title: Deploying Play-Scala application into production
author: ravinder_payal
---
<p>
I checked and searched multiple articles and posts, but nowhere complete information was available. In deploying section of official play-scala documentation, it only talks about how to package in different formats, sbt-native docs are also more focused on packaging. So, I learnt the packaging and ran the packages on a dedicated server, but came across a few tedious problem:</p>
1. The application was closing after a period, and was required to manually reload on the server.
2. On every restart it was required to manually start the application server.
3. Monitoring service health was also not easy, it was required to ssh the server and check the logs and server resource usage.

All three things could be solved only if we turn our application server into system service, and install a monitoring tool like `monit` for easy remote monitoring of server health.

So, after reading a bunch of different articles, and steps at different blogs and documentations, I decided to user `supervisor` and `monit`.

From http://supervisord.org/
> Supervisor: A Process Control System
Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.
It shares some of the same goals of programs like launchd, daemontools, and runit. Unlike some of these programs, it is not meant to be run as a substitute for init as “process id 1”. Instead it is meant to be used to control processes related to a project or a customer, and is meant to start like any other program at boot time.

From wikipedia:
> Monit is a free, open source process supervision tool for Unix and Linux. With Monit, system status can be viewed directly from the command line, or via the native HTTP(S) web server. Monit rose to popularity with Ruby on Rails and the Mongrel web server, because a tool was needed that could manage the many identical Mongrel processes that needed to be run to support a scalable Ruby on Rails site, and Monit was fairly uniquely suited for the needs of the Ruby on Rails community. Many popular Rails sites have used Monit, including Twitter and scribd.

Now the point is how to? So, we start right from packaging and complete the article with application running in production with auto-reload capability on failures(of process running the server) or server death due to run time exceptions or due to memory leakage.

# Packaging play-scala application for Debian/Ubuntu


## Step#1: Minimal settings
Add the following settings to your application's `build.sbt`:

```scala
lazy val root = (project in file("."))
  .enablePlugins(PlayScala, DebianPlugin)

maintainer in Linux := "First Lastname <first.last@example.com>"

packageSummary in Linux := "My custom package summary"

packageDescription := "My longer package description"
```

## Step#2: Packaging

```scala
[your-app] $ debian:packageBin
```

Now, in `project-root/target`we have `your-app.deb` which can be installed using `dpkg` command on debian/ubuntu stack.

## Step#3: Installing the package

```scala
root@machine:../target/#dpkg -i your-app-SNAPSHOT-Something.deb
```

-------------

# Packaging play-scala application for RPM(Redhat party)

## Step#1: Minimal settings
Add the following settings to your application's `build.sbt`:

```scala
lazy val root = (project in file("."))
  .enablePlugins(PlayScala, RpmPlugin)

maintainer in Linux := "First Lastname <first.last@example.com>"

packageSummary in Linux := "My custom package summary"

packageDescription := "My longer package description"

rpmRelease := "1"

rpmVendor := "example.com"

rpmUrl := Some("http://github.com/example/server")

rpmLicense := Some("Apache v2")
```
## Step#2: Packaging

```scala
[your-app] $ rpm:packageBin
```

Now, in `project-root/target`we have `your-app.rpm` which can be installed using `yum` command on redhat/centos/alike stack.

## Step#3: Installing the package

```scala
root@machine:../target/#yum install your-app-SNAPSHOT-Something.deb
```

Now, Installation is done. But, how to make the application-server keep running whenever the host-system is running? Let's find out!

## Step#4: Installing Supervisor

```
apt-get install supervisor
```

or

```
yum install supervisor
```

## Step#5: Installing Monit

```
apt-get install monit
```

or

```
yum install monit
```

## Step#6: Setting up Monit
Edit the file `/etc/monit/monitrc` using your favourite editor and add the following content.

```
check process your-app  with pidfile /usr/share/your-app/RUNNING_PID
 start program = "/usr/bin/supervisorctl start your-app" with timeout 60 seconds
 stop program  = "/usr/bin/supervisorctl stop your-app"
 if 10 restarts within 20 cycles then timeout
 if TOTALMEMORY > 1.4 GB for 4 cycles then restart
 group server
```

And go to following section, and un-comment it for accessing the monit web interface. Yes monit has it's own wev-server

```
#set httpd port 2812 and
#     use address YourServerIp  # Suggestion:only accept connection from localhost, and do ssh tunnelling for better security and preventing unauthorised access
#     allow 0.0.0.0/0.0.0.0        # allow localhost to connect to #the server and
#     allow admin:monit      # require user 'admin' with password 'monit'

```

>You can check the `monit` documentation and alter the basic configuration according to your need

## Step#7: Supervisor configuration
Now go to `/etc/supervisor/conf.d/`, and add a file named `app-name-anything.conf` with following content:

```
$_>cd /etc/supervisor/conf.d/
```

```
[program:app-name]
; remove -Dconfig.file if necessary (play will use the default application.conf)
; change address and port if necessary or remove them to use the default values
;command=java -Xms512M -Xmx1600M -cp .:./lib/*:./target/staged/* -Dplay.http.secret.key=affsfsfsfdosjsjovoeve46e4846vervev4e6r45geger play.core.server.NettyServe$
command=/usr/share/app-name/bin/app-name  -Dplay.http.secret.key=your_secret
; directory to the play framework app, where "app-name/" is the root
directory=/usr/share/app-name/
process_name=app-name
autorestart=true
startsecs=10
stopwaitsecs=10
```

Now, if supervisor is not running run it using `supervisord` command or if already running, it requires to reload the config. So, do this:-

```
$_> supervisorctl update
```

If configuration is saved correctly it should show something similar:

```bash
root@ubuntu-vuc:/etc/supervisor/conf.d# supervisorctl
app-name    RUNNING   pid 1478, uptime 1:21:15
supervisor> 
```


## Step#8: See the whole setup working
Go to the browser and type in the address:- `http://your-server-ip:2812` and enter the username/password entered into the configuration earlier.

# Notes:-
You can use supervisor alone also, if you don't need the advanced features of monit like mail reporting, web interface and all other.
Type in `supervisorctl` in terminal and then enter `help` to see the all things it support. Keeping the app-server on all the time, and reload during failure can be done by supervisor alone.

> One thing I noticed is that monit takes few minutes to put web-interface online, or it might be only with my installation.

Thanks and Cheers Play-Scala devs. Good night!

## Related posts:
1. https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-monit
2. https://www.playframework.com/documentation/2.6.x/Deploying
3. https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-supervisor-on-ubuntu-and-debian-vps
