---
layout: post
title: DIC vs AIC vs BIC
comments: true
---

Recently, I reviewed a paper written by my colleague. He computed *Deviance Information Criterion* (DIC), *Akaike Information Criterion* (AIC), *Bayesian Information Criterion* (BIC) values to quantify how good the performance of different logistic regression models he created to distinguish between normal and patients (in cardiac domain). Not going to show his results here, but I'm curious to why there are so many measurements to pick one best model.

It's all boiled down to the trade-off between *goodness-of-fit* and *model complexity*. A good model fits well with minimum variables. With increasing complexity, the model becomes so well fit to the data, but ...

> "you can always fit an elephant with thousands of parameters."

So statisticians came up with a metric that *penalizes* complexity.

The most famous metric is called **AIC**, which commonly translates into *Akaike Information Criterion*, named after Hirotogu Akaike, a Japanese statistician. It's funny though that AIC was first introduced by Akaike as *An Information Criterion*. He argued that the model selection should be penalized by the number of parameters $$k$$:

$$
\mathrm{AIC} = -2\ln\{ p(x | \hat{\theta}) \} + 2k
$$

where $$x$$ is the random variable and $$\hat{\theta}$$ is the maximum likelihood estimate. *Note that for all information criteria, the smaller the better.*

This criterion was so simple that was accepted by statisticians not from its rigorous theoretical development, but by being used in many applications. [Akaike commented][AkaikeComments]:

> It is the general applicability and simplicity of model selection by AIC that prompted its use in such diversified areas as hydrology, geophysics, engineering, econometrics, psychometrics, and medicine. The procedure has some proof of its optimality. Nevertheless, due to its nonconventional style, AIC is not yet fully accepted by professional statisticians. It is mainly the increasing number of successful applications that caused the frequent citation of the paper.

Akaike's [original 1974 paper][Akaike1974] has been cited 14,000+ times according to Web of Science.

The second most used criterion is known as **BIC** or *Bayesian Information Criterion*; some called it *Schwarz Information Criterion* after Gideon Schwarz, who adopted a model selection with Bayesian argument in [his 1978 paper][Gideon1978] (11,000+ citations). Schwarz derived BIC to approximate the Bayesian posterior probability of a candidate model. It is defined as

$$
\mathrm{BIC} = -2\ln\{ p(x | \hat{\theta}) \} + k\ln n
$$

where $$n$$ is the sample size. Notice that only the penalty term differs between AIC and BIC.

Hence, BIC is more stringent than AIC for larger sample size, as the penalty increases when the number of observations is getting larger although the number of parameters remains constant.

Now, how big is the BIC penalty term compared to AIC?

{% gist avansp/d3bcb6da7da98d47e71b AIC_vs_BIC.r %}

![AIC vs BIC penalty terms]({{ base.url }}/images/bic_vs_aic.png)

The BIC penalty dramatically increases with only a few number of samples. BIC becomes larger than AIC when the sample size is $$n > \exp(2)$$ or $$n > 7$$ (dotted horizontal line).

The last metric is **DIC**, which stands for *Deviance Information Criterion*, developed by David Spiegelhalter [in 2002][Spiegelhalter2002]. He defined the complexity as a measure of effective numbers of parameters, which is

$$
p_D = \mathbf{E}\left[ -2\ln\{ p(x | \theta) \} ) \right] + 2\ln\{ p(x | \hat{\theta}) \}
$$

It is equivalent to say that the effective number of parameters is the posterior mean deviance minus the deviance measured at the posterior mean of the parameters. Putting into the same equation as AIC and BIC, then

$$
\mathrm{DIC} = -2\ln\{ p(x | \hat{\theta}) \} + p_D
$$

It's a bit more complicated. Spiegelhalter acknowledged [12 years later][Spiegelhalter2012] that DIC has some problems:

1. the penalty term is invariant to reparameterization,
2. lack of consistency,
3. it is not based on a proper predictive criterion, and
4. it has weak theoretical foundation.

It's interesting to read how his paper was rejected and then was accepted at the same journal one year later. He commented:

> The authors of the 2002 DIC-paper still feel that DIC was a good idea but have always been aware of its limitations. We are somewhat surprised that it has lasted so well, but in spite of its problems we are confident that it has benefited a large number of researchers in a range of substantive fields. In our personal experience, when it has come out with idiotic results such as a negative $$p_D$$, then this has acted as an appropriate warning of issues in the modelling that need to be addressed.

Another side note from DIC is how criticisms he received from statisticians. One that I like was expressed by Jim Smith (University of Warwick):

> I shall not address technical inaccuracies but just present four foundational problems that I have with the model selection in this paper .... So my suggestion to a practitioner would be: if you must use a formal selection criterion do not use DIC.

Nevertheless, Spiegelhalter's 2002 paper has been already cited 3,000+ times.

As a final note, are there any other alternatives? Yes, a lot, but I prefer to stick with AIC or BIC. The difference is whether you want to include sample size as a penalty for model complexity, or not.

[AkaikeComments]: http://www.garfield.library.upenn.edu/classics1981/A1981MS54100001.pdf
[Akaike1974]: http://dx.doi.org/10.1109/TAC.1974.1100705
[Gideon1978]: http://dx.doi.org/10.1214/aos/1176344136
[Spiegelhalter2002]: http://dx.doi.org/10.1111/1467-9868.00353
[Spiegelhalter2012]: http://dx.doi.org/10.1111/rssb.12062
