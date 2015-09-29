---
author: Vladan
comments: true
date: 2015-01-30
layout: post
slug: how-to-evaluate-the-performance-of-your-new-firefox-feature
title: How to evaluate the performance of your new Firefox feature
wordpress_id: 163
---
There are a lot of good tools available now for studying Firefox performance, and I think a lot of them are not well known, so I put together a list of steps to follow when evaluating the performance of your next Firefox feature.

**1. Make sure to test your feature on a low-end or mid-range Windows computer**

* Our dev machines are uncommonly powerful. Think machines with spinning hard drives, not SSDs. Testing on Windows is a must, as it is used by the vast majority of our users.
* The perf team, fx-team, and gfx team have Windows [Asus T100](http://www.amazon.com/Transformer-T100TA-C1-GR-Detachable-Touchscreen-Laptop/dp/B00K0G2XA4) tablets available in multiple offices just for this purpose. Contact me, Gavin, or Milan Sreckovic if you need one.

**2. Ensure your feature does not touch storage on the main thread, either directly or indirectly**

* If there's any chance it might cause main-thread IO, test it with the [Gecko profiler](https://developer.mozilla.org/en-US/docs/Mozilla/Performance/Reporting_a_Performance_Problem). The profiler now has an option to show you all the IO done on the main thread, no matter how brief it is.
* Also be careful about using [SQLite ](https://wiki.mozilla.org/Performance/Avoid_SQLite_In_Your_Next_Firefox_Feature)

**3. Make sure to add Telemetry probes that measure how well your feature performs on real user machines.**

* Check the Telemetry numbers again after your feature reaches the release channel. The release channel has a diversity of configurations that simply don't exist on any of the pre-release channels.
  * You can check for regressions in the [Telemetry dash](https://telemetry.mozilla.org/), or you can ask the perf-team to show you how to do a custom analysis (e.g. performance on a particular gfx card type) using [MapReduce](http://mreid-moz.github.io/blog/2013/11/06/current-state-of-telemetry-analysis/) or [Spark](http://robertovitillo.com/2015/01/16/next-gen-data-analysis-framework-for-telemetry/).
  * The learning curve can be a bit steep, so the perf team can do one-off analyses for you.
  * We have additional performance dashboards; they are listed in the "More Dashboards" sidebar on [telemetry.mozilla.org](https://telemetry.mozilla.org/)
* Always set the "alert_mails" field for your histogram in [Histograms.json](http://mxr.mozilla.org/mozilla-central/source/toolkit/components/telemetry/Histograms.json) so you get automatic e-mail notifications of performance regressions and improvements.
  * Ideally, this email address should point to an alias for your team.
  * Note that the Telemetry [regression detector](http://mozilla.github.io/cerberus/dashboard/) has an extremely low false-positive rate so you won't be getting any emails unless performance has changed significantly.

**4. Keep an eye out on the Talos scores**

  * The Talos tests are much less noisy now than they used to be, and more sensitive as well. This is thanks to Avi Halachmi's, Joel Maher's, and others' efforts.  
  Partly as a result of this, we now have a stricter Talos sheriffing policy. The patch author has 3 business days to respond to a Talos regression bug (before getting backed out), and two weeks to decide what to do with the regression.
  * Joel Maher will file a regression bug against you if you regress a Talos test.
  * The list of unresolved regressions in each release is tracked in the meta bugs: [Firefox 36](https://bugzilla.mozilla.org/show_bug.cgi?id=1084461), [Firefox 37](https://bugzilla.mozilla.org/show_bug.cgi?id=1108235), [Firefox 38](https://bugzilla.mozilla.org/show_bug.cgi?id=1122690), etc
  * Joel tracks all the improvements together with all the regressions in a [dashboard](http://alertmanager.allizom.org:8080/alerts.html?showAll=1)
  * If you cause a regression that you can't reproduce on your own machine, you can capture a profile directly inside the Talos environment:
[https://wiki.mozilla.org/Buildbot/Talos/Profiling](https://wiki.mozilla.org/Buildbot/Talos/Profiling)
    * MattN has an excellent tool for comparing the scores of two Talos pushes: [http://compare-talos.mattn.ca/](http://compare-talos.mattn.ca/)
  * Some Talos tests can be run locally as extensions, others may require you to set up a [Talos harness](https://wiki.mozilla.org/Buildbot/Talos/Running#Running_locally_-_Source_Code). Instructions for doing this will be provided in the Talos regression bugs from now on.
  * The [graph server](http://graphs.mozilla.org/graph.html) can show you a history of test scores and test noise to help you determine if the reported regression is real.
    * William Lachance is working on a new & improved graphing UI for treeherder.

**5. Consider writing a new Talos test**

* [Add a new Talos test](https://wiki.mozilla.org/Buildbot/Talos/Misc#Adding_a_new_test) if the performance of your feature is important and it is not covered by existing tests. The Perf team would be happy to help you design a meaningful and reliable test.
* Make sure your test measures the right things, isn't noisy and that it is is able to detect real regressions

Thanks!

_I initially posted this message for discussion on the [firefox-dev](https://groups.google.com/forum/#!topic/firefox-dev/ZJn00jAkEjM) and [mozilla.dev.platform](https://groups.google.com/d/msg/mozilla.dev.platform/eXnB52BA2A4/wTTkuQRihg4J) newsgroups. This is now also a [wiki page](https://wiki.mozilla.org/Performance/Evaluating_Performance_of_New_Features) in the [Performance wiki](https://wiki.mozilla.org/Performance)._
