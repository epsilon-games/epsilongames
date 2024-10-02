---
layout: post
title:  "Season Dashboard"
author: "Eloy Chang"
categories: markdown
project_title: "fpl"
excerpt: "A brief look into the season performance of all players and teams"
path_image: <img class="header_post_image" src='../assets/img/fpl-header-post.png' alt="" />
---

This time I just wanted to know about the performance not just in the last gameweek, but all the season so far.

For this I have to use two other endpoints of the FPL API, the fixtures endpoint and the element endpoint.

## Fixtures endpoint

This endpoint has information about every scheduled fixture in the actual season of the premier league, like gameweek, home team, away team, date and time, and a level of difficulty estimated to the both teams involved.

In this case I just wanted to know the gameweek of each fixture, to download the fixture a new method was added to the `tableau_data` class

```
def _get_fixtures_data(self):
        url = "https://fantasy.premierleague.com/api/fixtures/"
        r = requests.get(url.format(id))
        json = r.json()

        self.fixtures = pd.DataFrame(json)[["id", "event"]].set_index("id")
```

## Elements endpoint

This endpoint has all the relevant information of each player in the premier league, a lot of useful information for future analysis like, goals, assists, clean sheets, how many trainers bought the player and how many sold it, etc. All open by fixtures.

This time I will focus just on points scored on each gameweek, to download this data I create the `get_historical_data` method.

```
def get_historical_data(self):
        self._get_fixtures_data()
        self.history = pd.DataFrame()
        columns = ["fixture", "total_points"]
        url = " https://fantasy.premierleague.com/api/element-summary/{}/"
        for id, player in self.elements.iterrows():
            r = requests.get(url.format(id))
            json = r.json()

            fixtures =  pd.DataFrame(json['history'])[columns].set_index("fixture")
            fixtures = fixtures.join(self.fixtures)
            fixtures["player_id"] = id
            self.history = self.history.append(fixtures)
```

And the resulting dashboard was this...

<body>
<div class='tableauPlaceholder' id='viz1617667000902' style='position: relative'><noscript><a href='https:&#47;&#47;echang1802.github.io&#47;epsilon.github.io&#47;'><img alt='Points history ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book2_16175767404040&#47;Pointshistory&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='Book2_16175767404040&#47;Pointshistory' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Bo&#47;Book2_16175767404040&#47;Pointshistory&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='es' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1617667000902');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1024px';vizElement.style.height='795px';} else { vizElement.style.width='100%';vizElement.style.height='1227px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>
</body>

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
