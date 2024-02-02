# Udacity-AB-Testing-Final-Project
Final Project for Udacity A/B Testing Course

## Experiment Overview: Free Trial Screener
At the time of this experiment, Udacity courses currently have two options on the course overview page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. [This screenshot](https://drive.google.com/file/d/0ByAfiG8HpNUMakVrS0s4cGN2TjQ/view?resourcekey=0-6_dPu8BRM1XlRgV51nIbtA) shows what the experiment looks like.

The **hypothesis** was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## Metric Choice
The practical significance boundary for each metric, that is, the difference that would have to be observed before that was a meaningful change for the business, is given in parentheses. All practical significance boundaries are given as absolute changes.  

Any place "unique cookies" are mentioned, the uniqueness is determined by day. (That is, the same cookie visiting on different days would be counted twice.) User-ids are automatically unique since the site does not allow the same user-id to enroll twice.  

### Invariant Metrics
As screener triggers after click "Start free trial", we should expect cookies are evenly distributed in prior stages, and behaviors are the same between two groups. Therefore, below three metrcs are selected as invariant metrics.

- **Number of cookies**: That is, number of unique cookies to view the course overview page. (dmin=3000)
- **Number of clicks**: That is, number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger). (dmin=240)
- Click-through-probability: That is, number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page. (dmin=0.01)  

### Evaluation Metrics
Under the hypothesis, if the screener feature act as expected, we should observe test group has lower `Gross conversion` as the screener filtered out students don't have enough time for free trial. On the other hand we should observe test group has higher `Retention` as students who decided to proceed with free trial already awared of the time requested. In terms of `Net conversion`, we should observe that test group has equal/higher rate than control group. 

- **Gross conversion**: That is, number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button. (dmin= 0.01)
- **Retention**: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout. (dmin=0.01)
- **Net conversion**: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button. (dmin= 0.0075)  

## Measuring Standard Deviation
Since the unit of diversion is the same as the unit of analysis (denominator of the metric formula) for each evaluation metric (cookie in the case of Gross Conversion and Net Conversion and user-id in the case of Retention) and we can make assumptions about the distributions of the metrics (binominal), we can calculate the standard errors analytically (instead of empirically).
In order to calculate standard deviation, we need to convert n = 5000 cookies to scaled size for other unit of analysis.

| Unit of Analysis | Estimator | Scaled_Estimator |
|----------|----------|----------|
| Cookies | 40000 | 5000 |
| Clicks | 3200 | 400 |
| User-ids | 660 | 82.5 |
  
Under assumption of binomial distribution, we can apply below formula:
$$
SE = \sqrt{\frac{p(1-p)}{n}}
$$

| Evaluation Metric | Unit of Analysis | Scaled_Estimator | Standard Deviation |
|----------|----------|----------|----------|
| Gross Conversion | Clicks | 400 | 0.020231 |
| Retention	| User-ids | 82.5 | 0.054949 |
| Net conversion | Clicks | 400 | 0.015602 | 

## Sizing
### Number of Samples vs. Power
Bonferroni correction is not applied to this analysis, due to it could be too conservative, and we could analysis three evaluation metrics separately to draw conclusion.   
Use an alpha of 0.05 and a beta of 0.2, the number of pageviews total (across both groups) needed to power the experiment as below:  

| Evaluation Metric | Unit of Analysis | Estimator | dmin | Sample Size (cookies) |
|----------|----------|----------|----------|----------|
| Gross Conversion | Clicks | 0.206250 | -0.01 | 630725 |
| Retention	| User-ids | 0.530000 | 0.01 | 4733091 |
| Net conversion | Clicks | 0.109313 | 0.0075 | 699525 |

As we want to test all three metrics, sample sized needed is the maximum of the three, which is 4,733,091 cookies. 
	
### Duration vs. Exposure
If we divert 100% of traffic, given 40,000 page views per day, the experiment would take ~ 119 days. If we eliminate Retention, we are left with Gross Conversion and Net Conversion. This reduces the number of required pageviews to 699,525, and an ~ 18 day experiment with 100% diversion and ~ 35 days given 50% diversion.

A 119 day experiment with 100% diversion of traffic presents both a business risk (potential for: frustrated students, lower conversion and retention, and inefficient use of coaching resources) and an opportunity risk (performing other experiments). However, in general, this is not a risky experiment as the change would not be expected to cause a precipitous drop in enrollment. In terms of timing, an 18 day experiment is more reasonable, but % diversion may be scaled down depending on other experiments of interest to be performed concurrently.

## Experiment Analysis
### Sanity Checks
For invariant metrics, below table shows lower and upper bound with 95% confidence interval, the actual observed value, and whether the metric passes sanity check.  
Here all three invariant metrics passed sanity check.

| Invariant Metric | Control | Experiment | CI_lower | CI_upper | obs     | pass_sanity_check |
|-----------|------------|----------|----------|---------|-------------------|-------------------|
| Pageviews | 345543     | 344660   | 0.49882  | 0.50118 | 0.50064           | True              |
| Clicks    | 28378      | 28325    | 0.495885 | 0.504115| 0.500467          | True              |
| CTP       | 0.082126   | 0.082182 | -0.001296| 0.001296| 0.000057          | True              |

### Result Analysis
#### Effect Size Tests
From result data, we can see enrollment and payment columns only available for 23 days, versus pageview and clicks available for all 37 days.  
In this case, the true sample size is 423,525, which is lower than 699,525 we were targeting. But without other data points, we'll proceed the analysis base on 23 days' data.   
From below table we can see that `Gross Conversion` is both statistical and practial significant, while `Net Conversion` is not significant in both tests.

| Evaluation Metric | Control | Experiment | CI_lower | CI_upper | obs     | dmin | stats_significant | practical_significant |
|-----------|------------|----------|----------|---------|-------------------|-------------------|---------|---------|
| Gross Conversion | 0.2189 | 0.1983 | -0.0086 | 0.0086 | -0.0206 | -0.01 | Y | Y |
| Net Conversion   | 0.1176 | 0.1127 | -0.0067 | 0.0067 | -0.0049 |  0.0075 | N | N |

#### Sign Tests
Day-by-day sign test p-value was calculated using [this online calculator](https://www.graphpad.com/quickcalcs/binomial1.cfm).  
Days with positive change:
- Gross Conversion: days that experiment group has lower gross conversion rate than control group.
- Net Conversion: days that experiment group has higher net conversion rate than control group.

| Evaluation Metric | # of Days | # of Days with positive change | p-value for sign test | statistically significant at alpha=.05 |
|-------------------|-----------|--------------------------------|-----------------------|----------------------------------------|
| Gross Conversion | 23 | 19 | 0.0026 | Y |
| Net Conversion   | 23 | 13 | 0.6776 | N |

#### Summary
The Bonferroni correction has not been incorporated into this analysis. Typically used to mitigate Type I errors when examining multiple metrics and to prevent the incorrect rejection of null hypotheses in the event of rare statistical significance.  
In this experiment, all evaluations must not only be statistically significant but also practically significant to warrant recommendations. Therefore, the Bonferroni correction is intentionally omitted.

Furthermore, it is noteworthy that the results from both the effect size test and the sign test align in terms of statistical significance.

### Recommendation
From what we observed, experiment group has practially significant lower `Gross Conversion` than control group, and we can't conclude statistical or practical significance in `Net Conversion` between two groups.  
This aligns with the hypothesis that reducing the number of frustrated students who left the free trial because they didn't have enough time, without significantly reducing the number of students to continue past the free trial and eventually complete the course.  
So the recommendation would be launch the feature.

## Follow-Up Experiment - How to Reduce Early Cancellations
Here, we define "Early Cancellation" as students who enrolled 14 days free trial but didn't make the payment of the course. There're two phase might impact students' decision:
1. Pre-enrollment: where students might didn't set up corrcet expectation before enrollment, then left frustrated during free trail period. 
2. Post-enrollment and pre-payment: where we assume students already have right expectation, but still dropped, there're various potential reasons such as less engagement, poor course quality, financial difficulty, etc.

The experiment above already focused on one point for pre-enrollment, we'll discuss a potential follow-up experiment in post-enrollment and pre-payment period. Main difference between paid version and free version of course is project feedback, which would motivate students to keep learning and feel accomplished. I suggest to add notifications in free trail asking students submit at leat one project, in that case if they did submit one, they'll have chance to experience of practicing skills and getting feedbacks.  
**Setup**: We'll randomly split students who enrolled in free trail into two groups, send project notification to experiment group, and keep everything the same for control group.     
**Null Hypothesis**: Project notification will not increase the number of students enrolled beyond the 14 day free trial period by a significant amount.  
**Invariant Metrics**: Number of user-ids  
**Evaluation Metrics**: Rentention, a statistically and practically significant increase in Retention would indicate that the change is succesful.  
If a statistically and practically signifcant positive change in Retention is observed, assuming an acceptable impact on overall Udacity resources (on sending out the notifications), the experiment will be launched.

