---
author: Vladan
comments: true
date: 2012-06-09
layout: post
slug: cache-plugin-font-operations-most-common-in-chrome-hang-reports
title: Cache, plugin, font operations most common in chrome hang reports
wordpress_id: 75
---
A few months ago, in [bug 712109](https://bugzilla.mozilla.org/show_bug.cgi?id=712109), I added functionality for reporting temporary main thread hangs to Telemetry and recently I obtained [the first batch](https://metrics.mozilla.com/projects/browse/METRICS-663) of reports. Hang reporting works by monitoring the event loop for inactivity -- if the event loop has not started processing a new event for 10 seconds, then the hang monitor will capture a snapshot of the main thread's call stack and attach the addresses to the next Telemetry report along with the measured duration of the hang. The default minimum hang duration is 10 seconds, but this value is configurable through the _hangmonitor.timeout_ preference. The chrome hang reporter only captures a single snapshot of the call stack, so the captured stack may not always be representative of all of Firefox's activities during the hang. Generally, it is enough to identify the offending code.

### The Hang Reporter Population

Over the 2-month period covered by this batch of data, there were only 42 Telemetry reports containing chrome hangs. This is because hang reporting is only enabled on the _nightly-profiling_ branch as it requires frame pointers for  unwinding the call stack. Additionally, there aren't many users on the nightly-profiling branch -- in fact, yesterday, there were only 14 unique Telemetry submissions by users of the _nightly-profiling_ branch.  Further, we currently don't save chrome hangs in persistent Telemetry, so some of the reports were likely lost during browser restarts ([bug 763113](https://bugzilla.mozilla.org/show_bug.cgi?id=763113)).

Some details immediately jumped out of the hang report data:

  * The median hang duration is 20 seconds.
  * Two underpowered hardware configurations kept popping up in the hang reports, so these two users are over-represented in the data. Together, they contributed 25 of the 42 Telemetry reports.
  * Most Telemetry pings with a chrome hang have only a single hang, but some reports have as many as 10 hangs during a single browsing session.

In order to symbolicate the PCs in the hang reports, we rely on Breakpad symbols. Unfortunately, it seems that symbols for nightly builds are deleted after 30 days, so most of the hangs from April and early May can not be symbolicated. That left 17 Telemetry pings out of 42 whose hangs could be symbolicated. Going forward, the plan is to automate this process and have symbolication done every day ([bug 763116](https://bugzilla.mozilla.org/show_bug.cgi?id=763116)).

I think it would also make sense to lower the hang threshold ([bug 763124](https://bugzilla.mozilla.org/show_bug.cgi?id=763124)). A threshold of 5 seconds likely wouldn't overwhelm the Metrics servers even if we deployed hang reporting on the regular nightly channel and it would allow us to detect more subtle performance issues.

### TOP HANGS

#### Cache Operations

Out of the 33 symbolicated hang stacks, call stacks ending with cache operations are the most common (12 hang stacks). A variety of cache operations are represented in these stacks:

* nsCacheEntryDescriptor::GetMetaDataElement **x3**
* nsCacheEntryDescriptor::GetStoragePolicy **x3**
* nsCacheEntryDescriptor::GetDeviceID **x2**
* nsCacheEntryDescriptor::Release **x2**
* nsCacheEntryDescriptor::GetExpirationTime **x2**

All of the stacks were reported by a single Windows XP machine, however the reports are over the span of a week and this machine is responsible for almost half of the hang stacks in the entire data set. I think these stacks are a good argument for devoting additional resources to investigating and improving the performance of our caching mechanisms.

#### Plugins

The second most common category of hangs are those involving plugins: loading plugins, destroying plugins, setting the window for a plugin and scripting plugins. In each of these cases, the main thread gets stuck waiting for the plugin running in the plugin-container process. It's not possible to identify the plugin using only the callstack and unfortunately, we currently don't collect the list of installed plugins.

#### GetFontTable

There were 2 hangs reported while getting font tables. I've also recently personally witnessed brief hangs lasting a couple of seconds while fetching the font list. I filed [bug 763134](https://bugzilla.mozilla.org/show_bug.cgi?id=763134).

Excerpt from a sample hang, lasting 16 seconds:

    KiFastSystemCallRet (in ntdll.dll)
    GDIFontEntry::GetFontTable(unsigned int,FallibleTArray<unsigned char> &) (in xul.dll)
    GDIFontEntry::ReadCMAP() (in xul.dll)
    gfxFontFamily::ReadAllCMAPs() (in xul.dll)
    gfxPlatformFontList::RunLoader() (in xul.dll)
    gfxFontInfoLoader::LoaderTimerFire() (in xul.dll)
    gfxFontInfoLoader::LoaderTimerCallback(nsITimer *,void *) (in xul.dll)
    nsTimerImpl::Fire() (in xul.dll)
    ...

#### GC

The data set contained two hangs where GC took 15 seconds and 31 seconds, and another hang where GC took 24 seconds but it also encompassed destructing an HTML document.

#### Gradients

There was a single Telemetry report containing 10 chrome hangs with very similar hang stacks of about ~30 seconds each. All of the stacks had to do with painting gradients via D2D. I think this issue might already be covered in [bug 750871](https://bugzilla.mozilla.org/show_bug.cgi?id=750871).

    DrawingContext::FillRectangle(D2D_RECT_F const *,ID2D1Brush *) (in d2d1.dll)
    D2DRenderTargetBase<ID2D1BitmapRenderTarget>::FillRectangle(D2D_RECT_F const *,ID2D1Brush *) (in d2d1.dll)
    _cairo_d2d_fill (in gkmedias.dll)
    _cairo_gstate_fill (in gkmedias.dll)
    _moz_cairo_fill_preserve (in gkmedias.dll)
    nsCSSRendering::PaintGradient(nsPresContext *,nsRenderingContext &,nsStyleGradient *,nsRect const &,nsRect const &,nsRect const &) (in xul.dll)
    nsImageRenderer::Draw(nsPresContext *,nsRenderingContext &,nsRect const &,nsRect const &,nsPoint const &,nsRect const &) (in xul.dll)
    nsCSSRendering::PaintBackgroundWithSC(nsPresContext *,nsRenderingContext &,nsIFrame *,nsRect const &,nsRect const &,nsStyleContext *,nsStyleBorder const &,unsigned int,nsRect *) (in xul.dll)
    nsCSSRendering::PaintBackground(nsPresContext *,nsRenderingContext &,nsIFrame *,nsRect const &,nsRect const &,unsigned int,nsRect *) (in xul.dll)
    nsDisplayCanvasBackground::Paint(nsDisplayListBuilder *,nsRenderingContext *) (in xul.dll)
    mozilla::FrameLayerBuilder::DrawThebesLayer(mozilla::layers::ThebesLayer *,gfxContext *,nsIntRegion const &,nsIntRegion const &,void *) (in xul.dll)
    mozilla::layers::ThebesLayerD3D10::DrawRegion(nsIntRegion &,mozilla::layers::Layer::SurfaceMode) (in xul.dll)
    mozilla::layers::ThebesLayerD3D10::Validate(mozilla::layers::ReadbackProcessor *) (in xul.dll)
    mozilla::layers::ContainerLayerD3D10::Validate() (in xul.dll)
    mozilla::layers::ContainerLayerD3D10::Validate() (in xul.dll)
    mozilla::layers::LayerManagerD3D10::Render() (in xul.dll)
    mozilla::layers::LayerManagerD3D10::EndTransaction(void (*)(mozilla::layers::ThebesLayer *,gfxContext *,nsIntRegion const &,nsIntRegion const &,void *),void *,mozilla::layers::LayerManager::EndTransactionFlags) (in xul.dll)
    nsDisplayList::PaintForFrame(nsDisplayListBuilder *,nsRenderingContext *,nsIFrame *,unsigned int) (in xul.dll)
    nsLayoutUtils::PaintFrame(nsRenderingContext *,nsIFrame *,nsRegion const &,unsigned int,unsigned int) (in xul.dll)
    PresShell::Paint(nsIView *,nsIWidget *,nsRegion const &,nsIntRegion const &,bool) (in xul.dll)
    nsViewManager::Refresh(nsView *,nsIWidget *,nsIntRegion const &,bool) (in xul.dll)
    nsViewManager::DispatchEvent(nsGUIEvent *,nsIView *,nsEventStatus *) (in xul.dll)
    ...

It's also possible that a developer had a debugger attached and that these pauses are from code hitting breakpoints. Telemetry submissions should add a flag to indicate whether a debugger is attached ([bug  763138](https://bugzilla.mozilla.org/show_bug.cgi?id=763138)).

#### Other Hangs

There were half a dozen other hangs in this data set that I still need to investigate further as their causes are somewhat perplexing and I am not familiar with the code in question. For example, in one hang stack, the CreateToolhelp32Snapshot API call used for gathering a list of libraries took 19 seconds while waiting to enter a critical section. In another, it looked like JavaScript ran for 63 seconds without being interrupted by the slow-script dialog box.

If anyone is interested in diagnosing these hangs, I'd appreciate it if you could take a look at the stacks and point me in the right direction or file bugs if you recognize a potential cause. You can leave me a comment on this post below.

These are the remaining stacks: [{{ site.url }}/assets/other_hang_stacks.txt]({{ site.url }}/assets/other_hang_stacks.txt)

