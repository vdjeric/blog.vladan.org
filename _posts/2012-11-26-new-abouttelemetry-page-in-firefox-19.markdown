---
author: Vladan
comments: true
date: 2012-11-26
layout: post
slug: new-abouttelemetry-page-in-firefox-19
title: New about:telemetry page in Firefox 19
wordpress_id: 81
---
Firefox 19 now has a built in "about:telemetry" page which displays the data gathered by the "Telemetry" system. If the user has opted in, Telemetry will gather information about Firefox performance, hardware, usage and customizations and submit it to Mozilla. The information in the about:telemetry page is meant to inform users about the data being sent to Mozilla, but it can also help technical users report performance bugs to Mozilla.

For example, the "Slow SQL Statements" section identifies slow SQLite operations which have caused Firefox "jank" (temporary hangs):

[![about:telemetry "Slow SQL Statements" section]({{ site.url }}/assets/aboutTelemetrySQL-300x251.png)]({{ site.url }}/assets/aboutTelemetrySQL.png)

In Nightly builds which support profiling, the "Browser Hangs" section will show call stacks from the main thread whenever it took longer than 5 seconds to process an event. Clicking on "Fetch function names for hang stacks" converts the stack PCs to human-readable function names:

[![about:telemetry "Browser Hangs" section]({{ site.url }}/assets/aboutTelemetryHangs-300x251.png)]({{ site.url }}/assets/aboutTelemetryHangs.png)

The Histograms sections contains various performance and usage measures expressed in histogram form. The histogram data is primarily intended for Firefox developers, but the brave can look up the histogram definitions in [Histograms.json](http://mxr.mozilla.org/mozilla-central/source/toolkit/components/telemetry/Histograms.json) and compare their own values to the Firefox population using the [Telemetry Dashboard](https://metrics.mozilla.com/data/).

[![about:telemetry "Histograms" section]({{ site.url }}/assets/aboutTelemetryHistograms-300x254.png)]({{ site.url }}/assets/aboutTelemetryHistograms.png)

I invite you to leave a comment or file a bug if there are any changes you would like to see made to the about:telemetry page to make it more useful for you.

Please note that the old [about:telemetry](https://addons.mozilla.org/en-us/firefox/addon/abouttelemetry/) extension has been obsoleted by the built-in page and that the extension is now marked incompatible with Firefox 19+.
