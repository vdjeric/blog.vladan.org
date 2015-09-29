---
author: Vladan
comments: true
date: 2013-06-11
layout: post
slug: performance-update-issue-3
title: 'Performance Update, Issue #3'
wordpress_id: 145
---
_This is a post summarizing the activities of the Performance team over the past week. You can see the full weekly Engineering progress report for all teams here: [https://wiki.mozilla.org/Platform/2013-06-11](https://wiki.mozilla.org/Platform/2013-06-11)._

* Mark Reid joined the Perf team. He will be working on the Telemetry backend reboot
* Dhaval Giani joined the Perf team as a summer intern. Dhaval is a Master's student at the University of Toronto where he works on detecting bugs in applications of RCU locking. Dhaval's first internship project is storing Firefox caches in volatile (purgeable) memory on Android & B2G ([bug 748598](https://bugzilla.mozilla.org/show_bug.cgi?id=748598)?)
* [bug 867757](https://bugzilla.mozilla.org/show_bug.cgi?id=867757), [bug 867762](https://bugzilla.mozilla.org/show_bug.cgi?id=867762): Aaron Klotz is extending the Gecko Profiler to support arbitrary annotations
* [bug 881578](https://bugzilla.mozilla.org/show_bug.cgi?id=881578), [bug 881575](https://bugzilla.mozilla.org/show_bug.cgi?id=881575), [bug 879957](https://bugzilla.mozilla.org/show_bug.cgi?id=879957): I wrote a few small improvements to reduce startup I/O
* [bug 880296](https://bugzilla.mozilla.org/show_bug.cgi?id=880296): We need to load fewer DLLs on startup
* Nathan Froyd is looking into improving Firefox startup on Android
* [bug 813742](https://bugzilla.mozilla.org/show_bug.cgi?id=813742): Nathan is also working on parallelizing the reftest and crashtest suites
* [bug 872421](https://bugzilla.mozilla.org/show_bug.cgi?id=872421), [bug 880664](https://bugzilla.mozilla.org/show_bug.cgi?id=880664): Yoric landed a module loader for chrome workers
* [bug 853388](https://bugzilla.mozilla.org/show_bug.cgi?id=853388): Irving & Felipe continue to work on converting Addon Manager storage from SQLite to JSON, and moving its I/O off the main thread

The team blogged about their work:

* **Avi**: [http://avih.github.io/blog/2013/06/10/tabstrip-4-vsync-and-newtab/](http://avih.github.io/blog/2013/06/10/tabstrip-4-vsync-and-newtab/)
* **Yoric**: [http://dutherenverseauborddelatable.wordpress.com/2013/06/05/project-async-responsive-issue-3/](http://dutherenverseauborddelatable.wordpress.com/2013/06/05/project-async-responsive-issue-3/)
* **Irving**: [http://www.controlledflight.ca/2013/05/31/saving-browser-state-asynchronously/](http://www.controlledflight.ca/2013/05/31/saving-browser-state-asynchronously/)
* **Julian Seward**: [https://blog.mozilla.org/jseward/2013/06/03/profiler-backend-news-3-june-2013/](https://blog.mozilla.org/jseward/2013/06/03/profiler-backend-news-3-june-2013/)

