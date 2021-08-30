---
title: "Rating Analytics"
date: 2020-01-01T21:34:52-05:00
draft: false
toc: false
images:
tags: 
  - boring man
  - data analysis
  - python
  - statistics
  - rating
  - programming
---
How do you compare rating algorithms? Or more generally, what is an objective measure of the accuracy of a predictor?

# The Motivation #

I run ratings on a [team based shooter called Boring Man](https://spasmangames.com/boringman/), and I've always wanted to be able to rank the players in terms of their skills. In fact I believe my first encounters with hobby programming were because of this game.

In previous iterations of my projects, I had always used the Elo rating scheme to adjust player's ratings, but this time I decided to experiment more and use the [TrueSkill](https://www.wikiwand.com/en/TrueSkill) rating system by the people at Microsoft. Specifically I used this wonderful [library](http://trueskill.org/). To explain the motivations behind this decision, I need to go into how Boring Man is played.

In Boring Man, players are assigned to either one of two teams. After assigning teams, a round starts and players try to eliminate each other. The winner of a round is decided when one team no longer has any alive players. Then the whole process restarts, and the first team to win 10 times is declared the winner of the match. For the sake of this discussion, I will ignore draws.

My previous rating system used Elo to adjust player's ratings when an elimination happened, but using TrueSkill, I can compare the likelihood that a team wins, and adjust the team's constituent's ratings. This was of great interest to me because I always had the suspicion that Elo rating didn't encapsulate what it meant to win a team game.

# Challenge: Turning Elo into a Team Rating #

In order to quantify my suspicions, I had to derive my version of the strength of an Elo team. I envisioned the strength of a team as the sum of the strengths of their individual players. As a unit of strength, I took that to mean: What how strong are they compared to a player that is average rated. From there, the formulation is pretty simple:

Original Expression of A's chance of beating B:
$$P(R_a, R_b) = \frac{1}{1+10^{\frac{R_b-R_a}{400}}}$$

Let B be a 1000 rated player:

$$P(R_a, R_b) = \frac{1}{1+10^{\frac{1000-R_a}{400}}}$$

In order to simplify, let

$$X = 1+10^{\frac{1000-R_a}{400}}$$

Relative strength of Ra over average player

$$\frac{\frac{1}{X}}{1 - \frac{1}{X}} = \frac{1}{X-1} = \frac{1}{10^{\frac{1000 - R_a}{400}}}$$

Then the strength of a team is:

$$S = \sum\limits_{i=1}^n \frac{1}{10^{\frac{1000-R_i}{400}}}$$

where S is the total strength of a team, n is the number of players on the team and Ri is the strength of the ith player on the team.

Then if there is two teams A and B, the likelihood of A winning (and if ignoring draws, 1 - likelihood of B winning) is:

$$W_a = \frac{S_a}{S_a + S_b}$$

# How to judge the strength of predictions #

In my statistics classes, I learned the concept of variance of samples and immediately thought of using that to see which rating system performed better.

In the end I decided to check the internet for some information, and maybe learn some statistics... Well I was glad I did. After some searching I found an interesting wikipedia article about [scoring rule](https://www.wikiwand.com/en/Scoring_rule).

As I understand it, there is functions that given a result and a prediction tell you how *good* the prediction is. My variance method was actually called the Brier Score, a restatement of the variance equation familiar to statistics undergraduates.

$$\frac{1}{N}\sum\limits_{i=1}^n (p - a)^2$$

Where `p` is prediction and `a` is the actual. Another scoring rule or scoring function is the logarithmic, which is actually just a restatement of the equation of entropy.

In my case, this is calculated as follows:

$$a ln(p) + (1 - a) ln(1 - p)$$

Where a is the actual outcome, and p is the prediction of team1 winning. Math is easy under two outcomes...

# The code #


First I copied the [win likelihood function for TrueSkill](https://trueskill.org/#win-probability).

Then I implemented the win likelihood for elo from the equations above.
```
def win_probability_elo(team1, team2):
    s1 = 0
    s2 = 0
    for player in team1:
        s1 += 1.0 / (10 ** ((1000.0 - player.elo) / 400.0))
    for player in team2:
        s2 += 1.0 / (10 ** ((1000.0 - player.elo) / 400.0))
    return s1 / (s1 + s2)
```

I made a logarithmic scoring function to compare the average scores of the two rating systems

```
def score(win_prediction, actual):
    return actual * math.log(win_prediction) + (1 - actual) * math.log(1 - win_prediction)
```

### Collecting data ###

With everything set up, I had my code output the scores of each round into a text file. Time to just sit back and watch.

That will the the subject of my follow up blog post. It takes a while to get a lot of data for this game since not too many people are playing... but hopefully that will change.

This is just a small part of what I intended to do with all this data. Follow the rest of the progress [here](https://github.com/coyote963/bman-data/).