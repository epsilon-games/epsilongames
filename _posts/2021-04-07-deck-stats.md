---
layout: post
title:  "Deck Stats"
author: "Eloy Chang"
categories: markdown
project_title: "mtg"
excerpt: "How many lands should I put on my deck? This and other basic questions are answered here to help in MTG deck building"
path_image: <img class="header_post_image" src='../assets/img/magic-header-post.png' alt="" />
---

One of the most important, and entertaining parts (at least for me) of MTG is deck building, there is a lot of stuff to take care about and need to plan how many cards of each type and costs add to the deck to secure a good start but do not flaw if the match takes too long.

A normal magic deck must contain at least 60 cards, and most decks do not add extra cards, the recommended number of lands to include in a deck are between 22 and 26 lands, this depends on the average cost of the cards in the deck and the mechanic of the deck itself.

Knowing the probabilities of some events becomes a key factor to deck building, this probabilities may be calculated analytically, this means with probability functions and combinatorial theory, but also, and maybe easier, with simulations.

## About the code

This time a lot of code is involved, so I just will show the code of the mains functions but not the classes definitions, if you are interested in how I do it, you can visit my [Github repository](https://github.com/echang1802/mtg) or contact me about any issue or question.

In the repository are also explained all the steps to use the code and get your own statistics of your decks.

## Lands in first hand

¿How many lands would be drawn in the first hand? This can be answered directly with a  hypergeometric distribution function, which returns the probability of k success in n draws of N events where K of those are success (what?!). Simple english, if we got N objects (call it, 60 cards), where K of those I call them success (lets say, a land card is a success and we have 24 lands between our 60 cards) and we draw n of them (7, by example) then the hypergeometric distribution function return the probability of that, in our draws are k lands (k the number of lands in the first hand we want, 3 for say something).

And, if we do the maths (or use a calculator):

```
from scipy.stats import hypergeom

fh = hypergeom(60, 7, 24)
fh.pmf(list(range(8)))
```

Lands | Prob.
------|------
0 | 2,16%
1 | 12,1%
2 | 26,94%
3 | 30,87%
4 | 19,64%
5 | 6,93%
6 | 1,25%
7 | 0,08%

But another way to do this is by simulations, we may just create a list of 60 zeros, make 24 of they 1 (whose represent the lands) and the randomly choose 7 entries, and count how many lands I drawn, the repeat this process 1000 times and with this I have a estimation of these probabilities, let do this!

```
def estimate_lands_in_fist_hand(deck, simulations = 10000):
    dist = distribution()
    for s in range(simulations):
        deck.draw_hand()
        lands = str(deck.cards_type_in_hand("land")["cards"])
        dist.add_data(lands)
        deck.reset()
    dist.show()
```

In this function I use a `deck` object which I have previously defined, with the method `draw_hand` in which 7 cards are drawn and the method reset which put cards back in deck, I also use a `distribution` object also previously created to handle the events counts, in this case, the number of land drawn in each hand.

The results were:

Lands | Prob.
------|------
0 | 3,98%
1 | 16,81%
2 | 30,52%
3 | 28,64%
4 | 15,22%
5 | 4,19%
6 | 0,61%
7 | 0,003%

As we can see these are not the same numbers as before, but this happens when we are simulating or estimating something, if we increase the number of simulations should trend to the real values.

## Lands in first hand by color

Alright, I can predict how many lands my first hand will have, but, what happens if my deck is not monocolored?

If you look at my last MTG post you will know my competitive deck is a white-black one, and mostly white, so, to me, it is not the same having 1 plain (white land) that is a swamp (black land). In this example my deck has 16 white land cards and 8 black land cards and I want the probability of each combination of lands in my opening hand.

The function to obtain this is very similar but, the event is not the total number of lands, but the lands by each color:

```
def estimate_lands_in_fist_hand_by_color(deck, simulations = 10000):
    dist = distribution()
    for s in range(simulations):
        deck.draw_hand()
        lands = deck.cards_type_in_hand("land")
        c =  "|".join(["{}:{}".format(c,lands["colors"][c]) for c in lands["colors"].keys()])
        dist.add_data(c)
        deck.reset()
    dist.show()
```  

The results were:

  B/W  | W0 | W1 | W2 | W3 | W4 | W5 | W6 | W7
-----|----|----|----|----|----|----|----| -----
B0 | 3,61% | 11,7% | 13.22% | 7,78% | 2,74% | 0,58% | 0,04% | 0%
B1 | 5,68% | 13,99% | 13,51% | 6,13% | 1,47% | 0,16% | 0,01% | 0%
B2 | 2,81% | 6,41% | 5,09% | 0,67% | 0,2% | 0,02% | 0% | 0%
B3 | 0,71% | 1,25% | 0,71% | 0,13% | 0,01% | 0% | 0% | 0%
B4 | 0,1% | 0,12% | 0,1% | 0,01% | 0% | 0% | 0% | 0%
B5 | 0,02% | 0% | 0% | 0% | 0% | 0% | 0% | 0%
B6 | 0% | 0% | 0% | 0% | 0% | 0% | 0% | 0%
B7 | 0% | 0% | 0% | 0% | 0% | 0% | 0% | 0%

This means the most common combination will be 1 land of each color in the first hand, and this happens in almost 1 of each 5 first hands drawn, and the other two frequent events are, draw 2 plains or 2 plains and 1 swamp.

## Combination of cards in first hand

Knowing the probabilities of how many lands will be drawn on the first hand is good, but most decks need to move fast on the firsts turns, this means they need not only to have drawn lands but low cost creatures too.

Let’s estimate this, let's say we call a _perfect first hand_ if it has at least 2 lands (1 plain, 1 swamp) and a 1 mana cost white creature

This is the code to get the estimations:

```
def estimate_combination_of_cards(deck, combination, simulations = 10000):
    counts = 0
    for s in range(simulations):
        deck.draw_hand()
        if deck.get_hand() in combination:
            counts += 1
        deck.reset()
    return counts / simulations
```

To simplify the code I created another class called combination, which is no more than a group of cards, and defined the operator `in` for it as true if the combination is in another group of cards, false if it is not.

In this example the deck has 12 plains, 12 swamp and 8 white creatures which cost 1 mana, and the probability of get at least 2 lands (when they are at least 1 plain and 1 swamp) and 1 white 1 cost creature was: 27,4%

This means I will have a _perfect first hand_ at least 1 each 4 games! Not bad at all!

## Combination of cards in first hand taking mulligans

In Magic, if you dislike your opening hand you may take a mulligan, this is, draw your opening hand again, but, you will have to return 1 card to your deck for each mulligan taken in the game.

So, the question is, if the combination defined as a _perfect first hand_ does not appear in the opening hand, is it worth taking the mulligan? or how many mulligans I must take to get the _perfect first hand_?

In this case the code is a bit more complex because in each simulation I have to check if I need to take the mulligan or not, and after each mulligan if another one must be taken.

```
def estimate_combination_of_cards_with_mulligans(deck, combination, simulations = 10000):
    dist = distribution()
    for s in range(simulations):
        deck.draw_hand()
        combo_appear = deck.get_hand() in combination
        cards_restriction = deck.cards_in_hand() == combination.cards_in_combo()
        ready =  combo_appear or cards_restriction
        while not ready:
            deck.mulligan()
            combo_appear = deck.get_hand() in combination
            cards_restriction = deck.cards_in_hand() == combination.cards_in_combo()
            ready =  combo_appear or cards_restriction
        dist.add_data(deck.mulligans_taken() if combo_appear else -1)
        deck.reset()
    dist.show()
```

The results were:

Mulligans | Prob. | A. Prob.
----------|-------|--------
0 | 28,33% | 28,33%
1 | 20,54% | 48,87%
2 | 14,73% | 63,6%
3 | 10,46% | 74,06%
4 | 7,41% | 81,47%
-1 | 18,53% | 100%

This means that almost 1 of each 2 games the perfect first hand combination will appear in the first drawn or with just 1 mulligan.  

Also, we can know that almost 1 time in 5 games, the _perfect first hand_ will not appear no matter how many mulligans I take.

## How many turns until draw M lands

A good start is not everything, we need to be able to get more mana later in the game, to cast greater spells or play combos.

So the final question is, given that I drawn N lands in the opening hand, how many turns will it take to draw M lands? We will suppose that, every turn the player draws a card, and neither of the players cast spells that affect the deck (make the player draw or mill cards).

This will be made with the next function:

```
def estimate_turns_until_M_lands(deck, N, M, simulations = 10000):
    dist = distribution()
    for s in range(simulations):
        ready = False
        while not ready:
            deck.reset()
            deck.draw_hand()
            lands = int(str(deck.cards_type_in_hand("land")["cards"]))
            ready = lands == N
        ready = False
        turns = 0
        while not ready:
            if deck.draw(return_type = True) == "land":
                lands += 1
            ready = lands == M
            turns += 1
        dist.add_data(turns)
    dist.show()
```

The estimated probabilities were:

Turns | Prob.
------|---------
3 | 5,32%
4 | 9,53%
5 | 12,41%
6 | 13,79%
7 | 13,18%
8 | 12,04%
9 | 8,84%
10 | 7,3%
… | ....
24 | 0,02%

About half the times the fourth land will be drawn between the fifth and the eight turns, this compared with the stats on previous post about games duration (in average a MTG game takes 6 turns) are not good stats, but starting with just one land is already a bad decision.

Curious how there were 2 simulations that tooked 24 turns to draw the fourth land, this means the deck is left with 29cards, which 20 are lands, this is bad luck.

What do you think about these stats? Useful for deck building?

<div class="row align-items-center no-gutters mb-4 mb-lg-5">
      <div class="featured-text text-center text-lg-left">
        <br>
        <p class="text-black-50 mb-0"><a href="{{ '../fpl.html#masthead' | replace: '..', site.url }}">Back to FPL project main page</a></p>
      </div>
</div>


<!-- Core theme CSS (includes Bootstrap)-->
<link href="{{ '../assets/css/fpl_masthead.css' | replace: '..', site.url }}" rel="stylesheet" />
