---
author: Vladan
comments: true
date: 2013-06-25
layout: post
slug: a-quick-firefox-startup-update
title: A quick Firefox startup update
wordpress_id: 148
---
Recently I've been working on a [project](https://wiki.mozilla.org/Performance/#Reducing_time_to_first_paint_-_Phase_1) to improve desktop Firefox's startup time during "cold starts" where none of the Firefox binaries or data are cached in memory (e.g. the first launch of the browser after a reboot). I've been paying special attention to the time required to reach the "first paint" startup milestone: the point in time when the first Firefox window becomes visible.

The analysis has mostly consisted of profiling the latest Firefox Nightlies using [XPerf](https://developer.mozilla.org/en-US/docs/Performance/Profiling_with_Xperf) on a reference dual-core Windows 7 laptop with a magnetic HDD. I've been working on several bugs arising from the investigation ([bug 881575](https://bugzilla.mozilla.org/show_bug.cgi?id=881575), [bug 881578](https://bugzilla.mozilla.org/show_bug.cgi?id=881578), [bug 827976](https://bugzilla.mozilla.org/show_bug.cgi?id=827976),  [bug 879957](https://bugzilla.mozilla.org/show_bug.cgi?id=879957), [bug 873640](https://bugzilla.mozilla.org/show_bug.cgi?id=873640)) and I have many more coming. This is an overview of a few challenges I've run into over the last month.

### Making startup times reproducible

I wanted to evaluate the impact of my experimental code changes by comparing startups, but I quickly discovered that there is a tremendous amount of variation in startup times in my test environment. I then turned off [Windows Prefetching](http://en.wikipedia.org/wiki/Prefetcher) & [Windows SuperFetch](http://www.tomshardware.com/reviews/windows-vista-superfetch-and-readyboostanalyzed,1532-2.html), two performance features responsible for pre-fetching files from disk based on the user's usage patterns, but I still recorded excessive variation in start times.

I then turned off a plethora of 3rd-party and Windows services that were running in the background and accessing the disk: Windows Update & Indexing Service, OEM "boot optimizer" software, Flash & Chrome automatic updaters, graphics card configuration & monitoring software bundled with drivers, etc. After rebooting the laptop several times and disabling any remaining programs causing disk activity, I was finally able to achieve reproducible startup times. I expected that cold starts would be dominated by disk I/O, but I was suprised by just how heavily I/O operations dominated startup time in a vanilla Firefox install.

### Startup time has improved almost 30% over the last year

In an attempt to reproduce the startup regression reported in [bug 818257](https://bugzilla.mozilla.org/show_bug.cgi?id=818257), I compared time to first paint for Firefox 13.0.1 and Firefox 21.0 using my test setup. To my surprise, I found Firefox 21.0 (current release channel) requires roughly 4.6 seconds to reach first paint during cold starts, while Firefox 13.0.1 (release channel from a year ago) required ~6.4 seconds! This is almost a 30% reduction in startup time.

[See the raw results from 5 runs in a Google Docs spreadsheet here](https://docs.google.com/spreadsheet/pub?key=0Aq4ElA0owpEbdDhMNjh0V3pDT09Hck82UnZRaWJRTXc&single=true&gid=0&output=html)

I was surprised by this result because I expected increases in code size and the overhead from initializing new components added over the course of a year to cause regressions in startup. On the other hand, many people have landed patches to improve startup by postponing component initialization and generally reducing the amount of work done before the first-paint milestone. I haven't tried to identify the patches responsible yet, but from a quick look at the XPerf profiles for each version, it looks like there were gains from fixing [bug 756313](https://bugzilla.mozilla.org/show_bug.cgi?id=756313)  ("Don't load homepage before first paint") and from changing the list of Mozilla libraries pre-loaded at startup (see _dependentlibs.list_).

### We are still fsync()-ing too much at startup

Apparently, the [FlushFileBuffers](http://msdn.microsoft.com/en-us/library/windows/desktop/aa364439%28v=vs.85%29.aspx) function on Windows causes the OS  to flush everything in the write-back cache as it ["does not know what part of the cache belongs to your file"](http://blogs.msdn.com/b/ce_base/archive/2006/03/15/increasefsthroughput.aspx). As you can imagine, calling FlushFileBuffers is bad news for Firefox startups even it's done off the main thread -- other I/O requests will be delayed while the disk is busy writing data. Unfortunately we are currently calling this method on browser startup to write out the _webapps.json_ file, the _sessionstore.js_ file, and several SafeBrowsing files. The flush method isn't being called directly, rather it's the _SafeFileOutputStream_ and _OS.File.writeAtomic()_ implementations that force flushes for maximum reliability. In general, we should avoid calling methods that fsync/FlushFileBuffers unless such reliability is explicitly required, and I've asked [Yoric](http://dutherenverseauborddelatable.wordpress.com/) to change _OS.File.writeAtomic()_ behavior to forego flushing by default.

### Next steps

I'm continuing to work on reducing the number of DLL loads triggered at startup and I'll soon be filing bugs for fixing some of the smaller sources of startup I/O.
