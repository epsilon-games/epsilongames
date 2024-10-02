---
layout: post
title:  "Impact of international break on the performance of Premier League players."
author: "Eloy Chang"
categories: markdown
project_title: "fpl"
excerpt: "How does the international break effect on players performance? Does players with international duty improve their performance after the international break?"
path_image: <img class="header_post_image" src='../assets/img/fpl-header-post.png' alt="" />
---

Today professional football is very demanding, with an intense schedule all over the years, many clubs not only compete in their countries leagues and cup, but also in international tournaments. In addition, many players have responsibilities with their nationals teams, so, how does this affect players' performance? Do players with international duties improve, or decrease, their performance after an international break?

To answer this we’ll use the point system of the [Fantasy Premier League](https://fantasy.premierleague.com/) (FPL), and find if there's a difference between the performance of players with international duties and players without it.

The FPL has an API service, which is updated at real time. We use this API to consume the points of each player in each Premier League match. The points granted to each player are in function of minutes played, goals scored, clean sheet, cards, bonuses, etc. so, these points are a good proxy to players performance.

We also use the [Transfermarket](https://www.transfermarkt.com/premier-league/nationalspieler/wettbewerb/GB1) data, to view which players have international duties.

In this analysis we will use the international break between September 1 and September 10, which is exactly between the matchweek 3 and 4 of the premier league.

As a first try to answer our question we approach a solution using a Difference In Difference (DID) approach, this is, compute the difference between the performance of players with and without international duty, before and after the international break, then compute the difference of this differences, with this analysis no evidence was found to say that there is a dramatic change of performance within the two groups, there were a decrease of just 5.46% on average, nevertheless, in overall all performance after the international break were lower than before, this could mean that, no matter if a player play, or not, with they national team, in average, his performance will be lower after the international break.

![DID]({{ '../assets/img/DID_played_False.png' | replace: '..', site.url }})

But in this case we are counting a lot of players that, even if they are called for their international teams, they did not play any game with then, so, we re-run the analysis, but in the group we called “international players” added only the players that actually played at least one minute with their national team.

In this case the difference was highly notable, the decrease between the performances difference was of 34.58% on average, this may be evidence that playing this kind of matches may decrease the performance of the player with his club, even more, players that did not play during the international break has seen a slighter boost on they performance after it.

![DID upgrade]({{ '../assets/img/DID_played_True.png' | replace: '..', site.url }})

Nevertheless the previous result, we know that a DID analysis is not too robust, so, we apply a matching experiment, which consist in form pairs of players, one that played with his national team, and one who did't, but this players must be similar in some way. To find this similarity a propensity score was applied, this score estimate the player probability of belonging to the internationals players group.

This propensity score was computed using a logistic regression based on different parameters of the players like the total points over the season, number of goals scored, yellow cards, total bonuses points etc. After each player has his propensity score, the pairs were formed securing that both players has a similar propensity score, this way, we have, two player that, we know, one is an international and the other is not, but they probability to be a international player is similar.

A total of 128 pairs were found, them, applying a T test with 95% of confidence, the results were a statistic of 0.06928 and a p-value of 0.94481, so, we can't conclude that their is a difference over the difference of performance between the two groups.

![Distribution]({{ '../assets/img/Distribution of total points difference.png' | replace: '..', site.url }})

Finally we can't conclude that there is a difference between the performance of players with international duty and the players without it, nevertheless, there is a decrease on overall performance after the international break, but this is only one international break, so, more examples are needed for a better conclusion, in the same line, a improved system to measure players performance would improve the analysis accuracy.

All the code used for this analysis may be found on [Github](https://github.com/echang1802/bada_mim).

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
