---
author: Vladan
comments: true
date: 2013-01-25
layout: post
slug: add-on-performance-problems
title: Add-on performance problems
wordpress_id: 102
---
<span class="emphasize">**NOTE TO READERS:**</span> I wrote this post to draw attention to a common pitfall when using SQLite DBs in extension and Mozilla code. I am not suggesting people stop using the addons mentioned below. The list of top sources of slow SQL is skewed by various factors: more popular add-ons are more likely to be in this list, add-ons that do slow operations against their own DBs will be in the list but add-ons that do slow operations against Firefox DBs will not, etc etc.

* * *
<br/>
The perf team has been using Telemetry data to find and fix performance problems in Firefox code, but the same data can also be used to find performance problems in add-ons.

The Telemetry data on long-running SQL statements, in particular, contains information about slow SQL operations carried out by add-ons. The aggregated reports are publicly accessible from the [Metrics dashboard](https://metrics.mozilla.com/)  (BrowserID account needed). If you navigate to the slow SQL section or click on [this link directly](http://tinyurl.com/bzwvv97), you will see stats on SQL statements that took longer than 100ms to execute on the main thread across at least 100 browsing sessions. In general, main thread I/O (including SQL) is a major performance problem in Firefox as the duration of I/O operations is essentially unbounded and any long running operation on the main thread causes browser unresponsiveness. As a privacy precaution, Telemetry never reports statement parameters and only reports the name of the DB if the SQL was executed against an extension DB. Note that the dashboard doesn't specify whether an SQL string was executed by Firefox or add-on code, but SQL executed against extension DBs has the form "Untracked SQL for \<_extension DB name_\>.sqlite". As can be seen from that dashboard, the majority of main-thread SQL work is still coming from Mozilla code, but addon SQL is well represented.

These are the top 20 addons doing main thread DB work*, found by sorting the slow SQL table on frequency ("Total Doc"):

1. Various extensions which use the customizable [Conduit.com toolbar code.](http://toolbar.conduit.com/) These toolbars sometimes ship with malware, but they also seem to be used in legitimate AMO extensions
2. The popular [ForecastFox](https://addons.mozilla.org/en-us/firefox/addon/forecastfox-weather/) weather extension
3. [Fast Dial](https://addons.mozilla.org/en-US/firefox/addon/fast-dial-5721/) visual bookmarks
4. [Evernote Web Clipper](https://addons.mozilla.org/en-us/firefox/addon/evernote-web-clipper/)
5. [Zotero](https://addons.mozilla.org/en-us/firefox/addon/zotero/) research tool
6. Mail.ru [toolbar](http://sputnik.mail.ru/)
7. [Web.de MailCheck](https://addons.mozilla.org/en-us/firefox/addon/webde-mailcheck/) + [Mail.com MailCheck](https://addons.mozilla.org/en-US/firefox/addon/mailcom-mailcheck/)
8. [Yoono sidebar](https://addons.mozilla.org/en-US/firefox/addon/yoono-twitter-facebook-linkedi/)
9. [Yandex toolbar](https://addons.mozilla.org/en-US/firefox/addon/yandexbar/)
10. [Lazarus](https://addons.mozilla.org/en-us/firefox/addon/lazarus-form-recovery/) form data recovery
11. [DownThemAll!](https://addons.mozilla.org/en-US/firefox/addon/downthemall/) downloader
12. [Xmarks Sync](https://addons.mozilla.org/en-us/firefox/addon/xmarks-sync/) bookmarking
13. Various extensions using [Brand Thunder](http://brandthunder.com/) themes code
14. [Ant Video Downloader](https://addons.mozilla.org/en-us/firefox/addon/video-downloader-player/)
15. [TRUSTe Tracker Protection](https://addons.mozilla.org/EN-US/firefox/addon/truste-tracker-protection/)
16. a DB which might be associated with [PrivacySuite](https://addons.mozilla.org/en-us/firefox/addon/privacysuite/) and [Targeted Advertising Cookie Opt-Out](https://addons.mozilla.org/en-US/firefox/addon/targeted-advertising-cookie-op/)
17. [GMail Checker](https://addons.mozilla.org/en-us/firefox/addon/gmail-checker/)
18. [Brief](https://addons.mozilla.org/en-us/firefox/addon/brief/) RSS reading
19. [Screenshot Pimp](https://addons.mozilla.org/en-US/firefox/addon/screenshot-pimp-screengrab-scr/) + [MediaPimp](https://addons.mozilla.org/en-US/firefox/addon/mediapimp-internet-radio-save-/) + [Notifications [...] for Facebook](https://addons.mozilla.org/en-US/firefox/addon/facebook-friend-request-notifi/)
20. [Form History Control](https://addons.mozilla.org/en-us/firefox/addon/form-history-control/)

\* The top offender is actually SQL executed against "cache.sqlite". This is the name of the DB used by the [Lightning](https://addons.mozilla.org/en-US/thunderbird/addon/lightning/) calendaring addon for Thunderbird. Its inclusion in the Firefox SQL list might be a bug in the dashboard or it might be an intentionally misleading name for a spyware DB. There were other highly ranked DBs I could not identify: addonstat.sqlite, store-pp.sqlite, livemargins_appcenter.sqlite, database.sqlite.

We will need a joint effort between Mozilla developers and add-on authors to make Firefox snappier for users. Our next step is to reach out to the authors of the above addons and ask them to change how they use SQLite DBs. We'll also need to improve our [documentation](https://developer.mozilla.org/en-US/docs/Extensions/Performance_best_practices_in_extensions) on best-practices for extension developers.

<span class="emphasize">**UPDATE:**</span> A few commenters pointed out that I never mentioned the async SQL APIs in my post. Indeed, developers should be using the asynchronous APIs to execute SQL statements off the main thread: [https://developer.mozilla.org/en-US/docs/Storage#Asynchronously](https://developer.mozilla.org/en-US/docs/Storage#Asynchronously)
