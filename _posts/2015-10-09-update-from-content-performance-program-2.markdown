---
author: Vladan
comments: true
date: 2015-10-09
layout: post
title: 'Update from the Content Performance program #2'
---

During Q3, Avi Halachmi, Aaron Klotz and I compared Firefox\'s scrolling & page-loading performance against other Windows browsers on several popular sites using a low-end [HP Pavilion 14t i3-5010u](http://www.amazon.com/HP-13-3-Inch-TouchScreen-Convertible-Processor/dp/B00WVFJO5Y) laptop. This post describes our findings.

### First, a refresher...

In my previous post from [June]({% post_url 2015-06-26-announcing-the-content-performance-program %}), I published our initial Q2 findings:

*  [bug 1213413](https://bugzilla.mozilla.org/show_bug.cgi?id=1213413), [bug 715376](https://bugzilla.mozilla.org/show_bug.cgi?id=715376) related: Process-per-tab e10s is necessary to prevent heavy activity in a background tab (e.g. loading GMail) from affecting scrolling smoothness in a foreground tab
* [bug 1213425](https://bugzilla.mozilla.org/show_bug.cgi?id=1213425): Firefox scrolling smoothness badly deteriorates when the laptop is in power-saver mode
* [bug 1174899](https://bugzilla.mozilla.org/show_bug.cgi?id=1174899): Aaron Klotz found a 100% CPU usage bug while scrolling a Facebook profile containing many HTML5 videos
* Scrolling a Twitter feed with YouTube HTML5 videos is jankier in Firefox, but Twitter newsfeed changed to no longer autoplay videos

The June post also outlined other scenarios needing testing which we studied in Q3.

### Our more recent discoveries

NOTE: All  phase 1 content-perf findings are in [meta bug 1213469](https://bugzilla.mozilla.org/show_bug.cgi?id=1213469), and all content perf bugs we have on file are tagged with [\[content perf\]](https://bugzilla.mozilla.org/buglist.cgi?resolution=---&status_whiteboard=[content%20perf]&status_whiteboard_type=allwordssubstr&list_id=12602314)

* [bug 1213434](https://bugzilla.mozilla.org/show_bug.cgi?id=1213434): Chrome navigates a lot faster to Facebook.com from a Google search result page. The difference is particularly noticeable with a cold network cache. Video showing the differences: [dropbox.com](https://www.dropbox.com/s/l7injr1i6e8u0sx/content_perf3_fb_on_3_browsers.flv?dl=0)
* [bug 1199468](https://bugzilla.mozilla.org/show_bug.cgi?id=1199468): Our smooth-scrolling parameters might not be optimal, but this is still being studied (and debated!)
* We learned that some graphics acceleration configurations actually harm Firefox scrolling performance:
    * [bug 1213429](https://bugzilla.mozilla.org/show_bug.cgi?id=1213429): Scrolling a Facebook profile page with D3D9 acceleration (and D2D disabled) is actually worse than scrolling without any gfx acceleration at all (no D3D + no D2D)
    * [bug 1213432](https://bugzilla.mozilla.org/show_bug.cgi?id=1213432): Similarly, D3D11 Warp acceleration (without D2D) provides worse scrolling performance than no gfx acceleration at all (no D3D + no D2D)
    * Direct3D 11 and Direct2D acceleration (either D2D 1.0 or 1.1) did not produce better scrolling performance on our reference pages
        * [bug 1213440](https://bugzilla.mozilla.org/show_bug.cgi?id=1213440): In particular, on a Yahoo search page with a few image results embedded, scrolling performance and consistency will be worse with D2D 1.1. than D2D 1.0

During our investigations, we also discovered other user-visible issues that did not directly relate to page scrolling & page loading:

* [bug 1213435](https://bugzilla.mozilla.org/show_bug.cgi?id=1213435): Firefox content-process memory usage is significantly worse than Chrome\'s and is lagging IE as well
* [bug 1172205](https://bugzilla.mozilla.org/show_bug.cgi?id=1172206): Firefox\'s page-loading tab throbber spins erratically while loading Amazon.com
* [bug 1213438](https://bugzilla.mozilla.org/show_bug.cgi?id=1213438): Scrolling through a Facebook profile triggers additional network requests, but only Firefox repeatedly changes the tab\'s title from between its real title and "Connecting...". This is distracting and draws unnecessary attention to any network delays

We tested many more use cases since the previous progress update, but we mostly found Firefox\'s performance on par or better:

* Firefox\'s scrolling performance did not regress on Windows 10 as compared to Windows 8 (for the 3 reference sites: Yahoo, Facebook and Twitter)
* Most gfx configurations did not hurt Firefox scrolling performance: external monitor connected, DPI scaling enabled, e10s enabled (comparing non-APZ e10s vs non-e10s), accessibility technology enabled, etc
    * In tests with external monitors, we noticed that Firefox consistently chooses the correct refresh rate, unlike other tested browsers
    * Unfortunately, this finding did not generalize to better gfx performance overall on all multi-monitor setups
* We also found page-loading times on Facebook, Twitter, and Yahoo comparable across browsers
* As an aside, APZC has a noticeably positive impact on scrolling smoothness, but there are issues with checkerboarding and correctness ([bug 1178298]((https://bugzilla.mozilla.org/show_bug.cgi?id=1178298)), so it might be a while before we see APZC riding the trains.

We didn\'t have time to get as far as we wanted to with content-perf tooling in Q3, but Wander Costa did contribute a prototype of a tool for measuring browser responsiveness during page-loads: [https://github.com/walac/page-load-test](https://github.com/walac/page-load-test).

### Content Performance in Q4

The Perf team\'s top priority in Q4 is to [verify e10s performance](https://docs.google.com/document/d/1TyE0BehzYhii3qfmcrfjXlRJL64CcJk0B4Voup4Q0Pg/) using our many measurement systems and generally help get e10s performance ready for release.

As a result, Avi will be the only developer working on content-perf this quarter. However, Avi will be working on content-perf full-time in Q4 and he has already covered additional ground (including Fennec). He will soon be blogging about additional content-perf findings [over at his blog](http://avih.github.io/)
