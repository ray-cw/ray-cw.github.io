---
layout: single
title:  "Can You Fill Smokes?"
date:   2022-07-05 11:10:31 -0400
categories: valorant
---

![Infinite Smokes](/assets/img/infinite_smokes.png)

Every Valorant player has heard this phrase directed at them. Maybe you're the type of player who waits until everyone else has locked in their agents, then sigh as you inevitably lock in Brimstone or Omen for the fifth game in a row. Or maybe someone has stolen your Jett or Reyna pick, and now you're forced to play controller.

To some players, having smokes is critical to winning any game of Valorant, and it could be considered a 'throw' to play without them. But does having smokes provide you that significant of an advantage?

This post aims to be the first of series of analyses that will question various things you hear player spout as fact in your ranked lobbies.

## Brief introduction to Valorant and the 'necessity' of smokes

Feel free to skip this section if you're well acquainted with Valorant. For the uninitiated, Valorant is a tactical first person shooter where two teams take turns trying to blow each other or die trying. Additionally, on top of having their guns, agents are equipped with various abilities a la Overwatch, allowing them to stun, blind, teleport, and equip bigger guns (looking at you Chamber).

One team starts as the attackers, the other as defenders, and then they swap roles after 12 rounds. Attackers must get to the bomb site, plant the bomb, and prevent the enemy from defusing if they are to win. (You could also just eliminate all enemies)

Multiple agents have access to 'smokes', which enable them to cut off sight lines. For attackers, this is particularly crucial, as you can effectively eliminate certain spots you can be shot from. By eliminating angles, you decrease the potential number of areas you could be murdered from, giving your site execute a higher chance of succeeding.

## Introducing Our Dataset

When I first decided to answer this question, my immediate instinct was to see what datasets were available. Datasets containing match data from professional games already exist on [Kaggle][kaggle], but I wanted to dive into current ranked trends, much more applicable to the average gamer.

I couldn't find a pre-existing dataset, so I tried getting the data directly from Riot, which requires you to apply for an API key. Short answer, I didn't get one, so I turned to webscraping [tracker.gg][tracker-gg] for player match data.

My web scraper works using Selenium, with code written in Python. The first step is to obtain a sufficient number of match_ids. It does so by going through one player's match history, pulling all the players in their matches, then goes through those players' match history. With a few loops, the bot was able to obtain 20000 match_ids.

Of course, there are going to be quite a few duplicates using this method. Thankfully, the following bash script very quickly eliminates them all:

{% highlight bash %}
#!/bin/bash

sort data/matches.txt | uniq > data/matches_cleaned.txt
{% endhighlight %}

Now with the unique match_id file, we can begin scraping for the match data itself. For each match_id, we are obtaining 10 rows of data, each row representing a different player's performance during that game.

Here are some sample rows:

<table>
  {% for row in site.data.val_dataset.samplematch %}
    {% if forloop.first %}
    <tr>
      {% for pair in row %}
        <th>{{ pair[0] }}</th>
      {% endfor %}
    </tr>
    {% endif %}

    {% tablerow pair in row %}
      {{ pair[1] }}
    {% endtablerow %}
  {% endfor %}
</table>

And here is some additional information on the data:

<table>
 {% for row in site.data.val_dataset.details %}
   {% if forloop.first %}
   <tr>
     {% for pair in row %}
       <th>{{ pair[0] }}</th>
     {% endfor %}
   </tr>
   {% endif %}

   {% tablerow pair in row %}
     {{ pair[1] }}
   {% endtablerow %}
 {% endfor %}
</table>

[tracker-gg]: https://tracker.gg
[kaggle]: https://www.kaggle.com/datasets?search=Valorant
