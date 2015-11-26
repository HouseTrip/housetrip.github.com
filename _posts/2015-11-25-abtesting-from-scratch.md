---
layout: post
title: AB Testing, from scratch
published: true
author: Alfredo Motta
author_role: Software Engineer
author_url: http://www.alfredo.motta.name
author_avatar: http://www.gravatar.com/avatar/c1044117a60a9c37a232cf8b6e2c87a8.png
summary: |
  If you work in a diligent web development business you probably know what an A/B test is. However, its fascinating statistical theory is usually left behind. Understanding the basics can help you avoid common pitfalls, better design your experiments, and ultimately do a better job in improving the effectiveness of your website.

  Please hold tight, and enjoy a pleasant statistical journey with the help of R and some maths. You will not be disappointed.
---

An A/B test is a randomized, controlled experiment in which the <em>performance</em> of two product variants are compared. Those variants are usually called A and B [<a href="#footnotes">1</a>]. From a business perspective we want to know if the <em>performance</em> of a certain variant outperforms the other.

As an example let's take a typical checkout page where we want to assess if a green checkout button is more effective than an orange one.

<a href="/images/2015-11-25-abtesting-from-scratch/abtestingexample.png"><img src="/images/2015-11-25-abtesting-from-scratch/abtestingexample.png" alt="Experiment example" width="400" height="257" style="box-shadow: none; border: none"/></a>

After 1 week we may collect the following numbers:
<a name="conversion_table"></a>

<a href="/images/2015-11-25-abtesting-from-scratch/table.png"><img src="/images/2015-11-25-abtesting-from-scratch/table.png" alt="Experiment example" width="600" style="box-shadow: none; border: none"/></a>

If at this point you are willing to conclude that the B variant outperforms A be aware you are taking a very naïve approach. Results can vary on a weekly basis because of the intrinsic randomness of the experiment. Put simply, you may be plain wrong. A more thorough approach would be to estimate the likelihood of B being better than A given the number we measured, and statistics is the best tool around for this kind of job.

## Statistical modeling

Statisticians love urns and, guess what, our problem can be modeled as an extraction from two different urns. Both urns have a certain ratio of red and green balls. Red balls are customers who end up paying while green balls are customers who leave.

Does the urn belonging to variant B have a greater proportion of red balls compared to the other one? We can estimate this ratio by doing an extraction with replacement [<a href="#footnotes">2</a>] from the urn. We extract a finite number of balls from each urn and we measure the proportions. In this type of experiment each time we pick a ball we also put it back to the urn to keep the proportions intact.

<div>
<div style="float: left;">

<a href="/images/2015-11-25-abtesting-from-scratch/populationA.png"><img src="/images/2015-11-25-abtesting-from-scratch/populationA-300x300.png" alt="" width="300" height="300" style="box-shadow: none; border: none"/></a>

</div>
<div style="float: left;">

<a href="/images/2015-11-25-abtesting-from-scratch/populationB.png"><img src="/images/2015-11-25-abtesting-from-scratch/populationB-300x300.png" alt="" width="300" height="300" style="box-shadow: none; border: none"/></a>

</div>
<div style="clear: both;"></div>
</div>

Now the good news is that the Binomial distribution [<a href="#footnotes">3</a>] models exactly this type of experiment. It will tell you the expected number of successes in a sequence of _n_ independent yes/no experiments, each of which yields success with probability _p_.

In other words, each time we extract n balls from our urn we are conducting an experiment, and a yes corresponds to a red ball while a no corresponds to a green ball. The binomial distribution [<a href="#footnotes">4</a>] computes the probability of picking exactly k red balls after n extractions as follows:

<a id="probability_mass_function"></a>
<a href="/images/2015-11-25-abtesting-from-scratch/binomial_formula.png"><img src="/images/2015-11-25-abtesting-from-scratch/binomial_formula.png" alt="" style="box-shadow: none; border: none"/></a>

which can be checked using the _dbinom()_ function in R. Assuming we have a urn with _30%_ of red balls, what is the probability that we extract exactly _10_ red balls out of a _100_? This is equivalent to _Pr(X=10)_ which is:

<pre class="lang:r decode:true ">dbinom(10, 100, 0.3)
# =&gt; 1.17041796785404e-06</pre>

Very small. Fair enough, the urn has definitely a bigger proportions of red and I was the unlucky guy here. Now, let's plot these values for x (number of successes) ranging between 0 and 100:

<pre class="lang:r decode:true" title="Probability mass function of the binomial distribution">
x =  1:100
y = dbinom(x, 100, 0.3)

options(repr.plot.width=7, repr.plot.height=3)
qplot(x, y, xlab="Number of successes", ylab="Probability") + xlim(0, 60)
</pre>

<a href="/images/2015-11-25-abtesting-from-scratch/binomial_example.png"><img src="/images/2015-11-25-abtesting-from-scratch/binomial_example.png" width="600" style="box-shadow: none; border: none"/></a>

Makes sense, doesn't it? The chances of having exactly k successes cumulates around the value of 30, which is the true proportion of red/green balls in our urn.

<h2>Naïve experiment assessment</h2>
Now that we know how statisticians model our problem let's go back to our <a href="#conversion_table">conversions table</a>.

One way of assessing if B is better than A is to plot their expected distributions. <em>Assuming that </em>A follows a Binomial distribution with <em>p=0.01</em> (we had 100 conversions over 10000 trials) and that B follows a Binomial distribution with <em>p=0.012</em> (we had 120 conversions over 10000 trials) this is how they relate to each other:

<pre class="lang:r decode:true">
x_a =  1:10000
y_a = dbinom(x_a, 10000, 0.01)

x_b =  1:10000
y_b = dbinom(x_b, 10000, 0.012)

data = data.frame(x_a=x_a, y_a=y_a, x_b=x_b, y_b=y_b)

options(repr.plot.width=7, repr.plot.height=3)
cols = c("A"="green","B"="orange")
ggplot(data = data)+
    labs(x="Number of successes", y="Probability") + xlim(0, 200) +
    geom_point(aes(x=x_a, y=y_a, colour="A")) +
    geom_point(aes(x=x_b, y=y_b, colour="B")) +
    scale_colour_manual(name="Variants", values=cols)
</pre>

<a href="/images/2015-11-25-abtesting-from-scratch/ab_testing_binomial_distributions.png"><img src="/images/2015-11-25-abtesting-from-scratch/ab_testing_binomial_distributions.png" width="600" style="box-shadow: none; border: none"/></a>

So <em>if the true mean </em>of the two distribution is <em>p_a = 0.01</em> and <em>p_b = 0.012</em> respectively we can conclude that B is better than A. If we repeat the experiment several times (always with 10000 participants) for A we will get <em>most of the time </em>values between <em>70</em> and <em>129</em> while for B we will get values between <em>87</em> and <em>152</em>. You can check these boundaries on the plot, or you can compute them manually with the <em>3 times standard deviation </em>rule of thumb [<a href="#footnotes">5</a>].

<pre class="lang:r decode:true">
n=10000; p=0.01; q=1-p; mean=100
paste(mean - 3 * sqrt(n*p*q), "," ,mean + 3 * sqrt(n*p*q))

n=10000; p=0.012; q=1-p; mean=120
paste(mean - 3 * sqrt(n*p*q), ",", mean + 3 * sqrt(n*p*q))
</pre>

But hold on one minute. How do we know that <em>p_a = 0.01</em> and <em>p_b = 0.012</em> are indeed the <em>true means</em> of our distributions? In the end we only did one extraction from our urns. If these numbers are wrong our distributions will look different and the previous analysis will be flawed. Can we do better?

## More rigorous experiment assessment

In order to estimate what is the true mean of our variants statisticians rely on the Central Limit Theorem (CLT) [<a href="#footnotes">6</a>] which states that <em>the sampling distribution of any statistic</em> <em>will be normal or nearly normal, if the sample size is large enough</em>.

In our case we are trying to estimate the mean of the sampling distribution of the proportions <em>p_a</em> and <em>p_b</em> for our variants. Suppose you run your A/B test experiment <em>N=100</em> times and that each time you collect <em>n=10000</em> samples you will end up having for variant A:

<a href="/images/2015-11-25-abtesting-from-scratch/clt1.png"><img src="/images/2015-11-25-abtesting-from-scratch/clt1.png" style="box-shadow: none; border: none"/></a>

and the CLT tells us that these are normally distributed with parameters:

<a href="/images/2015-11-25-abtesting-from-scratch/clt2.png"><img src="/images/2015-11-25-abtesting-from-scratch/clt2.png" style="box-shadow: none; border: none"/></a>

where <a href="/images/2015-11-25-abtesting-from-scratch/clt3.png"><img src="/images/2015-11-25-abtesting-from-scratch/clt3.png" style="display: inline; box-shadow: none; border: none; height: 24px"/></a> is the standard deviation of the binomial distribution. I know, this is hard to believe and proving these numbers goes definitely beyond the scope of this blog post so you will find some maths-heavy material in the footnotes [<a href="#footnotes">7</a>] [<a href="#footnotes">8</a>].

Back to our question, what is the true mean of <em>p_a</em> and <em>p_b</em>? Well, we don't know really, but they are distributed normally like this<a name="mean_distributions"></a>:

<pre class="lang:r decode:true">
x_a = seq(from=0.005, to=0.02, by=0.00001)
y_a = dnorm(x_a, mean = 0.01, sd = sqrt((0.01 * 0.99)/10000))

x_b = seq(from=0.005, to=0.02, by=0.00001)
y_b = dnorm(x_b, mean = 0.012, sd = sqrt((0.012 * 0.988)/10000))

data = data.frame(x_a=x_a, y_a=y_a, x_b=x_b, y_b=y_b)
options(repr.plot.width=7, repr.plot.height=3)
cols = c("A"="green","B"="orange")
ggplot(data = data)+
    labs(x="Proportions value", y="Probability Density Function") +
    geom_point(aes(x=x_a, y=y_a, colour="A")) +
    geom_point(aes(x=x_b, y=y_b, colour="B")) +
    scale_colour_manual(name="Variants", values=cols)
</pre>

<a href="/images/2015-11-25-abtesting-from-scratch/clt4.png"><img src="/images/2015-11-25-abtesting-from-scratch/clt4.png" width="600" style="box-shadow: none; border: none"/></a>

As you can see we are dealing with a risky business here. There is a good chance that the estimation of the <em>true values </em>of <em>p_a</em> and <em>p_b</em> are not correct since they can span anywhere between all the values plotted above. For a good number of them <em>p_a</em> may actually outperform <em>p_b</em> violating the conclusion we did in the previous section. There is no magic bullet that will solve this problem, this is the intrinsic nature of the probabilistic world in which we live. However, we can do our best to quantify the risk and take a conscious decision.

## Quantitative evaluation

In the previous section we have seen that <i>it is likely</i> that Variant B is better than Variant A, but how can we quantify this statement? There are different ways in which we can look at this problem, but the ones that statisticians use is <em>Hypothesis testing.</em>

In a series of papers in the early 20th century, J. Neyman and E. S. Pearson developed a decision-theoretic approach to hypothesis-testing [<a href="#footnotes">9</a>]. The theory was later extended and generalised by Wald [<a href="#footnotes">10</a>]. For a full account of the theory, see the book of Lehmann and Romano [<a href="#footnotes">11</a>].

In this framework we state a hypothesis (also called <em>null-hypothesis</em>) and by looking at the number we will try to reject it. In our example we hypothesize that the true conversion of our visitors is <em>p_a</em> and that the proportion <em>p_b</em> we collected in the B variant is simply due to chance. In other words we assume that our real world visitors behave like in variant A and we quantify the probability of seeing variant B's proportions under this hypothesis.

So, what is the probability of having <em>120</em> or more conversions if the true mean of the binomial distribution is <em>p_a = 0.01</em>? We simply have to sum the probability of all possible events:

<a href="/images/2015-11-25-abtesting-from-scratch/quantitative_evaluation1.png"><img src="/images/2015-11-25-abtesting-from-scratch/quantitative_evaluation1.png" style="box-shadow: none; border: none"/></a>

To actually compute the number you can use the probability mass function in <a href="#probability_mass_function">Formula (a)</a> or you can use R<a name="binomial_test"></a>:

<pre class="lang:r decode:true">
binom.test(120, 10000, p = 0.01, alternative = "greater")
</pre>

which gives us the following:

<pre class="lang:r decode:true">
Exact binomial test

data:  120 and 10000
number of successes = 120, number of trials = 10000, p-value = 0.0276
alternative hypothesis: true probability of success is greater than 0.01
95 percent confidence interval:
 0.01026523 1.00000000
sample estimates:
probability of success 
                 0.012
</pre>

I deliberately specified <em>alternative = "greater"</em> in the function call to compute the chance of getting more than 120. But there are other ways [<a href="#footnotes">12</a>] to approach the problem. The value we are looking for is the p-value [<a href="#footnotes">13</a>] <em>0.0276</em> which is exactly the probability of getting more than 120 successes, i.e. <em>P(X_a &gt;= 120)</em>. Visually it corresponds to the area under the right end tail of the distribution of A:

<pre class="lang:r decode:true">
x_a =  1:10000
y_a = dbinom(x_a, 10000, 0.01)

data = data.frame(x_a=x_a, area=append(rep(0, 119), seq(from=120, to=10000, by=1)), y_a=y_a)

options(repr.plot.width=7, repr.plot.height=3)
ggplot(data = data)+
    labs(x="Number of successes", y="Probability") + xlim(50, 150) +
    geom_point(aes(x=x_a, y=y_a)) + geom_area(aes(x=area, y=y_a), colour="green", fill="green")
</pre>

<a href="/images/2015-11-25-abtesting-from-scratch/quantitative_evaluation2.png"><img src="/images/2015-11-25-abtesting-from-scratch/quantitative_evaluation2.png" style="box-shadow: none; border: none" width = "600"/></a>

We are now ready to quantify to what degree our <em>null-hypothesis </em>is true or false according to the numbers we collected in our experiment.

## Type I and Type II errors

It is well known that statisticians do not have the same talent in the art of giving names as computer scientists do.  This is well proved by the definition of <em>Type I</em> and <em>Type II</em> errors which are equivalent to the Machine learning definitions of <em>False positive </em>and <em>False negative</em>.

A <em>Type I </em>error is the probability of rejecting the null-hypothesis when the null-hypothesis is true. In our example this happens when we conclude there is an effect in our AB test variant, while in reality there is none.

A <em>Type II </em>error is the probability of failing to reject the null-hypothesis when the null-hypothesis is false. In our example this happens when we conclude the AB test variant has no effect, while in reality it actually did.

In the previous section we quantified the probability of <em>Type </em><em>I </em>error, which is equivalent to the p-value returned by the <em>binom.test</em> function. To quantify the <em>Type II </em>error we need to know for what <em>pvalue = &alpha;</em> we are willing to reject the <em>null-hypothesis.</em> It is common practice to use <em>&alpha; = 0.05</em> so I'll just go with that.

Now, what is the critical number of conversions such that we reject the <em>null-hypothesis </em>at <em>&alpha; = 0.05</em>? This is called the quantile [<a href="#footnotes">14</a>] and it is simply the <em>x_&alpha;</em> such that:

<a href="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_1.png"><img src="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_1.png" style="box-shadow: none; border: none"/></a>

which can also be computed easily in R with the following:

<pre class="lang:r decode:true">alpha = 0.05
qbinom(1 - alpha, 10000, 0.01)
# =&gt; 117</pre>

Now we know that starting from <em>117 + 1</em> observations we will reject the null-hypothesis.

To compute the <em>Type II </em>error we need to assume that the null-hypothesis is false (<em>p_b = 0.012</em>) and quantify how likely it is to measure a value of <em>117</em> or less since this is exactly what will lead us to a mistake!

<a href="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_2.png"><img src="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_2.png" style="box-shadow: none; border: none"/></a>

which is equivalent to:

<pre class="lang:r decode:true">
pbinom(117, 10000, 0.012)
# =&gt; 0.414733285324603
</pre>

which means that we have a <em>~= 40%</em> chance to conclude our experiment did not have any effect, while in reality there was some.

That's seems harsh to me. What can we do? Apply the first law of the modern age of statistics which is, <em>get more data. </em>Assuming we can double our data and get <em>20.000</em> visitors instead of <em>10000</em> and that we measure the same proportions, our <i>Type II </i>error will go down drammatically<a name="type_ii_error_increase_population"></a>:

<pre class="lang:r decode:true">qbinom(0.95, 20000, 0.01) # critical value at which we reject the null-hypothesis
# =&gt; 223

pbinom(223, 20000, 0.012) # type II error
# =&gt; 0.141565461885161
</pre>

now we have <em>~= 14%</em> chance of concluding our experiment did not have any effect. We can even go one step further and see for every possible observation count what's the expected <em>Type II error</em> by simply brute forcing the computation as follows:

<pre class="lang:r decode:true">
v = c(); n = 1000:50000
for(i in n) {
    critical = qbinom(0.95, i, 0.01)
    t2_error = pbinom(critical, i, 0.012)
    v = append(v, t2_error)
}

options(repr.plot.width=7, repr.plot.height=3)
qplot(n, v, xlab="P(type II error)", ylab="# Observations")
</pre>

<a name="type_ii_error_for_increasing_size"></a>

<a href="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_3.png"><img src="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_3.png" width="600" style="box-shadow: none; border: none"/></a>

which seems a fair result, starting from <em>30.000</em> visitors we can safely assume that our probability or <em>Type II</em> error will be low.

The analysis as I just presented it is flawed by a fundamental assumption. To estimate the <em>Type I</em> error we assumed that <em>p_b</em> is exactly <em>0.012</em> while to estimate the <em>Type II </em>error we assumed that <em>p_a</em> is exactly <em>0.01</em>. We know from the CLT that this is not exactly true and the distributions of those values span across a certain range (see <a href="#mean_distributions">here</a>). So let's see what happens if I take several points across these distributions, for example the 1%, 25%, 50%, 75%, 99% quantiles and check what happens to our hypothesis testing errors.

For <em>Type II </em>errors I first collect the possible values of <em>p_a</em> for my parametric analysis:

<pre class="lang:r decode:true ">
mean = 0.01
sigma = sqrt((mean * 0.99)/10000)
p_a_values = c(
    qnorm(0.01, mean = mean, sd = sigma),
    qnorm(0.25, mean = mean, sd = sigma),
    qnorm(0.50, mean = mean, sd = sigma),
    qnorm(0.75, mean = mean, sd = sigma), 
    qnorm(0.99, mean = mean, sd = sigma))
p_a_values
</pre>

and then I estimate the error exactly like I did previously:

<pre class="lang:r decode:true ">
# parametric Type II
count = 50000; start = 1000
data = data.frame(x= numeric(0), error= numeric(0), parametric_mean = character(0))
p_a_values = factor(p_a_values)

for(p_a in p_a_values) {
    n = start:(start+count)
    x = rep(0, count); error = rep(0, count); parametric_mean = rep('0', count);
    for(i in n) {
        p_a_numeric = as.numeric(as.character(p_a))
        critical = qbinom(0.95, i, p_a_numeric)
        t2_error = pbinom(critical, i, 0.012)
        
        index = i - start + 1
        x[index] = i
        error[index] = t2_error
        parametric_mean[index] = p_a
    }
    data = rbind(data, data.frame(x = x, error = error, parametric_mean=parametric_mean))
}

options(repr.plot.width=7, repr.plot.height=3)
ggplot(data=data, aes(x=x, y=error, color=parametric_mean, group=parametric_mean)) +
    geom_line()
</pre>

which produces the following:

<a href="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_4.png"><img src="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_4.png" width="600" style="box-shadow: none; border: none"/></a>

where the green line is the same as the one plotted <a href="#type_ii_error_increase_population">here</a>. It is quite interesting to observe the pink line at <em>p_a = 0.0123</em>. This is the worst case scenario for us since we picked a value that is greater than <em>p_b = 0.012</em> and because of that our <em>Type II </em>error actually increases with the number of observations! However it is also worth mentioning that this value is very unlikely because the more data I collect the more the uncertainty around the values of <em>p_a</em> decrease. You may also notice that the lines in the graph are quite thick, and this is because discrete tests like the binomial one [<a href="#footnotes">15</a>] have quite large oscillations [<a href="#footnotes">16</a>].

The same type of parametric analysis can be performed on the <i>Type I </i>error. <a href="#binomial_test">Previously</a> we saw how to compute it with the <em>binom.test()</em> function, but we can do the computation manually as follows when <em>p_a=0.01</em> and <em>p_b=0.012</em>:

<pre class="lang:r decode:true ">
pbinom(119, 10000, 0.01, lower.tail=FALSE)
</pre>

where <em>119</em> is the value starting from which we fail to reject the null-hypothesis. The parametric analysis is a generalisation of this code like we did for the <i>Type II </i>error:

<pre class="lang:r decode:true ">
# parametric Type I
count = 50000
start = 1000
data = data.frame(x= numeric(0), error= numeric(0), parametric_mean = character(0))
p_b_values = factor(p_b_values)

for(p_b in p_b_values) {
    n = start:(start+count)
    x = rep(0, count); error = rep(0, count); parametric_mean = rep('0', count);
    for(i in n) {
        p_b_numeric = as.numeric(as.character(p_b))
        expected_b = i * p_b_numeric
        t1_error = pbinom(expected_b - 1, i, 0.01, lower.tail=FALSE)
        
        index = i - start + 1
        x[index] = i
        error[index] = t1_error
        parametric_mean[index] = p_b
    }
    data = rbind(data, data.frame(x = x, error = error, parametric_mean=parametric_mean))
}

options(repr.plot.width=7, repr.plot.height=3)
ggplot(data=data, aes(x=x, y=error, color=parametric_mean, group=parametric_mean)) +
    geom_line()
</pre>

which plots the following:

<a href="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_5.png"><img src="/images/2015-11-25-abtesting-from-scratch/type_i_and_ii_5.png" width="600" style="box-shadow: none; border: none"/></a>

as in the <em>Type II </em>error we notice that for <em>p_b = 0.009</em> the <em>Type I </em>error actually increase with the amount of data but this value become more and more unlikely as the data grows.

It also very interesting to notice how the value of the two type of errors goes down at a completely different rate. Overall, with this design, we are more likely to stick with the "<em>button colour makes no difference</em>" conclusion. When the reality is that button colour makes no difference, the tests will stick to reality most of the times (<i>Type I error goes down quickly</i>). When the reality is that button colour does make a difference, the test does take the risk of saying there is actually no difference between the two (<em>Type II error goes down slowly</em>).

<h2>Estimate the sample size</h2>
Before wrapping up let's make a step back and position ourself back in time before we started the experiment. How we can estimate for how long we should run our experiment in order to be confident that our results are statistically significant? To answer this question you simply need to use the same tools we just saw, from a different perspective.

First, you need to make an estimate around your current baseline conversion. If you use <a href="https://analytics.google.com">google analytics</a> it should be straightforward to know what is the conversion of your checkout page with the green button.

Second, you need to make a guess on the <em>effect size</em> you are willing to detect (<em>minimum detectable effect size</em>). In our example we would have chosen an effect size of 20%. In various disciplines effect sizes have been standardised to make different experiments comparable. Most famous effect size measure is <em>Cohen's d </em>[<a href="#footnotes">17</a>] [<a href="#footnotes">18</a>].

Third, you need to assert what risk you are willing to take on the <em>Type I error</em>, or equivalently, what <em>&alpha;</em> level you are willing to choose for your <em>p-value</em>.<em> </em>This is the value you will refer to at the end of the experiment to make your conclusion.

Finally, you need to assert what risk you are willing to take on the <em>Type II error</em>, or equivalently, how likely you are willing to say your orange button did not perfomed better when in reality there was an effect (i.e. orange button is better). This is equivalent to the <em>power</em>, where you assert how likely you are going to be correct when you conclude that the orange button is better, when it is actually real.

Mathematically speaking computing the required sample size seems to be a difficult problem in general and I would like to point you to the footnotes for a deeper discussion [<a href="#footnotes">19</a>] [<a href="#footnotes">20</a>]. Here I will show the approach taken from the Engineering statistics handbook [<a href="#footnotes">21</a>].

First, we look at the statistics:

<a href="/images/2015-11-25-abtesting-from-scratch/sample_size1.png"><img src="/images/2015-11-25-abtesting-from-scratch/sample_size1.png" style="box-shadow: none; border: none"/></a>

and we would like to compute the sample size <em>n </em>such that:

<p style="text-align: center;">
(1) P(reject H0 | H0 is true) = &alpha; <br>
(2) P(reject H0 | H0 is false) = <em>power</em> = 1 - &beta;
</p>

Now, you have to use some faith and intuition. What is the smallest value of &delta;, say &delta;_min, that we care about? Our minimum detectable effect size! You can imagine &delta;_min is a function of both (1) and (2). The smallest value in (1) for which we start rejecting is:

<a href="/images/2015-11-25-abtesting-from-scratch/sample_size2.png"><img src="/images/2015-11-25-abtesting-from-scratch/sample_size2.png" style="box-shadow: none; border: none"/></a>

while the smallest value in (2) for which we start rejecting is:

<a href="/images/2015-11-25-abtesting-from-scratch/sample_size3.png"><img src="/images/2015-11-25-abtesting-from-scratch/sample_size3.png" style="box-shadow: none; border: none"/></a>

where <em>power = 1 - &beta;</em> putting it all together we have:

<a href="/images/2015-11-25-abtesting-from-scratch/sample_size4.png"><img src="/images/2015-11-25-abtesting-from-scratch/sample_size4.png" style="box-shadow: none; border: none"/></a>

Take the N out of this equation and you get:

<a href="/images/2015-11-25-abtesting-from-scratch/sample_size5.png"><img src="/images/2015-11-25-abtesting-from-scratch/sample_size5.png" style="box-shadow: none; border: none"/></a>


So, using our example we state that: (i) our baseline conversion for the green button is <em>1%</em> or <em>p_a = 0.01</em>; (ii) our estimate of effect size is <em>20%</em> which leads to the orange button converting at <i>1.2%</i> or <em>p_b = 0.012</em>; (iii) we accept a <em>Type I error</em> probability of <em>5%</em> or <em>&alpha; = 0.05</em>; (iv) we want a <i>Power </i>of <em>80%</em> to make sure we detect the effect when it is there.  This is translated in R as follows:

<pre class="lang:r decode:true">
p_a = 0.01
p_b = 0.012
alpha = 0.05
beta = 0.2

delta = abs(p_a - p_b)

t_alpha = qnorm(1 - alpha)
t_beta = qnorm(1 - beta)

sd1 = sqrt(p_a * (1-p_a))
sd2 = sqrt(p_b * (1-p_b))

n = ((t_alpha * sd1 + t_beta * sd2) / abs(p_a - p_b))^2

# =&gt; ~= 16294
</pre>

which seems to be consistent with the simulations we did <a href="#type_ii_error_for_increasing_size">previously</a>. On the other hand it is 30% lower than the one reported by a fairly famous online tool [<a href="#footnotes">22</a>] [<a href="#footnotes">23</a>]. If you know why please let me know in the comments.

<h2>Wrapping up</h2>
In this blog post we took a simple conversion table and we did a step by step statistical analysis of our results. Our experiment is designed like an extraction with replacement from an urn which is modelled with a binomial distribution.

We concluded that given the observed data the checkout page with the orange button is 20% more likely (<em>(0.012 - 0.01) / 0.01</em>) to convert than the one with the green button. If the green button is equivalent to the orange button (no effect), we came to the wrong conclusion with a probability of <em>2.76%</em> (<em>P(type I error)</em>, also called <em>p-value</em>). If the orange button is better than the green button, we came to the right conclusion with a probability of <em>60%</em> (<em>1 - P(type II error)</em>, also called the <em>power</em>). In this same scenario (orange button is better) if we double the number of visitors in our experiment our probability of being correct (power) will go up to <em>86%</em>. Finally, we reviewed how to estimate the required sample size for our experiment in advance with a handy formula.

It is worth mentioning that in practice you rarely use a binomial test but I belive the simplicity of the distribution makes it adequate for this type of presentation. A better resource on choosing the right test can be found in the references [<a href="#footnotes">24</a>].

If you enjoyed this blog post you can also <a href="https://twitter.com/mottalrd">follow me</a> on twitter. Let me know your comments!

<h2>References</h2>

<a name="footnotes"></a>

<ol class="easy-footnotes-wrapper"><li class="easy-footnote-single"><span id="easy-footnote-bottom-1" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/A/B_testing">Wikipedia page on A/B testing</a><a class="easy-footnote-to-top" href="#easy-footnote-1"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-2" class="easy-footnote-margin-adjust"></span><em><a href="https://en.wikipedia.org/wiki/Urn_problem">Extraction with replacement</a></em><a class="easy-footnote-to-top" href="#easy-footnote-2"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-3" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/Binomial_distribution">Binomial distribution</a><a class="easy-footnote-to-top" href="#easy-footnote-3"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-4" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/Probability_mass_function">Probability mass function</a><a class="easy-footnote-to-top" href="#easy-footnote-4"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-5" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/68%E2%80%9395%E2%80%9399.7_rule">3 times standard deviation rule of thumb</a><a class="easy-footnote-to-top" href="#easy-footnote-5"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-6" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/Central_limit_theorem">Central Limit Theorem</a><a class="easy-footnote-to-top" href="#easy-footnote-6"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-7" class="easy-footnote-margin-adjust"></span><a href="http://stattrek.com/sampling/sampling-distribution.aspx">Sampling Distributions</a><a class="easy-footnote-to-top" href="#easy-footnote-7"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-8" class="easy-footnote-margin-adjust"></span><a href="http://projecteuclid.org/euclid.ss/1177013818">The Central Limit Theorem around 1935</a><a class="easy-footnote-to-top" href="#easy-footnote-8"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-9" class="easy-footnote-margin-adjust"></span><a href="http://www.jstor.org/stable/2331945">Neyman and Pearson, Biometrika, 1928</a><a class="easy-footnote-to-top" href="#easy-footnote-9"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-10" class="easy-footnote-margin-adjust"></span><a href="http://www.jstor.org/stable/2235609?seq=1#page_scan_tab_contents">Wald, The Annals of Mathematical Statistics, 1939</a><a class="easy-footnote-to-top" href="#easy-footnote-10"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-11" class="easy-footnote-margin-adjust"></span><a href="http://www.springer.com/gb/book/9780387988641">Lehmann and Romano, Springer Texts in Statistics, 2005</a><a class="easy-footnote-to-top" href="#easy-footnote-11"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-12" class="easy-footnote-margin-adjust"></span><a href="http://onlinestatbook.com/2/logic_of_hypothesis_testing/tails.html">One-tailed vs two-tailed tests</a><a class="easy-footnote-to-top" href="#easy-footnote-12"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-13" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/P-value">p-value on wikipedia</a><a class="easy-footnote-to-top" href="#easy-footnote-13"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-14" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/Quantile">Quantile on Wikipedia</a><a class="easy-footnote-to-top" href="#easy-footnote-14"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-15" class="easy-footnote-margin-adjust"></span><a href="https://en.wikipedia.org/wiki/Binomial_test">Binomial test on Wikipedia</a><a class="easy-footnote-to-top" href="#easy-footnote-15"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-16" class="easy-footnote-margin-adjust"></span><a href="http://stats.stackexchange.com/questions/44125/is-there-an-error-in-the-one-sided-binomial-test-in-r">Binomial test error oscillations on CrossValidated</a><a class="easy-footnote-to-top" href="#easy-footnote-16"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-17" class="easy-footnote-margin-adjust"></span><a href="http://www.polyu.edu.hk/mm/effectsizefaqs/thresholds_for_interpreting_effect_sizes2.html">Thresholds for interpreting effect sizes</a><a class="easy-footnote-to-top" href="#easy-footnote-17"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-18" class="easy-footnote-margin-adjust"></span><a href="http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3444174/">Using Effect Size—or Why the P Value Is Not Enough</a><a class="easy-footnote-to-top" href="#easy-footnote-18"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-19" class="easy-footnote-margin-adjust"></span><a href="http://stats.stackexchange.com/questions/91630/book-recommendation-sample-size-determination-for-hypothesis-testing-of-the-mea">Sample size determination for hypothesis testing of means</a><a class="easy-footnote-to-top" href="#easy-footnote-19"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-20" class="easy-footnote-margin-adjust"></span><a href="http://stats.stackexchange.com/questions/21237/calculating-statistical-power/21243#21243">Calculating statistical power</a><a class="easy-footnote-to-top" href="#easy-footnote-20"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-21" class="easy-footnote-margin-adjust"></span><a href="http://www.itl.nist.gov/div898/handbook/prc/section2/prc242.htm">Engineering statistics handbook - sample sizes required</a><a class="easy-footnote-to-top" href="#easy-footnote-21"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-22" class="easy-footnote-margin-adjust"></span><a href="http://www.evanmiller.org/ab-testing/sample-size.html">Evan Awesome AB Tools</a><a class="easy-footnote-to-top" href="#easy-footnote-22"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-23" class="easy-footnote-margin-adjust"></span><a href="https://gist.github.com/mottalrd/7ddfd45d14bc7433dec2">Evan Miller source code for sample size</a><a class="easy-footnote-to-top" href="#easy-footnote-23"></a></li><li class="easy-footnote-single"><span id="easy-footnote-bottom-24" class="easy-footnote-margin-adjust"></span><a href="http://www-users.cs.umn.edu/~ludford/stat_overview.htm">An Overview: choosing the correct statistical test</a><a class="easy-footnote-to-top" href="#easy-footnote-24"></a></li></ol>

