---
author: Vladan
comments: true
date: 2013-05-02
layout: post
slug: running-nightly-23-from-an-sd-card
title: Running Nightly 23 from an SD card
wordpress_id: 130
---
I've noticed that most Mozilla developers are using recent laptops with fast SSDs for their work. This observation shouldn't be surprising, as developers tend to be tech enthusiasts with higher requirements, but I wonder if these fast machines could also be masking some of the performance problems in the code we write?

We've known for a while that I/O operations on the main thread are a major source of Firefox jank, but I think we sometimes under-estimate the urgency of refactoring the remaining sources of main-thread I/O. While Firefox might feel fast on powerful hardware, I believe a significant share of our users are still using relatively old hardware. For example [Firefox Health Report data](https://groups.google.com/forum/?fromgroups=#!topic/mozilla.dev.planning/z6UB6fnmhKg) shows that 73.5% of our more-technically-inclined Beta users (Release channel data not yet available) are on a computer with 1 or 2 cores, including hyper-threaded cores.

Even when users are on modern machines, their storage systems might be slow because of hardware problems, I/O contention, data fragmentation, Firefox profiles/binaries being accessed over a network share, power-saving settings, slow laptop hard-drives, and so on. I think it would be beneficial to test certain types of code changes against slow storage by running Firefox from a network share, an SD card, or some other consistently slow I/O device.

### The Experiment

As an experiment to see what it's like to run Firefox when I/O is consistently slow, I decided to create a Firefox profile on an SD card and surfed the web for a couple of hours, capturing profiles of jank along the way. I used an SD card which advertised maximum transfer rates of 20MB/s for  reads and 15MB/s for writes, although in practice, large transfers to and from the card peaked around 10MB/s. For reference, my mechanical hard drive performs the same operations an order of magnitude faster. I left the Firefox binaries on my hard drive -- I was more interested in the impact of I/O operations that could be re-factored than the impact of Firefox code size.

As I visited pages to set up the new Firefox profile with my usual customizations (extensions, pinned tabs, etc), I observed regular severe jank at every step. After entering the URL of a new site and hitting Enter, Firefox would become unresponsive and Windows would mark it as "Not Responding". After about 5 seconds, it would start handling events again. Profiles showed that the network cache (currently being re-designed) was the most common source of these hangs. I also hit noticeable jank from other I/O bottlenecks when I performed common actions such as entering data into forms, logging into sites, downloading files, bookmarking pages, etc. Most of these issues are being worked on already, and I'm hopeful that this experiment will produce very different results in 6 months.

This is a screen recording of a simple 5-minute browsing session. The janks shown in the video are usually absent when the profile is stored on my hard drive instead. You'll need to listen to the audio to get all the details.

<iframe width="560" height="315" src="https://www.youtube.com/embed/XzSQS7sHgI8" frameborder="0" allowfullscreen></iframe>
<br/>

### Observations

The following is a selection of some of the I/O bottlenecks I encountered during my brief & unrepresentative browsing sessions with this profile.

1) Initializing a new profile takes a very long time. After first launch, even with the application binaries in the disk cache, Firefox did not paint anything for 20 seconds. It took an additional 5 seconds to show the "Welcome to Firefox" page. The "Slow SQL" section of about:telemetry showed ~40 SQL statements creating tables, indexes and views, with many taking multiple seconds each. To the best of my knowledge, there hasn't been much research into improving profile-creation time recently. We discussed packing pre-generated databases into omni.jar a year ago ([bug 699615](https://bugzilla.mozilla.org/show_bug.cgi?id=699615)).

2) Installing extensions is janky. The new extension's data is inserted into the addons.sqlite and extensions.sqlite DBs on the main thread. An equal amount of time is spent storing the downloaded XPI in the HTTP cache. See profile [here](http://people.mozilla.com/~bgirard/cleopatra/#report=5b66bc0577f23524bdeedba3c56d2499906b342b). Surprisingly, the URL classifier also calls into the HTTP cache, triggering jank.

3) The AwesomeBar is pretty janky at first. It seems the Places DB gets cloned several times on the main thread.

4) Unsurprisingly, startup & shutdown are slower, especially if the Add-on Manager has to update extension DBs or if extensions need to do extra initialization or cleanup. For example, AdBlock triggers main-thread I/O shortly after startup by loading the AdBlock icon from its XPI into the browser toolbar.

5) New bugs/bugs needing attention:

* The URL classifier opens/writes/closes "urlclassifierkey3.txt" on the main thread. Filed [bug 867776](https://bugzilla.mozilla.org/show_bug.cgi?id=867776)
* The cookie DB is closed on the main-thread after being read into memory. Filed [bug 867798](https://bugzilla.mozilla.org/show_bug.cgi?id=867798)
* The startupCache file might be another good candidate for read-ahead. Filed [bug 867804](https://bugzilla.mozilla.org/show_bug.cgi?id=867804)
* [bug 818725](https://bugzilla.mozilla.org/show_bug.cgi?id=818725): localStore.rdf is fsync'ed on the main thread during GC . Not completely new, but this bug could use some attention
* [bug 789945](https://bugzilla.mozilla.org/show_bug.cgi?id=789945): Preferences are flushed & fsynced on the main thread several times during a session and they can cause noticeable jank. They also show up frequently in chrome hangs.
* [bug 833545](https://bugzilla.mozilla.org/show_bug.cgi?id=833545): Telemetry eagerly loads saved pings on the main thread

6) Known issues observed during browsing:

* Network cache creates a ton of jank when storage is slow/contended
* Password Manager, Form History, Places and Download Manager do main-thread I/O
* SQLite DBs cause jank when opened on the main-thread
* Font loading causes jank

