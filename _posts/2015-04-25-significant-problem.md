---
layout: post
title:  "Stop abusing p-value! It is not a scientific test."
date:   2015-03-12
tags: [en]
comments: true
---

In [The Cult of Statistical Significance](http://www.press.umich.edu/186351/cult_of_statistical_significance), Ziliak and McCloskey wrote

> "The problem we are highlighting is that the so-called test of statistical significance does not in fact answer a quantitative, scientific question. Statistical significance is not a *scientific* test. It is a philosophical, qualitative test. It does not ask how much. It asks *whether*. Existence, the question of whether, is interesting, **but it is not scientific**."

The book is light, thoughtful and enjoyable (*highly recommended*). It discusses the biggest mistake in science ever since [Fisher](http://en.wikipedia.org/wiki/Ronald_Fisher) ruined [Gosset](http://en.wikipedia.org/wiki/William_Sealy_Gosset)'s (student *t*-test) greatest invention in 1925, ... and yet it is still continuing.

I came across this book when I was reviewing a manuscript. The authors compared several markers with a "ground truth" value using [linear correlation analysis](http://en.wikipedia.org/wiki/Pearson_product-moment_correlation_coefficient). The correlation coefficients $$R$$ of each marker are A: (R=0.44, p<0.001), B: (0.43, p<0.001), C: (0.40, p<0.001), D: (0.24, p<0.001) and E: (0.22, p<0.001). Their proposed method is A.

Now, looking to these results, all markers did not have good prediction. They were all weakly correlated with the ground truth value. However, instead of discussing *why* and *how* these markers did not correlate well, the authors were bashing on the small p-values. They said

> "From the results, all markers correlated with the ground truth (p<0.001), but the correlation of our proposed method A was significantly higher than D and E; and non-significantly higher than B & C."

They even made this (horrendous) plot

![Barplot of Correlation Coefficient]({{ base.url }}/images/rbarplot.png)

This is completely a misleading statement of statistical significance test. Yes, $$p$$<0.001 is statistically significance, but for the existence of the reported $$R$$ value. It merely means there is a very small probability the $$R$$ value was found by chance. The $$R$$ values themselves are indeed significantly low. It should be the authors' interest - *as a scientist* - to investigate why they were low.

This case appears to be common in many fields. In medicine, psychology and economics, statistical significance has been their idol. Statistical significance has - *as Ziliak and McCoskey said* - cost jobs, justice and lives. Damage has been done, but we need to change. Do you know that [Basic and Applied Social Psychology Journal](http://www.tandfonline.com/toc/hbas20) is now [rejecting papers containing $$p$$-value](http://www.nature.com/news/psychology-journal-bans-p-values-1.17001)?

So let's forget about the $$p$$-value, because science depends on size or magnitude, not just existence. The existence does not have size. Therefore, please, **don't be a sizeless scientist**.
