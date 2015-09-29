---
author: Vladan
comments: true
date: 2013-04-24
layout: post
slug: a-quick-update-on-snappy-progress
title: A Quick Update on Snappy Progress
wordpress_id: 125
---
The Performance team has [retired the Snappy name](http://taras.glek.net/blog/2013/03/27/snappy-number-55-snappy-evolution/), and the individual project leads are now blogging their projects' progress instead of Taras doing regular Snappy blog posts.

However, I think there might still be some interest in seeing the performance improvements summarized in one place, and since I'm already doing Performance team updates at the [Platform meeting](https://wiki.mozilla.org/Platform/), I thought I would try my hand at regularly blogging about ongoing performance work.

So without further ado, these are a few highlights from the past 2 weeks:

* [bug 859558](https://bugzilla.mozilla.org/show_bug.cgi?id=859558): John Daggett is working on eliminating font jank. The font bugs currently tracked in this meta-bug are top offenders according to [chrome-hang reports]({% post_url 2013-04-09-current-state-of-firefox-chrome-hangs %}).
* Honza Bambas wrote a [draft proposal](https://wiki.mozilla.org/Necko/Cache/Plans/Draft_Proposal) for a new network cache design. The locking in the current network cache is a common source of Firefox jank.
* [bug 830492](https://bugzilla.mozilla.org/show_bug.cgi?id=830492): Gregory Szorc changed SQLite behavior in FHR to require fewer fsyncs
* Kyle Lahnakoski developed a [tool](https://metrics.mozilla.com/bugzilla-analysis/Telemetry%20-%20Test.html) for comparing Telemetry simple measures. This tool is in the prototype stage and is currently only being used to look for correlations between slow startups and other Telemetry variables (more on that in another blog post). Since Kyle & I are currently the only users of this tool, the page is only accessible from Mozilla's Toronto network. You will also have to disable [mixed-content protection](https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/) on the page.

A recent bug affected Telemetry submission rates on Firefox 21, 22, and 23 for several weeks. It has since been resolved ([bug 844331](https://bugzilla.mozilla.org/show_bug.cgi?id=844331) and [bug 862599](https://bugzilla.mozilla.org/show_bug.cgi?id=862599)), but you'll need to exercise caution when interpreting dashboard results from the affected time period. Specifically, you may want to exclude data from time periods with relatively few Telemetry submission counts.

Finally, there were several blog posts from the individual project leads:

* [David Teller announced project "Async & Responsive"](http://dutherenverseauborddelatable.wordpress.com/2013/04/10/announcing-project-async-responsive/)
* [Avi Halachmi blogged about the "Tabstrip Animation Project"](http://avih.github.io/blog/2013/04/14/tabstrip-project-intro/) and [his recent progress](http://avih.github.io/blog/2013/04/15/tabstrip-animation-progress-vsync/)
* [Irving Reid is changing the Addon Manager's storage format](http://www.controlledflight.ca/)
* [Nathan Froyd looked at Talos pageload benchmark data](https://blog.mozilla.org/nfroyd/2013/04/18/cold-page-load-os-x-10-7-and-talos/)
* [Julian Seward published an update on native stack unwinding](https://blog.mozilla.org/jseward/)

