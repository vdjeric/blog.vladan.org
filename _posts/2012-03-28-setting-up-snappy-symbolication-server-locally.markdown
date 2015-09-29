---
author: Vladan
comments: true
date: 2012-03-28
layout: post
slug: setting-up-snappy-symbolication-server-locally
title: Setting up Snappy Symbolication Server locally
wordpress_id: 40
---
<span class="emphasize">UPDATE:</span>
If you have been granted access to Mozilla's jump host, you can create an SSH tunnel to the internal symbolication server:

    ssh -L 8000:breakpad-symbolapi1.dmz.phx1.mozilla.com:80 mpt-vpn.mozilla.com -N

You will then be able to use [Benoit's SPS extension](https://github.com/bgirard/Gecko-Profiler-Addon/blob/master/geckoprofiler.xpi) and the [custom about:telemetry extension](https://people.mozilla.com/~vdjeric/ping.telemetry.xpi) out of the box on Windows profiling builds.

<span class="emphasize">UPDATE #2:</span>
Refreshed the links to point to a more recent build of Firefox, the old version was incompatible with the latest SPS extensions

* * *
<br/>
In my [previous blog post]({% post_url 2012-03-22-introducing-chrome-hang-reporting-and-the-symbolication-server %}), I gave an overview of the new Hang Detector feature and the Snappy Symbolication Server. Since then, more than one person has asked me for info on deploying the server locally so I'll reproduce a quick and dirty walkthrough here for easy reference. The instructions below are the minimum required to see a demo of the functionality.

**Setting up a profiling Nightly:**  
1. Download and install the profiling Nightly from March 28th, 2012: [https://people.mozilla.com/~vdjeric/firefox-15.0a1.en-US.win32.installer.exe](https://people.mozilla.com/%7Evdjeric/firefox-15.0a1.en-US.win32.installer.exe)  
2. Disable updates in the profiling Nightly from Firefox -> Options -> Options -> Advanced -> Update -> "Never check for updates".  
This will prevent this test version of Nightly from getting out of sync with the debug symbols you will download in the next step.  
3. Opt into Telemetry from Firefox -> Options -> Options -> Advanced -> General -> "Submit performance data". Restart the browser.  
4. Install a custom version of the about:telemetry extension: [https://people.mozilla.com/~vdjeric/ping.telemetry.xpi](https://people.mozilla.com/%7Evdjeric/ping.telemetry.xpi)  

**Setting up a Symbolication Server locally:**  
5. Download some of the symbols for this Nightly profiling build from [https://people.mozilla.com/~vdjeric/symbols_ffx.zip](https://people.mozilla.com/%7Evdjeric/symbols_ffx.zip). Extract the zip file locally.  
6. From Windows command line: "git clone git://github.com/vdjeric/Snappy-Symbolication-Server.git". You can get "git" for Windows from [http://code.google.com/p/msysgit/](http://code.google.com/p/msysgit/) or you can install Cygwin.  
7. Edit the Snappy Symbolication Server's "sample.conf" file so that firefoxSymbolsPath and osSymbolsPath point to the "symbols_ffx" directory extracted in step #5. You might also want to set enableTracing = 1 to help with identifying issues.  
8. Run the symbolication server locally with "python symbolicationWebService.py sample.conf". You can get python for Windows from [http://www.activestate.com/activepython/downloads](http://www.activestate.com/activepython/downloads) or you can install Cygwin.  

**Testing:**  
9. Click on "Force Hang" from the ["about:telemetry"](about:telemetry) URL in the profiling Nightly, acknowledge all alerts. Reload the page and click "Symbolicate Stacks". You should now see a stack that looks something like this:  

<blockquote>
<h2><span style="text-decoration: underline;"><small><small>Hang report #1 (18 seconds):</small></small></span></h2>
<tt>??? (in ntdll.dll)</tt><br/>
<tt> PRMJ_Now (in mozjs.dll)</tt><br/>
<tt> js_Date(JSContext *,unsigned int,JS::Value *) (in mozjs.dll)</tt><br/>
<tt> ???</tt><br/>
<tt> ???</tt><br/>
</blockquote>

The question marks above are for OS libraries (ntdll.dll) for which you don't have local symbols and JIT-ed code on the bottom of stack which is not possible to stackwalk reliably.

Note: To generate your own Firefox symbols instead of downloading a sample as above, you'll first have to make a [local debug build of Firefox](https://developer.mozilla.org/en/Building_Firefox_with_Debug_Symbols) with an additional "ac_add_options --enable-profiling" line in your .mozconfig. After you've built this profiling version of Firefox, you can generate symbols for all your Firefox libraries by running "_make buildsymbols_" from your object directory.
