---
author: Vladan
comments: true
date: 2012-01-03
layout: post
slug: hello-world-2
title: Hello World
wordpress_id: 4
---
I have been postponing blogging for a while now, but I would like to start 2012 on the right foot, so here goes :)

I'm Vladan, a new-ish member of Mozilla's Performance team, having started back in October 2011. I did my schooling at the University of Waterloo (undergrad) and University of Toronto (grad) and worked on a [foreign exchange retail trading platform](http://fxtrade.oanda.com) before joining Mozilla.

So far I've had the opportunity to work on a couple of interesting performance projects. My first major bug was to extend Telemetry performance reporting to report slow SQL statements ([bug 699051](https://bugzilla.mozilla.org/show_bug.cgi?id=699051)). As a privacy precaution, Telemetry will only record the strings of prepared SQL statements executed against known Firefox DBs. So, for example, query arguments will never be recorded and neither will any SQL executed against extension-specific DBs.

For the time being, only [Firefox Nightlies](http://nightly.mozilla.org/) have the new slow SQL instrumentation. You can examine the data gathered by Telemetry by installing the (restartless!) [about:telemetry extension](https://addons.mozilla.org/en-US/firefox/addon/abouttelemetry/) and then navigating to the the "_about:telemetry_" URL. If any SQL statements in your browser needed more than 100ms to execute, you will see tables looking something like this:

<div class="oldblog">
<table id="mainSqlTable"><caption>Slow SQL Statements on Main Thread</caption>
<tbody>
<tr>
<th>Hits</th>
<th>Avg. Time (ms)</th>
<th>Statement</th>
</tr>
<tr>
<td>1</td>
<td>109</td>
<td>UPDATE moz_bookmarks SET lastModified = :date WHERE id = :item_id</td>
</tr>
</tbody>
</table>
<hr />
<table id="otherSqlTable"><caption>Slow SQL Statements on Other Threads</caption>
<tbody>
<tr>
<th>Hits</th>
<th>Avg. Time (ms)</th>
<th>Statement</th>
</tr>
<tr>
<td>3</td>
<td>135</td>
<td>DELETE FROM moz_classifier WHERE table_id=?1 AND chunk_id=?2</td>
</tr>
</tbody>
</table>
</div>

In addition to work on SQLite performance, I am also currently working on a few other tasks such as reporting main thread hangs ([bug 712109](https://bugzilla.mozilla.org/show_bug.cgi?id=712109)) and helping speed up Firefox shutdown (bugs [684513](https://bugzilla.mozilla.org/show_bug.cgi?id=684513), [662444](https://bugzilla.mozilla.org/show_bug.cgi?id=662444)). I will try to blog about some of this work next week.



