---
author: Vladan
comments: true
date: 2015-09-08
layout: post
slug: update-your-custom-telemetry-dashes-telemetry-js-is-obsolete
title: 'Update your custom Telemetry dashes: telemetry.js is obsolete'
wordpress_id: 176
---
If you built a custom dashboard and used [telemetry.js](https://telemetry.mozilla.org/docs.html#Telemetry) to query [telemetry.mozilla.org](https://telemetry.mozilla.org) for histogram data, you'll need to switch your dash to a newer library -- either the [new telemetry.js library](https://github.com/mozilla/telemetry-dashboard/tree/master/v2) (preferred option) or [our shim library](https://github.com/Uberi/telemetry-dashboard/blob/v1-shim/v2/v1-shim.js).

This change is necessary because Telemetry is retiring its old backend in favour of the new "unified" Telemetry backend which combines the capabilities of both FHR & Telemetry. The old backend sprouted data quality issues and supporting two backends is too time-consuming for a small team.

The old telemetry.js dashboarding library will be retired starting
<span class="emphasize">Monday, September 14th</span>
next week. If you don't replace telemetry.js in your dashboard, your dash will stop seeing any new data.

There is additional background on this decision here: [http://anthony-zhang.me/blog/telemetry-demystified/](http://anthony-zhang.me/blog/telemetry-demystified/)

You can ask for help with porting your dash in the comments below, or ping vladan / mreid / rvitillo on #telemetry
