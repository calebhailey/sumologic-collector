# SumoLogic Collector Docker Images

## Overview

This project contains Dockerfiles to build SumoLogic ["installed collectors"][1]
that can be used as a ["streaming metrics source"][2].

[1]: https://help.sumologic.com/03Send-Data/Installed-Collectors
[2]: https://help.sumologic.com/03Send-Data/Sources/01Sources-for-Installed-Collectors/Streaming-Metrics-Source

## Disclaimer

I'm a major SumoLogic newbie, so everything about this project could be wrong.
In fact, I'm certain I am reinventing wheels here, and as soon as I confirm that
much, I will likely delete the project.

## Why?

I found myself needing an ["installed collector"][1] (SumoLogic Collector) to
configure a SumoLogic ["streaming metrics source"][2] (specifically, to stream
Prometheus metrics via TCP). I had a heck of a time figuring out how to do this
(read: there were dragons). In the end the setup isn't too complicated, but I
fear that future me would have a very hard time reproducing this configuration,
so I'm leaving some breadcrumbs behind. Use this project at your own risk /
YMMV.

## Dependencies

The resulting container has at least one dependency, if not two (that I'm aware
of):

1. SumoLogic Collector `user.properties` config file:

   Contents of `/opt/SumoCollector/config/user.properties`:

   ```
   name=sensu-sumologic-collector
   accessid=sur9PeoJudHkxA
   accesskey=awSUyFgJYCbZx1TICJcP9m8i1ft3dxvsNENXNnCV49RPfINwgwbsVilWdR4cje8S
   syncSources=/etc/sumologic/sources/
   ```

2. SumoLogic Collector JSON config file(s):

   Contents of `/etc/sumologic/sources/*.json`:

   ```json
   {
     "api.version":"v1",
     "sources": [
       {
         "name": "sensu-metrics",
         "description": "Metrics collected by Sensu Go",
         "sourceType": "StreamingMetrics",
         "contentType": "Prometheus",
         "protocol": "TCP",
         "port": 2003
       }
     ]
   }
   ```

   _NOTE: the `syncSources` configuration setting in the `user.properties` file
   (shown above) has configured the SumoLogic Collector to read from one or more
   `.json` config files located in `/etc/sumologic/sources/` (this directory was
   not created by the RPM package, so you may have to create it yourself, or use
   another directory)._

   See ["Local Configuration File Management"][3] and ["Use JSON to configure
   Sources"][4] for more information.

[3]: https://help.sumologic.com/03Send-Data/Sources/03Use-JSON-to-Configure-Sources/Local-Configuration-File-Management
[4]: https://help.sumologic.com/03Send-Data/Sources/03Use-JSON-to-Configure-Sources

## Service Management

All the SumoLogic Collector documentation I could find recommended [running the
collector using a `start` command][5], which runs the collector as a daemon or
background service. This wouldn't work in a containerized environment, so I
needed a solution to run the collector as a non-daemon process, emitting output
to stdout. I found the  SumoLogic Collector service management documentation to
be quite lacking in this regard (it didn't even indicate that the collector
ships with a `help` command where you could find more information), so I'm going
to leave another breadcrumb here in the form of that same help output.

[5]: https://help.sumologic.com/Manage/Collection/02Start-or-Stop-a-Collector-using-Scripts

```
$ /opt/SumoCollector/collector help
Unexpected command: help

Usage: /opt/SumoCollector/collector [ console | start | stop | restart | condrestart | status | install | installstart | remove | dump ]

Commands:
  console      Launch in the current console.
  start        Start in the background as a daemon process.
  stop         Stop if running as a daemon or in another console.
  restart      Stop if running and then start.
  condrestart  Restart only if already running.
  status       Query the current status.
  install      Install to start automatically when system boots.
  installstart Install and start running as a daemon process.
  remove       Uninstall.
  dump         Request a Java thread dump if running.
```

There it is: `/opt/SumoCollector/collector console` is what we need.

## Debugging

To run this container locally, use the following command:

```
docker run --rm -v "$PWD/user.properties:/opt/SumoCollector/config/user.properties" -v "$PWD/streaming-metrics-source.json:/etc/sumologic/sources/streaming-metrics-source.json" -it calebhailey/sumologic-collector:19.253-14 /bin/bash
```
