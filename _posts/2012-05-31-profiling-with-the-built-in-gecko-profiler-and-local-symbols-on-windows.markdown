---
author: Vladan
comments: true
date: 2012-05-31
layout: post
slug: profiling-with-the-built-in-gecko-profiler-and-local-symbols-on-windows
title: Profiling with the Built-in Gecko Profiler and Local Symbols on Windows
wordpress_id: 71
---
If you would like to use the [built-in Gecko Profiler](https://developer.mozilla.org/en/Performance/Profiling_with_the_Built-in_Profiler) with a local build of Firefox for Windows, you will need to point the profiler to a local [Snappy Symbolication Server](https://github.com/vdjeric/Snappy-Symbolication-Server/) instead of the official Mozilla symbolication server. The server will host the local Firefox symbols for the profiler and it will fetch any symbols not available locally from the official Mozilla symbolication server (e.g. symbols for Windows DLLs and plugin DLLs).

Instructions:  
[https://developer.mozilla.org/en/Performance/Profiling_with_the_Built-in_Profiler_and_Local_Symbols_on_Windows](https://developer.mozilla.org/en/Performance/Profiling_with_the_Built-in_Profiler_and_Local_Symbols_on_Windows)
