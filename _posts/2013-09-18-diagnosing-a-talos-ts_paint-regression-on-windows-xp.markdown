---
author: Vladan
comments: true
date: 2013-09-18
layout: post
slug: diagnosing-a-talos-ts_paint-regression-on-windows-xp
title: Diagnosing a Talos ts_paint regression on Windows XP
wordpress_id: 157
---
Recently, I worked on diagnosing a [3% startup time regression](https://bugzilla.mozilla.org/show_bug.cgi?id=880611) in the [Talos ts_paint](https://wiki.mozilla.org/Buildbot/Talos/Tests#ts_paint) benchmark and I thought I'd share my experience with others who might be dealing with a similar regression.

### Procedure

My first step was to reproduce the regression on a Windows XP machine under my control. To prep the machine, I turned off unneeded [background services](http://www.netsquirrel.com/msconfig/) to prevent them from interfering with measurements, created new Firefox profiles, configured Firefox to stop checking for updates and to use a blank page as its homepage, disabled extensions (that had been installed system-wide), and launched Firefox several times so that any data needed from disk during startup would be cached (ts_paint measures "hot" startups). I also wrote a batch script to automate the launching and shutting down of Firefox so that I could easily collect data from dozens of startups.

Finally, I turned on Telemetry data gathering since Telemetry records timestamps for each startup phase. During Firefox exit, Telemetry writes the collected data as a JSON file in **\<profile directory\>\saved-telemetry-pings\,** so every time I launched & shut down Firefox, it would spit out a new JSON file containing the startup timings. You can see a list of the startup times collected by Telemetry in the "Simple Measurements" section of your Firefox's about:telemetry page.

I then wrote a simple Python script to extract and format the startup data from the Telemetry JSON file. This is [the script](https://github.com/vdjeric/tools) and these are [the results in a Google Docs spreadsheet](https://docs.google.com/spreadsheet/ccc?key=0AvVupx3cwu4edEtIeXhCTUZJbEFERXQwS1NfWEVhdnc&usp=sharing). The regression is highlighted in red in the spreadsheet. The times show surprisingly little variation between runs and suggest that the regression is entirely contained in the startup phase directly preceding the first paint of a Firefox window. Luckily, the Gecko profiler has already been initialized at this point of startup so it was possible to capture profiles of the browser's activity. [Matt Noorenberghe](http://matthew.noorenberghe.com/blog) and [Mike Conley](http://mikeconley.ca/blog/) were able to show that at least part of this regression is caused by initialization of the new "Customize UI" functionality and painting of tabs inside the titlebar ([bug 910283](https://bugzilla.mozilla.org/show_bug.cgi?id=910283)).

### Problems profiling on Windows XP

I first tried profiling the startup regression with XPerf and I soon discovered that setting up XPerf on Windows XP s a surprisingly cumbersome task. I had to install XPerf from the [Microsoft Windows SDK for Windows Server 2008](http://www.microsoft.com/en-us/download/details.aspx?id=24826) on a different computer running a more modern, 32-bit version of Windows (XP wouldn't work, I used Vista), and then I manually copied xperf.exe onto the Windows XP machine. Unfortunately, I found that [it's not possible to profile with stackwalking enabled on XP](http://blogs.msdn.com/b/pigscanfly/archive/2009/08/06/stack-walking-in-xperf.aspx) and also that the captured profiles can't be examined on an XP machine, requiring yet more copying to a newer Windows machine.

I also found that the pseudo-stack and native stack frames were not being interleaved properly in the Gecko Profiler on Windows XP ([bug 900524](https://bugzilla.mozilla.org/show_bug.cgi?id=900524)). This was particularly irksome since it meant the JS and C++ stacks would not be merged correctly. It turned out that the version of StackWalk64 that shipped with dbghelp.dll in Windows XP was not walking the data stack properly and that replacing it with a newer version from [Debugging Tools for Windows](http://msdn.microsoft.com/en-us/windows/hardware/gg463009.aspx) resolved the problem.
