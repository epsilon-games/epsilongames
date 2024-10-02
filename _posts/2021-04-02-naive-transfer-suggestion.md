---
layout: post
title:  "Naive Transfer Suggestion"
author: "Eloy Chang"
categories: markdown
project_title: "fpl"
excerpt: "It's time to make data based transfers to improve the team value of the season."
path_image: <img class="header_post_image" src='../assets/img/fpl-header-post.png' alt="" />
---

## FPL is Back!

The Premier League is back tomorrow, and I have updated my FPL project to suggest to me which transfer should I do for this gameweek. In a first iteration the suggestion will be made based only on the value over the season of players and, obviously, their cost.

For this purpose I created a new class named `fpl_class`, the `__ini__` method of it is almost the same as the `tableau_data` class explained in my last post, download the elements data frame.

```
class fpl_class:

    def __init__(self):
        # get elements info
        url = 'https://fantasy.premierleague.com/api/bootstrap-static/'
        r = requests.get(url)
        json = r.json()
        self.elements = pd.DataFrame(json['elements'])[["code", "element_type", "first_name", "second_name", "team", "value_season", "total_points"]]
        self.elements["value_season"] = self.elements.value_season.astype(float)
        self.elements["total_points"] = self.elements.total_points.astype(int)
        self.elements["cost"] = self.elements.total_points / self.elements.value_season

        elements_types = pd.DataFrame(json['element_types'])[["id", "singular_name"]].set_index("id")
        self.elements["position"] = self.elements.element_type.map(elements_types.singular_name)
        self.elements.drop(columns = "element_type", inplace = True)

        teams = pd.DataFrame(json['teams'])[["id", "name"]].set_index("id")
        self.elements["team"] = self.elements.team.map(teams.name)
```

## Create The Team

But, to suggest a transfer the code needs to know the available players on the team, so, I created an auxiliary file, `create_team`, this file creates and saves a data frame with all players in my team “Moka FC” (Yes, I’m a coffee lover).

The create_team file is like this:

```
import pickle
import requests
import pandas as pd
from sys import argv

url = 'https://fantasy.premierleague.com/api/bootstrap-static/'
r = requests.get(url)
json = r.json()
elements = pd.DataFrame(json['elements'])[["code", "element_type", "first_name", "second_name", "team", "value_season", "total_points"]]
elements["value_season"] = elements.value_season.astype(float)
elements["total_points"] = elements.total_points.astype(int)
elements["cost"] = elements.total_points / elements.value_season

elements_types = pd.DataFrame(json['element_types'])[["id", "singular_name"]].set_index("id")
elements["position"] = elements.element_type.map(elements_types.singular_name)
elements.drop(columns = "element_type", inplace = True)

teams = pd.DataFrame(json['teams'])[["id", "name"]].set_index("id")
elements["team"] = elements.team.map(teams.name)

team_players = [98980, 218023, 177815, 153366, 87873, 118748, 59859, 97299, 171314, 141746, 195473, 78830, 85971, 101982, 55459]
bank = 0.1

team = elements.loc[elements.code.isin(team_players)]
print(team)

with open("data/{}".format(argv[1]), "wb") as file:
    pickle.dump((team, bank), file)
```

The only argument is the name of the file, in this instance the codes of each player needs to be known from before, which is not the best, but I will improve it in the future.

Now I can create my team with just one line command

`python create_team.py mokaFC`

## Make Transfer Suggestion

The player suggestion need to respect some rules:

* The incoming player has to be of the same position that the leaving player.
* The incoming player has a greater `value_season` than the leaving player.
* The incoming player cost needs to be lower than the leaving player cost plus the money on the team bank.
* The incoming player must not be already in the team.
* The incoming player cannot be of a club with three players in the team, unless one of these players is the leaving player.

The strategy tooked was, iterate for each position, for each player in the team in that position find which transfer generates the greater increase in `value_season`, and select the one that is best, finally I select the best transfer in the team.

This is made with the method `suggest_transfer`.

```
def suggest_transfer(self):
        transfers = {}
        for pos in self.team.position.unique():
            transfers[pos] = {"out": None, "in": None, "improvement": 0}
            for _, player_out in self.team.loc[self.team.position == pos].iterrows():
                self._get_banned_teams(player_out)
                candidates = (self.elements.position == pos) & (self.elements.value_season >= player_out.value_season) & (self.elements.cost <= (player_out.cost + self.bank)) & (~self.elements.code.isin(self.team.code)) & (~self.elements.team.isin(self.banned_teams))
                if sum(candidates) == 0:
                    continue
                candidates = self.elements.loc[candidates]
                candidates["cost"] *= -1
                candidates = candidates.sort_values(by = ["value_season", "cost"], ascending = False).head(1)
                improvement = int(candidates.value_season - player_out.value_season)
                if transfers[pos]["in"] is None or improvement > transfers[pos]["improvement"]:
                    transfers[pos] = {"out": player_out, "in": candidates, "improvement": int(improvement)}
        self._choose_best_transfer(transfers)
```

Two auxiliary methods are used here, `_get_banned_teams`, this updates the `banned_teams` attribute of the class and represents all the  clubs with 3 players in the team, taking into account the leaving player club. This is the method.

```
def _get_banned_teams(self, player):
        teams = self.team.team.value_counts()
        teams[player.team] -= 1
        self.banned_teams = teams.index[teams == 3]
```

The second method is `_choose_best_transfer`, this selects the best transfers between the best transfers for each position and prints the leaving player, the incoming player and the improvement value.  

```
def _choose_best_transfer(self, transfers):
    if len(transfers) == 0:
        self.transfer = {}
        print("No transfer is suggested")
        return
    improvement = 0
    for transfer in transfers.values():
        if transfer["improvement"] > improvement:
            improvement = transfer["improvement"]
            self.transfer = transfer
    print("-----> Player Out <-----")
    print(self.transfer["out"].first_name, self.transfer["out"].second_name)
    print("-----> Player In <-----")
    print(self.transfer["in"].first_name.values[0], self.transfer["in"].second_name.values[0])
    print("-----> Improvement <-----")
    print(self.transfer["improvement"])
```

Good, the suggestion is made, but, what if I want to make it and save the changes in my team?

To make it possible another method was made called make_transfer.

```
def make_transfer(self, proceed):
    if proceed.lower() != "yes":
        return

    self.bank += (self.transfer["out"].cost + self.transfer["in"].cost).values[0]
    self.team = self.team.loc[self.team.code != self.transfer["out"].code].append(self.transfer["in"])
    with open(self.teamFile, "wb") as file:
        pickle.dump((self.team, self.bank), file)

    print("Transfer made")
    print("Money left in bank: ", self.bank)
    print("Improvement: ", self.transfer["improvement"])
```

This method interacts with the user by the command line, if the user approves the transfer the same is made and the new team and bank value are saved.

Nice, now all I need is a how to use this, a main file, like this:

```
from sys import argv
from fpl_class import fpl_class

if __name__ == "__main__":

    fpl = fpl_class()

    fpl.get_team("data/{}".format(argv[1]))

    fpl.suggest_transfer()

    fpl.make_transfer(input("\nMake transfer?\n"))
```

With this file I can ask for a transfer suggestion for my team, which is passed as a argument and then do it, all with the command:

`python main.py mokaFC`

My team before gameweek 29 is:

Position | Player | value_season
---------|--------|-----------
GK | Martínez | 28.3
GK | Johnstone | 21.3
DEF | Dallas | 23.9
DEF | Stones | 23.0
DEF | Dias | 20.8
DEF | Cresswell | 22.9
DEF | Dunne | 2.1
MID | Salah | 14.0
MID | Reed | 13.6
MID | Gündogan | 23.3
MID | Son | 19.0
MID | Fernandez | 18.2
FOR | Calvert-Lewin | 17.9
FOR | Kane | 16.5
FOR | Brewster | 6


The transfer suggestion was:

* **Sell**: Mohamed Salah
* **Buy**: Tomas Sousek

Which makes an improvement of **9** points in value season.

Salah is not probably the first selling option, and there are clearly better ways to make the transfer suggestion, but I wanted to do some simple functional examples before using more complex transfer suggestion algorithms that take into consideration long term approaches, future matches, players forms, and probabilities of future points.  

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
