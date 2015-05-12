---
layout: post
title: "Stata: wbopendata temp file error r(601) and solution"
categories:
  - coding
tags:
  - stata
  - wbopendata
---

Recently I tried `wbopendata`(version 9.0) with Stata(Stata/SE 12.0 for Windows (32-bit), Revision 13 Oct 2011), but got an error(601), saying that an temp file is missing.

## Short Solution:
upate your Stata to latest version.

<!-- more -->

## Details

By running `set trace on`, we can view the very steps running this command. In my environment and test arguments, the error occurs here:

```
  - if ("`indicator'" != "") {
  = if ("" != "") {
    capture : copy "`servername'/`language'/countries/`country2'/`parameter'format=csv" `temp'
    }
  - insheet using `temp', `clear' name
  = insheet using C:\Users\FRANCI~1\AppData\Local\Temp\ST_01000001.tmp,  name
file C:\Users\FRANCI~1\AppData\Local\Temp\ST_01000001.tmp not found
```

Try the `copy` command, it failed to download the data file. So the tmp file does not exist, and of course the following `insheet` reports file missing error.

`copy` is a build-in function, so I updated my Stata to latest version(Revision 23 Jan 2014), and try `wbopendata` again, it works well. Probably the `copy` command is the reason, but as it is a built-in function, I don't know how to dig into it.

Hope this help.
