---
author: Vladan
comments: true
date: 2013-05-07
layout: post
slug: progress-on-performance-issue-2
title: 'Performance Update, Issue #2'
wordpress_id: 140
---
_This is my regularly scheduled post summarizing performance work from the past two weeks. Alternate title: Vlad's Big Bowl of Performance Chilli_

The Performance team had its first monthly status meeting. We decided on projects and set goals & timelines: [wiki](https://wiki.mozilla.org/Performance#Projects). The next meeting is on Thursday, June 6th @ 11am PDT (Vidyo room "Performance"), people from other teams who are working on related projects will be invited.

Main thread I/O continues to be a major source of Firefox jank. To illustrate this point, I ran Nightly 23 with its profile stored on an SD card and captured a [screen recording](http://www.youtube.com/watch?v=XzSQS7sHgI8). The results were not pretty, as Firefox hung repeatedly during common actions (see [blog post]({% post_url 2013-05-02-running-nightly-23-from-an-sd-card %})). Patrick McManus posted a band-aid patch ([bug 868441](https://bugzilla.mozilla.org/show_bug.cgi?id=868441)) that will allow Firefox to by-pass the network cache when locking in the network cache is taking too long. The long-term solution is a [network cache re-design](https://wiki.mozilla.org/Necko/Cache/Plans/Draft_Proposal). Aaron Klotz and Joel Maher are working on detecting when new sources of main-thread I/O are added to the code in our test environment ([bug 644744](https://bugzilla.mozilla.org/show_bug.cgi?id=644744)).

Drew Willcoxon wrote a patch to capture page thumbnails (for about:home) in the background ([bug 841495](https://bugzilla.mozilla.org/show_bug.cgi?id=841495)). Once it's hooked up, this will move the thumbnailing operation off the main thread and will allow Firefox to take snapshots of sites loaded without cookies to avoid capturing sensitive data.

Other fixes:

* [bug 852467](https://bugzilla.mozilla.org/show_bug.cgi?id=852467): nsDisableOldMaxSmartSizePrefEvent runs on the gecko main thread, blocks for long periods of time
* [bug 649216](https://bugzilla.mozilla.org/show_bug.cgi?id=649216): Remove unnecessary delay when clicking tab close buttons sequentially
* [bug 699331](https://bugzilla.mozilla.org/show_bug.cgi?id=699331): Reduce impact of font name enumeration at startup

The team blogged about progress on their longer-term projects:

* [Julian Seward: Profiler backend news](http://blog.mozilla.org/jseward/2013/04/30/profiler-backend-news-30-april-2013/)
* [Avi Halchmi: Tabstrip Animation Progress](http://avih.github.io/blog/2013/05/06/tabstrip-animation-number-3/)
* [Irving Reid: Add-On Manager Progress](http://www.controlledflight.ca/2013/05/03/add-on-manager-progress/)
* [David Teller: Project "Async & Responsive" Progress](http://dutherenverseauborddelatable.wordpress.com/2013/04/26/project-async-responsive-issue-1/)


