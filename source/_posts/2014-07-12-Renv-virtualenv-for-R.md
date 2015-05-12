---
layout: post
title: "Renv: environement/version management for R"
categories:
  - coding
tags:
  - R
  - Renv
  - virtualenv
---

# About Renv
[Renv](https://github.com/viking/Renv) lets you be able to install multiple R versions and choose which to use globally or per-project.

<!-- more -->

Usage, tutorial and source and be found at its [github](https://github.com/viking/Renv). Here's some additional hints.

* When you add `Renv` to your path, the code given at the page above may not work, because your linux may not use `.bash_profile`, though this is common case. How to add it to your path depends on your linux, probably `.profile` if `.bash_profile` dose not work.

* [R-build](https://github.com/viking/R-build) is an `Renv` plugin that provides an Renv install command to compile and install different versions of R on UNIX-like systems.

  You can also use `R-build` without Renv in environments where you need precise control over R version installation.

  Again follow the official tutorial [here](https://github.com/viking/R-build).

* If you need your Renv R versions work with [RStudio](http://www.rstudio.com/), you need to provide `--enable-R-shlib` option to `./config`. If you are using `R-build`, you can set an environemnt named `CONFIGURE_OPTS` to provide additional configuration options.

It might not be so amazing as to have `virtualenv` with Python, but it helps when you need multiple R versions, or some packages of a pecific version. 