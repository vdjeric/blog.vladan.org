---
author: Vladan
comments: true
date: 2015-06-26
layout: post
slug: announcing-the-content-performance-program
title: Announcing the Content Performance program
wordpress_id: 165
---
### Introduction

[Aaron Klotz](http://dblohm7.ca/), [Avi Halachmi](http://avih.github.io/) and I have been studying Firefox's performance on Android & Windows over the last few weeks as part of an effort to evaluate Firefox "content performance" and find actionable issues. We're analyzing and measuring how well Firefox scrolls pages, loads sites, and navigates between pages. At first, we're focusing on 3 reference sites: Twitter, Facebook, and Yahoo Search.

We're trying to find reproducible, meaningful, and common use cases on popular sites which result in noticeable performance problems or where Firefox performs significantly worse than competitors. These use cases will be broken down into tests or profiles, and shared with platform teams for optimization. This "Content Performance" project is part of a larger organizational effort to improve Firefox quality.

I'll be regularly posting blog posts with our progress here, but you can can also track our efforts on our mailing list and IRC channel:

**Mailing list:** [https://mail.mozilla.org/listinfo/contentperf](https://mail.mozilla.org/listinfo/contentperf)  
**IRC channel:** #contentperf  
**Project wiki page:** [Content_Performance_Program](https://wiki.mozilla.org/Firefox/Content_Performance_Program)

### Summary of current findings (June 18)

Generally speaking, desktop and mobile Firefox scroll as well as other browsers on reference sites when there is only a single tab loaded in a single window.

* We compared Firefox vs Chrome and IE:
  * Desktop Firefox scrolling can badly deteriorate when the machine is in power-saver mode
  <sup class="note">[1](#note1)</sup> (Firefox performance relative to other browsers depends on the site)
  * Heavy activity in background tabs badly affects desktop Firefox's scrolling performance
  <sup class="note">[1](#note1)</sup>
  (much worse than other browsers -- we need E10S)
  * Scrolling on infinitely-scrolling pages only _appears_ janky when the page is waiting on additional data to be fetched
* Inter-page navigation in Firefox can exhibit flicker, similar to other browsers
* The Firefox UI locks up during page loading, unlike other browsers (need E10S)
* Scrolling in desktop E10S (with heavy background tab activity) is only as good as the other browsersn1 when Firefox is in the process-per-tab configuration (dom.ipc.processCount >> 1)

<sup class="note"><a name="note1">1</a></sup>
You can see Aaron's scrolling measurements here: [http://bit.ly/1K1ktf2](http://bit.ly/1K1ktf2)

#### Potential scenarios to test next:

* Check impact of different Firefox configurations on scrolling smoothness:
  * Hardware acceleration disabled
  * Accessibility enabled & disabled
  * Maybe: Multiple monitors with different refresh rate (test separately on Win 8 and Win 10)
  * Maybe: OMTC, D2D, DWrite, display & font scaling enabled vs disabled
    * If we had a Telemetry measurement of scroll performance, it would be easier to determine relevant characteristics
* Compare Firefox scrolling & page performance on Windows 8 vs Windows 10
  * Compare Firefox vs Edge on Win 10
* Test other sites in Alexa top 20 and during random browsing
* Test the various scroll methods on reference sites (Avi has done some of this already): mouse wheel, mouse drag, arrow key, page down, touch screen swipe and drag, touchpad drag, touchpad two finger swipe, trackpoints (special casing for ThinkPads should be re-evaluated).
  * Check impact of pointing device drivers
* Check performance inside Google web apps (Search, Maps, Docs, Sheets)
  * Examine benefits of Chrome's network pre-fetcher on Google properties (e.g. Google search)
  * Browse and scroll simple pages when top Google apps are loaded in pinned tabs
* Compare Firefox page-load & page navigation performance on HTTP/2 sites (Facebook & Twitter, others?)
* Check whether our cache and pre-connector benefit perceived performance, compare vs competition

#### Issues to report to Platform teams

* Worse Firefox scrolling performance with laptop in power-save mode
* Scrolling Twitter feed with YouTube HTML5 videos is jankier in Firefox
* [bug 1174899](https://bugzilla.mozilla.org/show_bug.cgi?id=1174899): Scrolling on Facebook profile with many HTML5 videos eventually causes 100% CPU usage on a Necko thread + heavy CPU usage on main thread + the page stops loading additional posts (videos)

#### Tooling questions:

* Find a way to to measure when the page is "settled down" after loading, i.e. time until last page-loading event. This could be measured by the page itself (similar to Octane), which would allow us to compare different browsers
* How to reproduce dynamic websites offline?
* Easiest way to record demos of bad Firefox & Fennec performance vs other browsers?

#### Decisions made so far:

* Exclusively focus on Android 5.0+ and Windows 7, 8.1 & 10
* Devote the most attention to single-process Nightly on desktop, but do some checks of E10S performance as well
* Desktop APZC and network pre-fetcher are a long time away, don't wait

