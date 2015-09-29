---
author: Vladan
comments: true
date: 2013-04-09
layout: post
slug: current-state-of-firefox-chrome-hangs
title: Current state of Firefox chrome hangs
wordpress_id: 119
---
This post summarizes the top "chrome hangs" reported to Telemetry by Nightly 22 on Windows during the first half of March.

A "chrome hang" is a period of time during which the main thread is stuck processing a single event. It is not a permanent hang. By default, Nightly on Windows reports any chrome hangs lasting at least 5 seconds to Telemetry. You can see your own chrome hangs from your current browsing session in your Nightly's _about:telemetry_ page.

### Data set

There are roughly 270,000 Firefox sessions with chrome hangs in this data set, reporting a total of ~570,000 chrome hang stacks. There are 84,000 "unique" stack signatures, but the heuristic I use for stack signature generation is far from perfect.

The top hangs:

1. [Top 100 stack signatures]({{ site.url }}/assets/Top100.txt)
2. [Top 50 signatures, excluding plugin hangs]({{ site.url }}/assets/Top50_Without_PluginHangs.txt)<span class="emphasize">*</span>
3. [Top 50 signatures, excluding plugin, GC, CC, HTTP cache, font, or content hangs]({{ site.url }}/assets/Top50_Without_CommonHangTypes.txt)<span class="emphasize">*</span>

<span class="emphasize">*</span> _I removed JS-only stacks that did not contain any useful identifying frames_

### Top causes of chrome hangs

The raw chrome hang data is challenging to categorize perfectly since there is a very long tail of stack signatures.

Instead, I used a simple heuristic to categorize the stacks, and I found that hang stacks involving plugins are by far the most common (36% of hangs), stacks with font operations are second (12%), GC and CC are third (10%), and the HTTP cache is fourth (4%). The long-tail of "other" operations makes up the remaining 38% of the data.

After filtering the data set to only the top 100 most common hang signatures, I found the distribution of hangs mostly unchanged. Most of the hangs are again caused by plugins: 53 out of the top 100 signatures are plugins (67% by number of hangs). Font-loading operations take second place (11% by number of hangs), followed by GC and CC (9%) and locking in the HTTP cache (4%). The remaining 9% of hangs is mostly made up of long-running JavaScript scripts. Unfortunately, the chrome hang reporting code currently does not  walk the JavaScript stack, so it's impossible to obtain useful information out of pure JavaScript hangs.

**Data:** [Top 100 chrome hang stack signatures]({{ site.url }}/assets/Top100.txt)

### The plugin problem

We've known for a while that plugins are a major source of browser unresponsiveness. In particular, the initial synchronous load of the plugin's library and the creation of a new plugin instance are the most common hang stacks. The top 100 list also shows plugins taking a long time to handle events and destroy plugin instances. There is also a large number of stacks where Flash hangs cause Firefox hangs on account of Windows input queue synchronization ([bug 818059](https://bugzilla.mozilla.org/show_bug.cgi?id=818059)).

Since we're stuck with the synchronous NPAPI, I suspect we won't be able to minimize the impact of of plugin hangs until we separate content and chrome into separate processes (i.e. the Electrolysis project). In the short term, we're mitigating some of the plugin pains by adding read-ahead to improve plugin library loading speed ([bug 836488](https://bugzilla.mozilla.org/show_bug.cgi?id=836488)) and starting with Firefox 20, we're showing users a UI that allows them to terminate an unresponsive plugin after 11 seconds ([bug 805591](https://bugzilla.mozilla.org/show_bug.cgi?id=805591)). Both of these patches were written by Aaron Klotz.

Benoit Girard, Georg Fritzsche and Benjamin Smedberg are working on uncovering the causes of some of these hangs by adding profiling support to the plugin-container.exe process ([bug 853358](https://bugzilla.mozilla.org/show_bug.cgi?id=853358) and [bug 734691](https://bugzilla.mozilla.org/show_bug.cgi?id=734691)) and exposing IPC message information to the profiler ([bug 853864](https://bugzilla.mozilla.org/show_bug.cgi?id=853864) and [bug 853363](https://bugzilla.mozilla.org/show_bug.cgi?id=853363)). You can find Benoit's write-up [here](http://benoitgirard.wordpress.com/2013/03/25/profiler-snappy-work-week/).

Finally, it might also be possible to move some of the heavy plugin operations off the main thread (e.g. [bug 856743](https://bugzilla.mozilla.org/show_bug.cgi?id=856743)).

### Other causes

After plugins, **font loading operations** are the most frequent single category of browser hangs. To see an example of a fonts hang, restart your computer (or clear your OS disk cache by other means) and load the Wikipedia homepage ([bug 832546](https://bugzilla.mozilla.org/show_bug.cgi?id=832546)). You should see a hang lasting at least 100ms (depending on your storage device) while the browser loads dozens of fonts from disk to render all the different languages displayed on the page.

There are several existing bugs for font hangs ([bug 734308](https://bugzilla.mozilla.org/show_bug.cgi?id=734308) and [bug 699331](https://bugzilla.mozilla.org/show_bug.cgi?id=699331)), but I'd like to ask someone with some knowledge of  the fonts code to file individual bugs for the rest. I filed [bug 859558](https://bugzilla.mozilla.org/show_bug.cgi?id=859558) as a catch-all bug for all the font stacks that I did not recognize.

**Data:** [Top 50 chrome hang stack signatures, excluding plugin hangs]({{ site.url }}/assets/Top50_Without_PluginHangs.txt)<span class="emphasize">*</span>

After plugins and fonts, the locking done in the **HTTP network cache** causes most of the chrome hangs.  It seems the main thread often gets blocked waiting on a lock that is being held by a background cache thread. I assume the background thread is doing disk I/O while holding the contended lock. If you move your Firefox profile to a slow a storage device (e.g. an SD card), you can reliably reproduce these hangs by visiting new sites. The Necko team is currently working on plans for a [new network cache design](https://wiki.mozilla.org/Necko/Cache/Plans).

**Garbage Collection & Cycle Collection** are the third most common category of hangs. Surprisingly, incremental GC stacks also show up in the top 50.

### Other hangs

In addition to the hangs described above, there is a significant number of  stacks with JIT-ed JavaScript code, page reflows and other content operations. Since chrome hangs don't report the names of JavaScript functions on the stack and Telemetry doesn't collect page URLs, these stacks are not useful.  I filtered out these types of stacks and came up with a list of the top 50 "other" hangs.

**Data:** [Top 50 chrome hang stack signatures, excluding common hang types]({{ site.url }}/assets/Top50_Without_CommonHangTypes.txt) (plugins, GC, CC, HTTP cache, font, content hangs<span class="emphasize">*</span>)

This list shows:

* The JavaScript debugger frequently causes chrome hangs
* JavaScript calls an nsIDNSService interface function that synchronously resolves DNS names. Could we deprecate use of this function on the main thread?
* Switching graphics to hardware-accelerated mode after startup (filed [bug 859652](https://bugzilla.mozilla.org/show_bug.cgi?id=859652)) and Direct3D device initialization (filed [bug 859664](https://bugzilla.mozilla.org/show_bug.cgi?id=859664)) causes hangs
* Printing causes hangs (filed [bug 859655](https://bugzilla.mozilla.org/show_bug.cgi?id=859655))
* JavaScript functions called by nsContentPolicy::ShouldLoad take a long time to return
* DOM workers can take a long time to return the amount of memory in use (filed [bug 859657](https://bugzilla.mozilla.org/show_bug.cgi?id=859657))
* Extension and chrome JavaScript uses nsLocalFile for main thread I/O. Some of the main-thread calls to nsLocalFile::Exists might be TestPilot main-thread I/O (filed [bug 856867](https://bugzilla.mozilla.org/show_bug.cgi?id=856867))
* Proxy resolution jank doesn't seem to have been completely fixed ([bug 781732](https://bugzilla.mozilla.org/show_bug.cgi?id=781732))
* Destroying CSS style sheets takes a long time ([bug 819489](https://bugzilla.mozilla.org/show_bug.cgi?id=819489))
* nsSafeFileOutputStream is used on the main thread, blocking the main thread with fsyncs and other file I/O
* JSON stringify is sometimes called on very large objects
* Main-thread SQL is a common source of hangs, e.g. nsNavBookmarks::QueryFolderChildren, nsAnnotationService::GetPageAnnotationString, nsDownload::UpdateDB
* JAR files are opened and closed from the main thread causing hangs

I have not identified all of the hangs in this top list:

* nsCryptoHash::UpdateFromStream is called from the main thread with a file stream input. I haven't found the source of this
* A timer callback evals a string and calls js::SourceCompressorThread::waitOnCompression which causes hangs
* nsIncrementalDownload::OnDataAvailable writes downloaded data to disk on the main thread
* nsExternalAppHandler::SaveToDisk moves files on the main thread (known isssue?)

### Conclusion

Plugins are the foremost cause of Firefox janks lasting multiple seconds. Fonts, GC/CC, and the HTTP network cache are also common sources. The long tail of hang signatures contains new bugs, some of which could be fixed quickly.

