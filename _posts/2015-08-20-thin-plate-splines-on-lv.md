---
layout: post
title:  "Thin-Plate Splines Experiment on the Left Ventricle"
date:   2015-08-20
tags: [en]
comments: true
---

Here are four examples of 3D left ventricular (LV) motion. Each of them has different motion patterns. The question is *how we can compare them* with respect to the distributions of normal ranges.

![original motion]({{ base.url }}/images/RCH0104_ORG.gif)

My idea is to fix all motions to start from the same shape. To do this, I've used a [thin-plate splines](http://mathworld.wolfram.com/ThinPlateSpline.html) warping implementation from the [R Morpho](https://cran.r-project.org/web/packages/Morpho/index.html) package. It's a beautiful [3D Geometric Morphometrics and Shape Analysis](https://www.uniklinik-freiburg.de/anthropology/research/applied-anthropology/geometric-morphometrics-and-shape-analysis.html) package written by [Stefan Schlager](http://zarquon42b.github.io/about/) from the University of Freiburg.

So I calculated the mean shape of 1,991 asymptomatic LV models at ED, warped each patient ED shape to the mean ED shape, and made the same deformation to the remaining frames. The results are shown below. Note that at the beginning of contraction, all shapes are identical, which is the mean ED shape.

![tps motion]({{ base.url }}/images/RCH0104_TPS.gif)
