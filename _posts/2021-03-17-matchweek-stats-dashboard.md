---
layout: post
title:  "FPL API - Matchweek Stats Dashboard"
author: "Eloy Chang"
categories: markdown
project_title: "fpl"
excerpt: "I started to ask myself how to improve my game decisions, then discovered the FPL API while investigating , now lets see what I can do."
path_image: <img class="header_post_image" src='../assets/img/fpl-header-post.png' alt="" />
---

Asking myself how can I improve my FPL performance the answer was very clear, I need data, but manually recollecting data is really hard work, then the question was straightforward, if there is a FPL or Premier League API? When I searched for it the answer was, yes, there is a FPL API, and is open.

To use the API I followed the work on [David Allen](https://towardsdatascience.com/fantasy-premier-league-value-analysis-python-tutorial-using-the-fpl-api-8031edfe9910) and [Frenzel Timothy](https://towardsdatascience.com/fantasy-premier-league-value-analysis-python-tutorial-using-the-fpl-api-8031edfe9910) posts.

I used python to extract and process the data, in this article I will just extract, but later I’ll post some nice projects I have in mind.

For downloading the data we will need 2 library, [requests](https://pypi.org/project/requests/) and [pandas](https://pandas.pydata.org/).

```
import requests
import pandas as pd
```

Now let's set the main api url, and make the call

```
url = 'https://fantasy.premierleague.com/api/bootstrap-static/'
r = requests.get(url)
json = r.json()
```

With these lines we get a python dictionary (variable json) with all the data from the API, those are 8 main sets.

To create a pandas data frame with one of this sets we can use:

```
elements = pd.DataFrame(json['elements'])
```

### Events

This has the information of the matchweeks (or gameweeks) like most selected player, best player, average and highest team score, etc. The variables in this set are:

**['id', 'name', 'deadline_time', 'average_entry_score', 'finished',  'data_checked', 'deadline_time_game_offset', 'highest_score', 'is_previous',  'is_current', 'is_next', 'chip_plays', 'most_selected', 'most_transferred_in', 'top_element', 'top_element_info', 'transfers_made', 'most_captained', 'most_vice_captained']**

### Game settings

This has information about the rules of the game, size of each squad and other stuff. This one cannot be converted into a data frame, but we will not use it anyway.

### Phases

Information about each phase of the game, each phase is a month, useful for track phase performance. The variables in this set are:

**['id', 'name', 'start_event', 'stop_event']**

### Teams

This has the information and stats of each premier league team. The variables in this set are:

**['code', 'draw', 'form', 'id', 'loss', 'name', 'played', 'points', 'position', 'short_name', 'strength', 'team_division', 'unavailable', 'win', 'strength_overall_home', 'strength_overall_away', 'strength_attack_home', 'strength_attack_away', 'strength_defence_home', 'strength_defence_away', 'pulse_id']**

### Total Players

This is just how many players (users) are in the game.

### Elements

This is the most important set, with all information and stats of each player in the premier league, there are a bunch of columns like name, last gameweek points, total points, value, goals, assists, minutes played, etc.  The variables in this set are:

**['chance_of_playing_next_round', 'chance_of_playing_this_round', 'code', 'cost_change_event', 'cost_change_event_fall', 'cost_change_start',  'cost_change_start_fall', 'dreamteam_count', 'element_type', 'ep_next', 'ep_this', 'event_points', 'first_name', 'form', 'id', 'in_dreamteam', 'news', 'news_added', 'now_cost', 'photo', 'points_per_game', 'second_name', 'selected_by_percent', 'special', 'squad_number', 'status', 'team', 'team_code', 'total_points', 'transfers_in', 'transfers_in_event', 'transfers_out', 'transfers_out_event', 'value_form', 'value_season', 'web_name', 'minutes', 'goals_scored', 'assists', 'clean_sheets', 'goals_conceded', 'own_goals', 'penalties_saved', 'penalties_missed', 'yellow_cards', 'red_cards', 'saves', 'bonus', 'bps', 'influence', 'creativity', 'threat', 'ict_index', 'influence_rank', 'influence_rank_type', 'creativity_rank', 'creativity_rank_type', 'threat_rank', 'threat_rank_type', 'ict_index_rank', 'ict_index_rank_type', 'corners_and_indirect_freekicks_order', 'corners_and_indirect_freekicks_text', 'direct_freekicks_order', 'direct_freekicks_text', 'penalties_order', 'penalties_text']**

### Elements Stats

These are just human readable names of each stat saved for the players. The variables in this set are:

**['label', 'name']**

### Elements Type

The definition of each position for players, those are: goalkeeper, defenders, midfielders and forwards. The variables in this set are:

**['id', 'plural_name', 'plural_name_short', 'singular_name', 'singular_name_short', 'squad_select', 'squad_min_play', 'squad_max_play', 'ui_shirt_specific', 'sub_positions_locked', 'element_count']**

### Others

There are other sets with different endpoints of the API, but we will not use it in this post.

### Downloading the data

For this post we just want to make a descriptive dashboard of the last gameweek, so we will extract the elements set and replace the team id and position.

The code is:

```
url = 'https://fantasy.premierleague.com/api/bootstrap-static/'
r = requests.get(url)
json = r.json()

elements = pd.DataFrame(json['elements'])
elements["value_season"] = elements.value_season.astype(float)
elements["total_points"] = elements.total_points.astype(int)
elements["cost"] = elements.total_points / elements.value_season

elements_types = pd.DataFrame(json['element_types'])[["id", "singular_name"]].set_index("id")
elements["position"] = elements.element_type.map(elements_types.singular_name)
elements.drop(columns = "element_type", inplace = True)

teams = pd.DataFrame(json['teams'])[["id", "name"]].set_index("id")
elements["team"] = elements.team.map(teams.name)
```

Now to save it we use the datetime library to get the date and use it in the name of the file, in this case a csv file.

```
from datetime import datetime
elements.to_csv("data/tableau_{}.csv".format(datetime.now().strftime("%Y%m%d")), index = False, sep = ";")
```

Finally we use Tableau Public to make our dashboard, here is the results:

<body>
<div class='tableauPlaceholder' id='viz1616257006579' style='position: relative'><noscript><a href='https:&#47;&#47;bit.ly&#47;3liI6YB'><img alt=' ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fa&#47;FantasyPremierLeague_16157556653850&#47;MatchweekStats&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='FantasyPremierLeague_16157556653850&#47;MatchweekStats' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fa&#47;FantasyPremierLeague_16157556653850&#47;MatchweekStats&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='es' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1616257006579');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else { vizElement.style.width='100%';vizElement.style.height='2177px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>
</body>

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
