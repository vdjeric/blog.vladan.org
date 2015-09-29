---
author: Vladan
comments: true
date: 2012-03-22
layout: post
slug: introducing-chrome-hang-reporting-and-the-symbolication-server
title: Introducing Chrome Hang Reporting and the Symbolication Server
wordpress_id: 36
---
The [Hang Reporter](https://wiki.mozilla.org/Hang_Detector_and_Reporter:) is a new Firefox feature that reports **transient** main thread hangs to Telemetry. It relies on the existing hang monitor thread to detect when the main thread has not processed any new events in the last X seconds. If it detects a hang, it will suspend the main thread to walk its call stack for PCs and to gather version and address information about the modules loaded in process memory. This data along with the hang's duration is reported to Telemetry, assuming the user has opted in to submitting performance data. This Telemetry hang information will help us identify and reduce common causes of browser "stalls". In the interest of saving Telemetry bandwdith, only the information about the modules in memory directly involved in chrome hangs is reported + the memory maps in each hang report are a diff against the memory map in the first hang report of the session.

Currently, the Hang Reporter is only active on Windows Nightly builds from the [Profiling branch](http://ftp.mozilla.org/pub/mozilla.org/firefox/nightly/latest-profiling/) because it relies on having Firefox compiled with frame pointers to unwind the call stacks. The Hang Reporter functionality is #ifdef'd out on regular release builds. The default minimum interval for capturing a hang is 10 seconds, but the value can be changed via the "_hangmonitor.timeout_" config preference.

This next bit of functionality is not quite ready for prime time, but you can examine your own browser's hang reports locally by installing [this custom version](https://people.mozilla.com/~vdjeric/ping.telemetry.xpi) of the about:telemetry extension that adds a "Main Thread Hangs" section to the about:telemetry page and a button for causing hangs at will for testing.  You have to use either a local debug build of Firefox with symbols or a Nightly Profiling build to see the hang reports.

The hang reports contain raw PCs from the hung stack and a list of the modules involved:

* * *

### Main Thread Hangs:

### Hang report #1 (1 seconds):

**Stack:** 0x6027b8df 0x6027b897 0x602ce2b7 0x602cf482 0x602d2a7a 0x602d24c1 0x1d6cef50

**Memory map:**  
0x60230000,mozjs.dll,6107136,6,{3680d18b-e796-48be-b6fa-92475b2636b9},mozjs.pdb

### Hang report #2 (8 seconds):

**Stack:** 0x75b9f5be 0x10e9051d 0x10e90928 0x112441a6 0x111cb534 0x11018c6f 0x112a4f4e 0x112a4e72 0x112a4d7d 0x10e905b0 0x10e33907 0x10b59f4a 0xf884cac 0x12324ac 0x1231d26 0x1231319 0x1237cf8 0x1237b3f 0x74cf339a 0x775d9ef2 0x775d9ec5

**Memory map:**  
0x1230000,firefox.exe,978944,6,{5715c404-ce7c-4e42-bb7b-08da9bbd57e8},firefox.pdb
0xf780000,xul.dll,56299520,20,{ad69354f-72e3-4aa7-9607-02a519e386df},xul.pdb
0x74ce0000,kernel32.dll,1114112,2,{dfb4e9eb-d165-4db2-acf1-290cd316cea2},wkernel32.pdb
0x75b60000,USER32.dll,1048576,2,{0fce9cc3-01ed-4567-a819-705b2718e1d6},wuser32.pdb
0x775a0000,ntdll.dll,1572864,2,{d74f79eb-1f8d-4a45-abcd-2f476ccabacc},wntdll.pdb

* * *
<br/>
Pending [security](https://wiki.mozilla.org/Security/Reviews/SnappySymbolSrv) & [privacy review](https://wiki.mozilla.org/Privacy/Reviews/SnappySymbolicServer), a symbolication server will eventually be available publicly to symbolicate these hang reports for about:telemetry users. This symbolication server will also be used by the [SPS profiler](https://developer.mozilla.org/en/Performance/Profiling_with_the_Built-in_Profiler). If you are a developer comfortable with Firefox development and you feel like killing a bit of time :), you can already symbolicate the Firefox portions of these hang reports by running our new [Snappy Symbolication Server](https://wiki.mozilla.org/Snappy_Symbolication_Server) locally ([available on Github](https://github.com/vdjeric/Snappy-Symbolication-Server/)). You will need to [run the server locally](https://github.com/vdjeric/Snappy-Symbolication-Server/blob/master/README) on port 8000 with the config file pointing to your local .sym symbol files. You can generate these .sym files by running "make buildsymbols" from the object directory of your local build. You can then use the "Symbolicate Stacks" functionality in the about:telemetry page.

I don't expect <del>many</del> any people to go through the trouble of setting all this up :), but the Hang Reporter really does become informative to use after configuring the hang interval to its minimum (1 second) and observing how often and why the browser hangs during regular browsing.

**EDIT:** Further instructions on setting up the Snappy Symbolication Server can be found [here]({% post_url 2012-03-28-setting-up-snappy-symbolication-server-locally %}).
