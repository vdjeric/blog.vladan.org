---
author: Vladan
comments: true
date: 2015-08-15
layout: post
slug: new-policy-24-hour-backouts-for-major-talos-regressions
title: 'New policy: 24-hour backouts for major Talos regressions'
wordpress_id: 167
---
Now that I've caught your attention with a sufficiently provocative title, please check out this new Talos regression policy that we
<span class="emphasize"><sup>*</sup></span>
will be trying out starting next week :)

[https://groups.google.com/forum/#!topic/mozilla.dev.platform/QHdn-ogf8kQ](https://groups.google.com/forum/#!topic/mozilla.dev.platform/QHdn-ogf8kQ)

**tl;dr:** _Perf sheriffs will back out any Talos regression of 10% or more if it affects a reliable test on Windows. We'll give the patch author 24 hours to explain why the regression is acceptable and shouldn't be backed out. Perf sheriffs will aim to have such regressions backed out within 48 hours of landing._

I promise this policy is much more nuanced and thought-through than the title or summary might suggest, but I really want to hear developers' opinions.

<span class="emphasize"><sup>*</sup></span>
I'm taking point on publicizing this new policy and answering any questions, but [Joel Maher](https://elvis314.wordpress.com/), [William Lachance](http://wrla.ch/blog/) and [Vaibhav Agarwal](https://vaibhavag.wordpress.com/) of the A-Team did all the heavy lifting. They built the tools for detecting & investigating Talos regressions and they're the perf sheriffs.

[Avi Halachmi](http://avih.github.io/) from my team is helping to check the tools for correctness. I just participate in Talos policy decisions and occasionally act as an (unintentional) spokesperson :)
