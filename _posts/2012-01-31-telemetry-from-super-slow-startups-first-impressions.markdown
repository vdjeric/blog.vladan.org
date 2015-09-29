---
author: Vladan
comments: true
date: 2012-01-31
layout: post
slug: telemetry-from-super-slow-startups-first-impressions
title: Telemetry from Super-slow Startups, First Impressions
wordpress_id: 18
---
I have been investigating very slow startups by looking at Telemetry reports of [startups taking longer than 30 seconds](https://metrics.mozilla.com/projects/browse/METRICS-245). There were 49 reports of super-slow startups during a one month period starting Dec 15th that met the following criteria:

* First paint<span class="emphasize">*</span> > 30 seconds
* Uninterrupted startup
* Windows OS
* Recent Nightly or Aurora build reporting on [slow SQL Telemetry](https://bugzilla.mozilla.org/show_bug.cgi?id=699051)
* Uptime < 5minutes


<span class="emphasize">*</span>
There are 3 metrics used for representing startup speed: "main", "firstPaint" and "sessionRestore". The names are mostly self-explanatory, but you can find more detail [here](http://blog.mozilla.org/tglek/2011/01/14/builtin-startup-measurement/).

Two thirds of the slow startup reports had more than 5 addons. While addons may be contributing to slow startups in some cases, there doesn't seem to be much difference between Telemetry reports for browsers with few addons and many addons. There were also two reports with negative values for uptime (filed [bug 722651](https://bugzilla.mozilla.org/show_bug.cgi?id=722651), similar to [bug 670008](https://bugzilla.mozilla.org/show_bug.cgi?id=670008)), but this is a minor bug and not a sign of corruption in the reports.

<span style="emphasize">UPDATE:</span>
Some of these reports may come from developer builds with Telemetry manually enabled ([Bug 722240](https://bugzilla.mozilla.org/show_bug.cgi?id=722240)) which might explain some of the long startup times. It doesn't look like there is any way to distinguish local builds from official builds in this dataset (fixed in [Bug 720875](https://bugzilla.mozilla.org/show_bug.cgi?id=720785)).

### Startup times

The root causes behind the long startups are not immediately obvious from these Telemetry reports. This is partly because Telemetry doesn't measure everything of interest yet and partly because the Telemetry data was not divided into startup and post-startup measurements (now fixed in bug [720456](https://bugzilla.mozilla.org/show_bug.cgi?id=720456)). Currently, the Telemetry histograms contain measurements both from startup and the initial 5 minutes of browser use.

Most of the startups (as measured by firstPaint) were between 30-60s long. In 32 of the 49 reports, there was a delay of at least 10 seconds before XRE_main was called, perhaps suggesting that these systems were very slow or the startup took place during a time of very high contention for the computer's resources, perhaps during boot up or a resume from hibernation. In 7 reports, the time to reach XRE_main was even greater than the additional time for producing the first paint. The additional time required for session restore was either negligible (nothing to restore) or surprisingly relatively small compared to total startup time (less than 15%).

One observation of note is that sessionRestore sometimes occurs before firstPaint ([bug 715402](https://bugzilla.mozilla.org/show_bug.cgi?id=715402)). This likely further degrades the user experience during slow startups.

### Histogram data

The total times recorded by Telemetry histograms in the slow startup reports are not large enough to fully explain the lengthy start up times. The only histograms reporting times on par with the total start up times (or much larger!) are those for:

* asynchronous SQL execution (MOZ_STORAGE_ASYNC_REQUESTS_MS)
* URL classifier work (URLCLASSIFIER_PS_CONSTRUCT_TIME, MOZ_SQLITE_URLCLASSIFIER_READ_MS, ...)
* and HTTP page loading (HTTP_SUBITEM_OPEN_LATENCY_TIME, HTTP_SUB_COMPLETE_LOAD_CACHED, ...)

None of these operations should be affecting startup times: page-loading should not be happening before first paint (barring bug 715402 above) and URL classifier operations &  asynchronous SQL are executed on worker threads and should not delay startup.

There are, however, a few operations lasting multiple seconds that should not be, such as DWRITEFONT_DELAYEDINITFONTLIST_TOTAL and DWRITEFONT_DELAYEDINITFONTLIST_COLLECT taking 17 seconds each in one report. Similar font performance issues are already being addressed in [bug 705594](https://bugzilla.mozilla.org/show_bug.cgi?id=705594).

The table below shows a few histograms of interest with total times in excess of 1 second during the initial 2-4 minutes:

<div class="oldblog">
<table width="100%" border="1" cellspacing="0" cellpadding="5"><colgroup> <col width="48*" /> <col width="208*" /> </colgroup>
<tbody>
<tr valign="TOP">
<td width="19%">
<p align="CENTER"><strong># of Telemetry reports
</strong></p>
</td>
<td width="81%">
<p align="LEFT"><strong>Histogram</strong></p>
</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">32</p>
</td>
<td width="81%">FIND_PLUGINS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">28</p>
</td>
<td width="81%">MOZ_SQLITE_OTHER_SYNC_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">28</p>
</td>
<td width="81%">CACHE_DEVICE_SEARCH</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">25</p>
</td>
<td width="81%">MOZ_SQLITE_OPEN_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">22</p>
</td>
<td width="81%">MOZ_SQLITE_OPEN_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">21</p>
</td>
<td width="81%">NETWORK_DISK_CACHE_OPEN</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">18</p>
</td>
<td width="81%">MOZ_SQLITE_PLACES_READ_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">16</p>
</td>
<td width="81%">TELEMETRY_PING</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">12</p>
</td>
<td width="81%">GC_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">10</p>
</td>
<td width="81%">MOZ_SQLITE_OTHER_READ_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">9</p>
</td>
<td width="81%">GC_MARK_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">9</p>
</td>
<td width="81%">XUL_REFLOW_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">8</p>
</td>
<td width="81%">CYCLE_COLLECTOR</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">8</p>
</td>
<td width="81%">MOZ_SQLITE_PLACES_SYNC_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">6</p>
</td>
<td width="81%">MOZ_SQLITE_WEBAPPS_SYNC_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">6</p>
</td>
<td width="81%">CACHE_DISK_SEARCH</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">5</p>
</td>
<td width="81%">GC_SWEEP_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">5</p>
</td>
<td width="81%">SYSTEM_FONT_FALLBACK_FIRST</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">3</p>
</td>
<td width="81%">MOZ_SQLITE_OTHER_SYNC_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">3</p>
</td>
<td width="81%">GDI_INITFONTLIST_TOTAL</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">2</p>
</td>
<td width="81%">MOZ_SQLITE_OTHER_READ_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">2</p>
</td>
<td width="81%">NETWORK_DISK_CACHE_DELETEDIR</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">2</p>
</td>
<td width="81%">IMAGE_DECODE_ON_DRAW_LATENCY</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">2</p>
</td>
<td width="81%">FONTLIST_INITFACENAMELISTS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">2</p>
</td>
<td width="81%">FX_TAB_ANIM_OPEN_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">1</p>
</td>
<td width="81%">CHECK_JAVA_ENABLED</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">1</p>
</td>
<td width="81%">FONTLIST_INITOTHERFAMILYNAMES</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">1</p>
</td>
<td width="81%">MOZ_SQLITE_COOKIES_READ_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">1</p>
</td>
<td width="81%">MOZ_SQLITE_PLACES_WRITE_MAIN_THREAD_MS</td>
</tr>
<tr valign="TOP">
<td width="19%">
<p align="CENTER">1</p>
</td>
<td width="81%">SYSTEM_FONT_FALLBACK</td>
</tr>
</tbody>
</table>
</div>

Although the operations measured by the histograms above are likely not solely responsible for the long startup times, they merit further performance investigation.

### Slow SQL data

Adding to the slow system I/O hypothesis, all of the slow startup reports contain multiple [SQL statements with execution times over 100ms]({% post_url 2012-01-03-hello-world-2 %}). Nevertheless, the time taken by these slow SQL statements is still not enough to adequately explain the slow startups.

These are the top 10 most common slow SQL statements across the slow startup reports:

**Main Thread**

<div class="oldblog">
<table width="100%" border="1" cellspacing="0" cellpadding="5"><colgroup> <col width="35*" /> <col width="221*" /> </colgroup>
<tbody>
<tr valign="TOP">
<td width="13%">
<p align="CENTER"><strong># of Telemetry reports </strong></p>
</td>
<td width="87%">&nbsp;

<strong>SQL string</strong></td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">23</p>
</td>
<td width="87%">RELEASE SAVEPOINT 'default'</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">13</p>
</td>
<td width="87%">SELECT h.id, h.url, IFNULL(b.title, h.title), h.rev_host, h.visit_count, h.last_visit_date, f.url, null, b.id, b.dateAdded, b.lastModified, b.parent, null, h.frecency, b.position, b.type, b.fk, b.guid FROM moz_bookmarks b LEFT JOIN moz_places h ON b.fk = h.id LEFT JOIN moz_favicons f ON h.favicon_id = f.id WHERE b.parent = :parent ORDER BY b.position ASC</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">13</p>
</td>
<td width="87%">INSERT OR IGNORE INTO moz_places (url, title, rev_host, hidden, typed, frecency, guid) VALUES (:page_url, :page_title, :rev_host, :hidden, :typed, :frecency, GENERATE_GUID())</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">12</p>
</td>
<td width="87%">SELECT session FROM moz_historyvisits ORDER BY visit_date DESC</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">12</p>
</td>
<td width="87%">SELECT COUNT(1) AS numEntries FROM moz_formhistory</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">10</p>
</td>
<td width="87%">DELETE FROM addon WHERE internal_id=:internal_id</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">10</p>
</td>
<td width="87%">DELETE FROM moz_formhistory WHERE lastUsed &lt;= :expireTime</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">10</p>
</td>
<td width="87%">SELECT a.item_id FROM moz_anno_attributes n JOIN moz_items_annos a ON n.id = a.anno_attribute_id WHERE n.name = :anno_name</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">8</p>
</td>
<td width="87%">UPDATE moz_bookmarks SET lastModified = :date WHERE id = :item_id</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">7</p>
</td>
<td width="87%">PRAGMA page_size</td>
</tr>
</tbody>
</table>
</div>

**Other Threads**

<div class="oldblog">
<table width="100%" border="1" cellspacing="0" cellpadding="5"><colgroup> <col width="34*" /> <col width="222*" /> </colgroup>
<tbody>
<tr valign="TOP">
<td width="13%">
<p align="CENTER"><strong># of Telemetry reports</strong></p>
</td>
<td width="87%">&nbsp;

<strong>SQL string</strong></td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">45</p>
</td>
<td width="87%">SELECT domain, partial_data, complete_data FROM moz_classifier</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">36</p>
</td>
<td width="87%">DELETE FROM moz_classifier WHERE table_id=?1 AND chunk_id=?2</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">19</p>
</td>
<td width="87%">UPDATE moz_places SET frecency = ROUND(frecency * .975) WHERE frecency &gt; 0</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">19</p>
</td>
<td width="87%">SELECT 1 FROM moz_places h WHERE url = ?1 AND last_visit_date NOTNULL</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">18</p>
</td>
<td width="87%">INSERT INTO expiration_notify (v_id, url, guid, visit_date, expected_results) SELECT v.id, h.url, h.guid, v.visit_date, :limit_visits FROM moz_historyvisits v JOIN moz_places h ON h.id = v.place_id WHERE (SELECT COUNT(*) FROM moz_places) &gt; :max_uris AND visit_date &lt; strftime('%s','now','localtime','start of day','-7 days','utc') * 1000000 ORDER BY v.visit_date ASC LIMIT :limit_visits</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">18</p>
</td>
<td width="87%">UPDATE moz_places SET frecency = CALCULATE_FRECENCY(:page_id) WHERE id = :page_id</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">18</p>
</td>
<td width="87%">SELECT id, title, hidden, typed, guid FROM moz_places WHERE url = :page_url</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">18</p>
</td>
<td width="87%">ANALYZE moz_places</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">17</p>
</td>
<td width="87%">INSERT INTO moz_historyvisits (from_visit, place_id, visit_date, visit_type, session) VALUES (:from_visit, :page_id, :visit_date, :visit_type, :session)</td>
</tr>
<tr valign="TOP">
<td width="13%">
<p align="CENTER">16</p>
</td>
<td width="87%">UPDATE moz_places SET frecency = CALCULATE_FRECENCY(id) WHERE frecency &lt; 0</td>
</tr>
</tbody>
</table>
</div>

Most of the statements above were executed during regular browser use and not during startup time. Perhaps the most interesting query in the list above is the main thread "PRAGMA page_size" query. It should be a quick read-only query, but in one report it took 4.888s on the main thread while another, more complex query took the exact same amount of time on a different thread. As it turns out, several main thread queries suffer from lock contention with queries on other threads (e.g. [bug 722242](https://bugzilla.mozilla.org/show_bug.cgi?id=722242)).

Another interesting point is the factor of 10 or even 100 difference between MOZ_STORAGE_ASYNC_REQUESTS_MS and the slow SQL telemetry times. This is caused by slow SQL telemetry limiting itself to reporting only on prepared statements executed against known Firefox databases. We should also track slow SQL executed from dynamic strings, perhaps by reporting the state of the JavaScript stack instead of the SQL string itself which may contain privacy-sensitive argument values ([bug 722368](https://bugzilla.mozilla.org/show_bug.cgi?id=722368)).

### Next steps

The obvious next steps are to 1) eliminate the SQL blind spots by tracking times for all SQL executed in the browser and 2) analyze startup data from the most recent Nightlies containing separate startup metrics. I will also be posting more startup analysis from the individual slow startup reports in this batch of data.
