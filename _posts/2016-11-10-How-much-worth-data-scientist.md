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

To perform this analysis I have selected a job posting site called [indeed.com](http://www.indeed.com). 

Pretty much I have extracted from the site all the data science job ads being advertised on November 9th 2016 (which took 2+ hours) and for each ad I have recorded the recruiting company name, the location, the job title and the salary when it was available. As it turns out less than 10% of job ads mention salary information.

However the website offers the option to search for a job with a minimum salary. So for each state, I have run searches multiple times, each time starting with very high salaries at $200,000 + a year, recording the outputs, then again running the search for jobs at $150,000 + a year, which obviously includes the former category of $200,000 + as well, so I recorded each job unique ID and removed duplicates as they appeared. This was repeated multiple times for each categories (which I have arbitrarily defined):
* A: Above 200,000 yearly
* B: Between 150,000 and 200,000
* C: Between 100,000 and 150,000
* D: Between 75,000 and 100,000
* E: Between 50,000 and 75,000
* F: Below 50,000

With this data at hand, I was then ready to play and see if I can make interesting findings.

### How many Data Scientist job are vacant in the US currently?
My web scaping exercise returned 12,394 outputs. Yep, at the time I write this blog, in the US alone and on one job posting site only, there are more than 12,000 data scientist jobs being advertised.

That makes me confident that at the end of my 12 weeks immersive I should get at least one of them!

### What are the median and 75th percentile salaries in US?
Data showed that half of the data scientists in the US make more than $100,000 a year.
And a quarter make more than $140,000 !

If you are a data scientist, in the US, and you make less than $100,000, there is a good chance that you should walk to your boss office ASAP and remind him how essential to the company you are!

### What are the most and least data scientist hungry states in the US?

Here we simply look at what proportion of the total data science jobs each state is currently recruiting.

This map shows the density of job ads (the darker, the more searches, the whiter, the less)

<center><b>Repartition of data scientists job ads in the US</b> </center>
<img src="/assets/us_jobs_repartition.png">

5 states collectively hire 40% of the US data scientists. The highest recruiter is... California (what a surprise!) with 9% of total recruitement. Then come Massachussets, New York, DC and Washington.

Now the state where a data scientists has the least chances to get hired is South Dakota with 0,02% of the need in data scientists. Then come Alaska, Wyoming, North Dakota, Vermont.

In addition, here are the top 10 cities recruiting:
<table style="width:40%" align="center">  
<tr>
    <th>New York, NY</th>
    <th>564</th>
</tr>
<tr>
    <th>Seattle, WA</th>
    <th>          496</th>
  </tr>
<tr>
    <th>Cambridge, MA </th>
    <th>       234</th>
  </tr>
<tr>
    <th>Boston, MA  </th>
    <th>         233</th>
  </tr>
<tr>
    <th>Chicago, IL  </th>
    <th>        203</th>
  </tr>
<tr>
    <th>San Francisco, CA</th>
    <th>    187</th>
  </tr>
<tr>
    <th>Washington, DC </th>
    <th>      166</th>
  </tr>
<tr>
    <th>Los Angeles, CA </th>
    <th>     163</th>
  </tr>
<tr>
    <th>St. Louis, MO </th>
    <th>       147</th>
  </tr>
<tr>
    <th>Atlanta, GA  </th>
    <th>        132</th>
  </tr>
</table>

### What are the most and least generous states with data scientist in the US?

Here is where you should move if you want to make more than \$200,000 as a data scientist:

<center><b>Proportion of data scientists job ads paid $200,000+ per state</b> </center>
<img src="/assets/very_high_salaries.png">

Not much choice right? California is probably where you want to head off. Unless you prefer the cold winters of New York, New Jerey, Pennsylvania or Massachusetts.  Pretty much everywhere else, such jobs don't exist!

However if you are ready to make less than $50,000, you have a much wider choice:

<center><b>Proportion of data scientists job ads paid  less than $50,000 per state</b> </center>
<img src="/assets/very_low_salaries.png">

But this time better avoid California! If you try hard you surely will find a data scientist job for less than $50,000, however this will definitely represent a bigger challenge than in Montana!

### Top companies recruiting data scientists in the US

Finally I want to present the companies hiring the highest number of data scientists.

<table style="width:60%" align="center">  
<tr>
    <th><em>Top 10 Companies</em> </th>
    <th><em>Hires</em></th>
    <th><em>Top 20 Companies</em></th>
    <th><em>Hires</em></th>
</tr>
<tr>
    <th>Amazon Corporate LLC </th>
    <th>         253</th>
    <th>AstraZeneca</th>
    <th>                    65</th>
</tr>
<tr>
    <th>Ball_Aerospace </th>
    <th>               164</th>
        <th>Beckman Coulter, Inc.</th>
    <th>          65</th>
  </tr>
<tr>
    <th>ADP </th>
    <th>                          104</th>
        <th>Vencore</th>
    <th>                        62</th>
  </tr>
<tr>
    <th>Booz Allen Hamilton</th>
    <th>            94</th>
        <th>Microsoft</th>
    <th>                      61</th>
  </tr>
<tr>
    <th>ICF  </th>
    <th>                          90</th>
        <th>GlaxoSmithKline</th>
    <th>                57</th>
  </tr>
<tr>
    <th>Capital One  </th>
    <th>                  85</th>
        <th>NYU Langone Medical Center</th>
    <th>    54</th>
  </tr>
<tr>
    <th>HDR</th>
    <th>                            76</th>
        <th>The Aerospace Corporation</th>
    <th>      52</th>
  </tr>
<tr>
    <th>Leidos </th>
    <th>                        71</th>
        <th>Google</th>
    <th>                         51</th>
  </tr>
<tr>
    <th>BOEING </th>
    <th>                        71</th>
        <th>MONSANTO</th>
    <th>                       48</th>
  </tr>
<tr>
    <th>CDK Global </th>
    <th>                    70</th>
        <th>Engility Corporation</th>
    <th>           48</th>
      </tr>
</table>

After checking this list, I hope no one will still believe that data science is a tech company thing. Yes, Amazon, Microsoft and Google are in the top 20, but you also have pharmaceutical, finance, aeronautics etc.

### Additional comments
In this project I experimented the python package "vincent" which helps draw great maps, unfortunately I was not able to find (yet) how to print the scale. Apologies for not complying with this best practice for this post!

If you want to see the project code, you can check it on my <a href="https://github.com/GregVial/DSI_LDN_1_HOMEWORK/tree/master/greg/project4">github repo</a>. 


