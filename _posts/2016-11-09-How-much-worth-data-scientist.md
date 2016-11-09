
---
layout: post
title: GA Project week 4 - How much is worth a data scientist?
author: Gregory Vial
Categories: GeneralAssemblyProject
Tag: Web scapping, Logistic Regression, Salaries
---

How much is worth a data scientist?

There is no one who can tell you better... than a data scientist! That's probably what makes it the "sexiest job of the XXIst century" :)

In this week 4 project we explored the data science job market in the US.

### Approach

To perform this analysis we have selected a job posting site called [indeed.com](http://www.indeed.com). 

Pretty much I have extracted all the data science jobs being advertised on November 9th 2016 (which took 2+ hours) and for each ad I have recorded the recruiting company name, the location, the job title and the salary when it was available. As it turns out less than 10% of job ads mention the salary range.

However the website offers the option to search for a job with a minimum salary. So for each state, I have run searches multiple times, each time starting with very high salaries at $200,000 + a year, recording the outputs, then again running th search for jobs at $150,000 + a year, which obviously includes the former category of $200,000 + as well, so I recorded each job unique ID and removed duplicates as they appeared. This was repeated multiple times for each categories (which I have arbitrarily defined):
* A: Above 200,000 yearly
* B: Between 150,000 and 200,000
* C: Between 100,000 and 150,000
* D: Between 75,000 and 100,000
* E: Between 50,000 and 75,000
* F: Below 50,000

With this data at hand, I was then ready to play and see if I can make interesting findings.

### What are the median and 75th percentile salaries in US
Data showed that half of the data scientists in the US make more than $100,000 a year.
And a quarter make more than $150,000 !

If you are a data scientist, in the US, and you make less than $100,000, there is a good chance that you should walk to your boss office tomorrow and remind him how essential to the company you are!

### What are the most and least data scientist hungry states in the US?

Here we simply look at what proportion of the total data science jobs each state is currently recruiting.

This map shows the density of job seaches (the darker, the more searches, the whiter, the less)

![US_jobs_repartition](./assets/us_jobs_repartition.png)

5 states collectively hire 40% of the US data scientists. The highest recruiter is... California (what a surprise!) with 9% of total recruitement. Then come Massachussets, New York, DC and Washington.

Now the state where a data scientists has the least chances to get hired is South Dakota with 0,02% of the need in data scientists. Then come Alaska, Wyoming, North Dakota, Vermont.

### What are the most and least generous states with data scientist in the US?

Here is where you should move if you want to make more than \$200,000 as a data scientist:

![very_high](./assets/very_high_salaries.png)

Not much choice right? California is probably where you want to head off.

However if you are ready to make less than $50,000, you have a much wider choice:

![very_low](./assets/very_low_salaries.png)

But this time better avoid California! If you try hard you surely will find a data scientist job for less than $50,000, however this will definitely represent a bigger challenge than in Montana!



