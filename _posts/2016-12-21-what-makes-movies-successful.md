---
layout: post
title: GA Project week 6 - Analysing Internet Movie DataBase
author: Gregory Vial
Categories: GeneralAssemblyProject
Tag: IMBD, tree
---

## Background
Over the past decades, the amount of video content available to watch has literally boomed. 

From a handful TV channels in the eighties, people now have the choice between hundreds of TV channels, over 1 billion YouTube videos and nearly the whole history of movies and series on Netflix or other paying streaming platforms.

In this context, standing out as a content producer or finding out what to watch as a "consumer" has become harder and harder. As a result recommendations systems have become crucial in this new content jungle. It helps improve visibility of content which results in more views, and helps the public to identify more easily content they will enjoy.

But one question one might ask in the first place is obviously, what makes a movie successful?

## Goals and method
In this week's project, we intend to analyse the criteria that impact movie ratings.

IMDB (Internet movie database) is a website launched in 1996 where internet users can rate and comment movies, TV series and other video contents such as documentaries.

Based on the list of the 250 best rated movies on IMDB, plus a user list of 1000 best movies, plus some movies picked randomly in IMDB, we will observe various features and identify which ones play a role in the movie sucess. We intend to identify a few of these features in order of importance, and produce a recommendation for film producers so they know how to enhance their future movies ratings.

The features we will study are:
* year of release
* duration (in minutes)
* genre (Drama, Romance etc)
* main actor
* director

## Risks and assumptions
It is worth noting that IMBD belongs to Amazon, a company that intends to become a major player on the streaming market, and therefore a direct competitor of Netflix (just to name this one).

As a content provider, ratings provided by Amazon may be biased since the company has an interest in highly rating movies so they are watched by more people, and hence generating more revenue.

Such a biased has been observed and well documented for the Fandango review web site, see the article on FiveTheiryEight [website](http://fivethirtyeight.com/features/fandango-movies-ratings/) 

Other data sources may also provide different movie ratings and therefore the studied information should be considered not as the sole version of the truth, but rather a biased version of it.

Finally, we assume that the information recorded and made available to us by IMBD, in particular the movie features (director, actors, date of release etc) are accurate.

## Results
Let's look at a few features from our movie set.

#### Movies by rating
<img src="/assets/movies_by_rating.png">
The data is clearly skewed to the right, i.e. we have much more movies with a rating higher than the average theoretical rating (5).

This was expected since most our movies are part of "best of" lists

#### Movies by duration
<img src="/assets/movies_by_duration.png">
The median value is 120 minutes, i.e. two hours. Movies are fairly normally distributed around this value.

#### Duration by genre
<img src="/assets/duration_by_genre.png">
There are clear differences between genres, with some running usually around 1 hour 30 minutes (Film Noir, Animation) whereas others are closer to 2 hours 30 minutes (western)

#### Most popular actors
<img src="/assets/most_pop_actor.png">

#### Most popular directors
<img src="/assets/most_pop_director.png">

#### Predicting very high scores (above 8) with a decision Tree
Accuracy: 0,77

F1-score: 0,77

Confusion Matrix
<table>
  <tr>
    <th></th>
    <th>predicted < 8</th>
    <th>predicted >= 8</th>
  </tr>
  <tr>
    <td>is < 8</td>
    <td>255</td>
    <td>37</td>
  </tr>
  <tr>
    <td>is >= 8</td>
    <td>53</td>
    <td>51</td>
  </tr>
</table>

#### Predicting very high scores (above 8) with a decision boosting Tree
Accuracy: 0,85

F1-score: 0,82

Confusion Matrix
<table>
  <tr>
    <th></th>
    <th>predicted < 8</th>
    <th>predicted >= 8</th>
  </tr>
  <tr>
    <td>is < 8</td>
    <td>292</td>
    <td>0</td>
  </tr>
  <tr>
    <td>is >= 8</td>
    <td>60</td>
    <td>40</td>
  </tr>
</table>


## Interpretation

We built two models that try to predict whether the IMDB score of the movie is going to be equal or higher than 8, or lower.

Both models perform reasonably well at this job. Let's look at them in more details.

#### Simple decision tree
The simple decision tree has the advantage of leading to a very interpretable result. However its prediction capacities are slightly below the boosting tree.

The tree looks like this:
<img src="/assets/DecisionTreeClassifier_optimized.png">

This models manages to identify about 50% of the highly rated movies. But when it predicts a high rated movies, 40% of the time is is wrong.

Similarly, it catptures 87% of the low rated movies.But in 17% of the cases it is wrong, and the move actually scored higher.

#### Boosring tree
Boosting tree unfortunately doesn't allow easy interpretation. It cannot be mapped under a form that can simply be understood.

However its results are superior. Basically in our movies list, it was able to predict with 100 confidence when a movies would not be rated 8 or more. So it is good at telling us which movies get lower ratings.

It does a slightly poorer job at capturing very high rated movies. When it predicts one, it is actually right, however it fails to catpure nearly 60% of them.


## Conclusion
The models we build were extremely simple and yet allowed to flag with a fairly good accuracy which movies would score more or less than 8 in IMDB.

The time dedicated to the project didn't allow us to use more precious information that could potentially be a strong predictors for success of a movie. Primarily, budget information is the one that comes to mind. But other information such as tagline analysis may be useful too.

Also it would be good to analyse a much larger movie base. Here we study 1200 movies, including a large proportion that were well rated. It should be possible to come up with a finer analysis by scrapping more movies, and more diverse as well.
