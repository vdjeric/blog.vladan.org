---
author: Vladan
comments: true
date: 2012-05-14
layout: post
slug: more-telemetry-from-super-slow-startups
title: More Telemetry from Super-Slow Startups
wordpress_id: 49
---
Last week, I revisited the issue of super-slow startups by poring over [another batch](https://metrics.mozilla.com/projects/browse/METRICS-429) of Telemetry data from February/March of this year. There were 2263 Telemetry submissions matching the following criteria:

* uptime < 5minutes
* firstpaint > 30 seconds
* uninterrupted startup
* Windows OS
* containing "appUpdateChannel" field (Firefox 11+)

I further limited my investigation to submissions with fewer than 5 addons, bringing the total down to 1209 such Telemetry submissions -- a decidedly small number compared to the number of daily Telemetry submissions over a 2 month period. I was motivated to take another look at super-slow startup reports because Telemetry now reports Firefox startup metrics separately: select performance histograms from startup are now reported separately, as are slow SQL statements from startup.

As in the [previous analysis]({% post_url 2012-01-31-telemetry-from-super-slow-startups-first-impressions %}), part of the startup woes seen in these reports can be blamed on Firefox occasionally trying to restore the previous session before painting the UI. There is a fix available in [bug 715402](https://bugzilla.mozilla.org/show_bug.cgi?id=715402) but it hasn't landed yet. However, my overall impression is that these super-slow startups are primarily caused by very slow computers or computer resources temporarily being in very short supply while Firefox is starting. As such, we may not have many options for helping these users. This is a fairly typical timeline of a very slow startup:

14.406s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; main  
51.110s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; createTopLevelWindow  
62.344s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; firstLoadURI  
62.543s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; delayedStartupStarted  
62.547s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; firstPaint  
62.584s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sessionRestoreInitialized  
62.614s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sessionRestoreRestoring  
62.684s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; delayedStartupFinished  
62.688s &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sessionRestored  

It takes 14 seconds just to reach Firefox's main and then another 37 seconds to create the top level window, strongly suggesting an over-loaded system. It would be nice if we could confirm this through Telemetry, for example by gathering Firefox page fault stats but it seems that's [pretty tricky](http://glandium.org/blog/?p=1963) to do on Windows. Another very common pattern in the reports (also seen in the sequence above) is that the first real URI (not about:blank) most often starts to load before first paint is complete (filed [bug 756313](https://bugzilla.mozilla.org/show_bug.cgi?id=756313)).

Slow SQL statements and other Firefox I/O activities definitely do contribute to the super slow startup times but, as before, their totals are roughly an order of magnitude smaller than the total time required for startup. In particular, STARTUP\_NETWORK\_DISK\_CACHE\_OPEN and STARTUP\_CACHE\_DEVICE\_SEARCH frequently appear in super-slow startup reports, contributing several seconds each to startup times. Additionally, MOZ\_SQLITE\_**COOKIES\_READ**\_MAIN\_THREAD\_MS, MOZ\_SQLITE\_**OTHER\_READ**\_MAIN\_THREAD\_MS, MOZ\_SQLITE\_**OTHER\_SYNC**\_MAIN\_THREAD\_MS, MOZ\_SQLITE\_**OPEN**\_MAIN\_THREAD\_MS also very commonly report multi-second values. Surprisingly, it also seems very common in these reports for Firefox to read over a megabyte from the cookies.sqlite database during startup alone. This flurry of activity likely comes from the first URI (the homepage?) being loaded and the previous session being restored.

Interestingly, the addons system also seems to be a source of slow main-thread SQL during startup:

* _RELEASE SAVEPOINT 'default'_
* _INSERT INTO locale (name, description, creator, homepageURL) VALUES (:name, :description, :creator, :homepageURL)_
* _DELETE FROM addon WHERE internal\_id=:internal\_id_

I believe these SQL statements are associated with addon updates during startup. There may be an opportunity to improve performance by moving more of these operations to a background thread.

Finally, there also appears to be lock contention between the main thread and the async thread on cookies.sqlite DB accesses, e.g. the following statements:

<blockquote>
SELECT name, value, host, path, expiry, lastAccessed, creationTime, isSecure, isHttpOnly FROM moz_cookies WHERE baseDomain = :baseDomain
<br/><br/>
SELECT name, value, host, path, expiry, lastAccessed, creationTime, isSecure, isHttpOnly, baseDomain FROM moz_cookies WHERE baseDomain NOTNULL
</blockquote>

This analysis was a lot quicker than the last since I had scripts for sifting through the data and generating the reports. The next step will be to follow up on some of these issues (homepage load before first paint, lock contention, synchronous SQL). Ideally, it will eventually be possible to automate analysis of Telemetry data and have scripts automatically flag troublesome SQL queries or regressions in reported startup times.
