---
layout: post
title:  "MTG - Winning Rate"
author: "Eloy Chang"
categories: markdown
project_title: "mtg"
excerpt: "I wanted to know how was my winning rate on ranked matches, and if I have some weakness again a specific color."
path_image: <img class="header_post_image" src='../assets/img/magic-header-post.png' alt="" />
---

Magic The Gathering (MTG) is a popular fantasy card game, in each game we represent planeswalkers, some powerful beings capable of invoke creatures and cast all sorts of spells, and our objective is to defeat others planeswalkers in combat.

Each combat is made by turns, in these we can cast spells and creatures with mana, which come from lands we have played before, basic lands give us one mana of one specific color:

* Plains generate white mana.
* Islands generate blue mana.
* Swamps generate black mana.
* Mountains generate red mana.
* Forests generate green mana.

In general MTG may be relatively easy to start, but is a really complex game, with a lot of mechanics and cards, and one of the principal attractiveness is not only in combat, but building your own deck to play, choose colors, cards and even balance the number of spells cards and lands cards and mana cost levels. This makes probability a huge factor in MTG gameplay.

MTG Arena is the digital version of the game, it maintains almost all characteristics of game excepts the trading part of it. But, in general, it is a great game to start playing and test strategy and combinations, also it has its own competitive mode, making it a full MTG experience, one of the latest competitions had a pool of $250.000 in prizes.

## Data Gathering

I wanted to know how was my winning rate on ranked matches, so I started to record the games results of them in a very simple table with just three columns:

* **id**: Identification of the row.
* **day**: Gameday, this is an integer that group all games played on the same day.
* **result**: 1 if I win, 0 if I lost.

This was enough to calculate my winning rate, then I started to believe that my deck has a weakness against blue-black decks, but I had no way to verify it, so, I started to record a lot more of information, the new columns where:

* **date**: Date when the games happends.
* **game_type**: I started to record not just ranked matches, but all matches, like casual and events like Friday Night Magic (FNM) and drafts.
* **deck**: Which deck I use, this is the name of my deck.
* **colors**: Colors used on my deck.
* **opponent_colors**: Colors used on opponent deck.
* **turns**: How many turns were taken on the game.
* **mulligans**: How many mulligans I took (Mulligans is when a player does not like their opening hand and draws a new one but with one less card).
* **opponent_mulligans**: How many mulligans my opponent took.  
* **tier**: The tier of competitive games.

With this information I may answer my question and a lot more questions, like my performance in short/long games, even more, I can start to approximate which color are more used in each game type and tier, by example, blue is a common color I encounter in casual games, but is a very rare color in ranked matches, the opposite happens with white decks, it’s more frequently found in ranked matches that in casual ones, by other side, black is a favorite in both scenarios, and the white - black is the most popular combination.

Was a surprise to me, while analysing mulligans by color that green is the color with greater percent of mulligans, when is very characteristic of this color of a lot of creatures and spells that let you find mana and call greater creatures at early stages of the game.

Finally the average game takes 6 turns, however, this result is biased because I was playing in every game.

## Tableau Public dashboard

<body>
  <div class='tableauPlaceholder' id='viz1615658701884' style='position: relative'><noscript><a href='https:&#47;&#47;echang1802.github.io&#47;epsilon.github.io&#47;'><img alt=' ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ma&#47;MagicTheGathering&#47;WinningRate&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='MagicTheGathering&#47;WinningRate' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ma&#47;MagicTheGathering&#47;WinningRate&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='es' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1615658701884');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else { vizElement.style.width='100%';vizElement.style.height='1877px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>
</body>

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../mtg.html#masthead' | replace: '..', site.url }}">Back to MTG project main page</a></p>
      </div>
</div>

<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/mtg_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
