---
layout: post
title:  "Gameweek Dreamteam"
author: "Eloy Chang"
categories: markdown
project_title: "fpl"
excerpt: "Is the dreamteam the best team every gameweek? Now I will compare the gameweek dreamteam with the FPL dreamteam"
path_image: <img class="header_post_image" src='../assets/img/fpl-header-post.png' alt="" />
---

## New Changes

Last time I publish a Tableau Public Dashboard visualizing top players by position, and the development of the FPL Dreamteam, but it was very clear that the best player on the gameweek wasn't part of the dream team, so I decide to create a alternative dreamteam using exclusive the results of the gameweek, I also take this opportunity to improve the code used to get the data to the dashboard and even start another dashboard.

The great change I have made is create a class where the magic happens, so my main file has left as this:

```
from sys import argv
from tableau_data_class import tableau_data

if __name__ == '__main__':

    data = tableau_data(argv[1])

    data.generate_gameweek_dreamteam()

    data.write_data()
```

In this case I use the argv function of sys library to allow me pass a argument when executing the file, in this case I used the first argument as the gameweek number to paste on the file name so if I want to generate a file named “gameweek_29.csv” the right command would be:

python tableau.py 29

The `tablea_data` object is the class name I have created.

## About classes

Classes are the structure or the template of objects that may be created in python or any other object oriented code language.
A class has attributes that are related to the data stored in it and methods that are functions declared in the classes, these methods may alter or use the attributes of the class or even interact with other classes.

As I said before, a class is only a template, creating an object using this template is called instantiation, when an instance of a class is made the special method `__init__` is used, all class definitions must have this method, with this the basic structure of the class is made.

You can add as many methods as you need for your class and define how it interacts with other objects and functions, by example, if you need your objects to be added, then you must first define the method `__add__`.

## Tableau Data Class

My tableau data class instantiation method is almost all that the tableau.py file did before, made the FPL API endpoint call, and created the elements dataframe, this time as an attribute of the class, and filtering just the columns used on the dashboard.

```
import requests
import pandas as pd

class tableau_data:

    def __init__(self, gameweek):
        self.gameweek = gameweek

        url = 'https://fantasy.premierleague.com/api/bootstrap-static/'
        r = requests.get(url)
        json = r.json()

        columns = ["code", "first_name", "second_name", "event_points", "element_type", "team", "in_dreamteam", "total_points", "value_season"]
        self.elements = pd.DataFrame(json['elements'])[columns]
        self.elements["value_season"] = self.elements.value_season.astype(float)
        self.elements["total_points"] = self.elements.total_points.astype(int)
        self.elements["cost"] = self.elements.total_points / self.elements.value_season
        self.elements.drop(columns = ["value_season", "total_points"], inplace = True)

        elements_types = pd.DataFrame(json['element_types'])[["id", "singular_name"]].set_index("id")
        self.elements["position"] = self.elements.element_type.map(elements_types.singular_name)
        self.elements.drop(columns = "element_type", inplace = True)

        teams = pd.DataFrame(json['teams'])[["id", "name"]].set_index("id")
        self.elements["team"] = self.elements.team.map(teams.name)
```

Nice. Now, let’s go and create the gameweek dreamteam.

First things first, what is the dreamteam and how to assemble it, I defined the dreamteam as the combinations of 11 players that maximize the score that a FPL user can get, keeping this in mind, some rules must apply:
The formation must respect the rules this means, the team must have a goalkeeper, at least 3 defenses, at least 2 midfielders and at least 1 forward, and cannot be more of 5 defenses, 5 midfielders or 3 forwards.
There cannot be more than three players of the same team.
The team price may be added as a restriction, but the budget of a team changes over time, so, in this instance the budget will not be added as a constraint.  

This will be accomplished with the generate_gameweek_dreamteam method:

```
def generate_gameweek_dreamteam(self):
    dreamteam = []
    # Goalkeeper
    dreamteam.append(self._get_top_players_by_postition("Goalkeeper").code[0])

    # Best minimal quantity of player by each position
    positions = ["Forward", "Midfielder", "Defender"]
    topPlayers = pd.DataFrame()
    for aux in range(1,4):
        topPositionPlayers = self._get_top_players_by_postition(positions[aux - 1])
        for x in topPositionPlayers.code[:aux]: dreamteam.append(x)
        topPlayers = topPlayers.append(topPositionPlayers[aux:])

    dreamteam = self.elements.loc[self.elements.code.isin(dreamteam)]

    # Fill remaining spots
    topPlayers.sort_values(by = ["event_points", "cost"], ascending = False, inplace = True)
    postion_limit = {"Defender": 5, "Midfielder": 5, "Forward": 3}
    for _, player in topPlayers.iterrows():
        if (dreamteam.team == player.team).sum() >= 3 or (dreamteam.position == player.position).sum() >= postion_limit[player.position]:
            continue
        dreamteam = dreamteam.append(player)
        if dreamteam.shape[0] == 11:
            break

    print(dreamteam)

    # Add column to main DataFrame
    self.elements["gw_dreamteam"] = self.elements.code.isin(dreamteam.code)
```

The strategy followed was, first get the top 10 players by position, this with two goals:
* Reduce the size of the dataframe to work with.
* Secure players from all positions taking into account that some player from any position could be dismissed due to the rule of maximum three players from the same club.

The top players by each position are obtained with the following method.

```
def _get_top_players_by_postition(self, position):
        topPlayers = self.elements.loc[self.elements.position == position]
        topPlayers["cost"] *= -1
        return topPlayers.sort_values(by = ["event_points", "cost"], ascending = False).head(10).reset_index(drop = True)
```

After the top players were selected then the positions are filled as follow:
* First the top players data frame is sorted by event points and as a tiebreaker, the player cost.
* Then the top player data frame is iterated, and by each player the validation of position and max team players are made.
* If the player passed the validation then is added to the team, else the next player is validated.
* After each player is added a extra validation is made, if the dreamteam have already eleven players then the loop is breaked.

With this is secure the team will accomplish all but one of the rules, which is, using this method a formation of 1 goalkeeper, 5 defends and 5 midfielders is allowed, but, this is an event with very low odds.

Finally, we add the column `gw_dreamteam` to the main data frame that is true if the player is in the gameweek dreamteam, false if is not.

And add the information to the new dashboard.

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
