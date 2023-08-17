---
layout: single
title:  "Tracking Stats for a Valorant Tournament (Part 1)"
date:   2023-08-01 11:10:31 -0400
categories: valorant analysis
---

So in my free time, I'm helping out as a moderator for this Twitch streamer who plays Valorant, a tactical first person shooter. He hosts this tournament series 'ELO Heroes' and one of the ways I contribute is by compiling all of the match stats, creating interesting visualizations to highlight the best teams and players, all to paint an engaging narrative backed up by data.

The process was quite involved and I thought it deserved a comprehensive writeup. I'll be breaking down the writeup into 2 parts: this one focused on the data collection and transformation aspect, and the second about building the visualizations in Tableau.
 
## Project Objective

The end product is to build a **convenient** method for end users to add match information to a 'stats database' (really just a .csv file) over the course of a tournament series. For this, we will need a script that essentially does the following:

```python
def add_match_information(match_id):
	update maps.csv ## 1 row = 1 map/match results
	update performances.csv ## 1 row = 1 player's performance in 1 given match
```

Now I can just make the script, and use it whenever I need to update the database, but ideally I want to make a **tool** that makes it convenient for **any user**, with zero developer knowledge, without needing Python, to update the database on their own. 

Someone else running their own tournament series could use my tool to collect their own stats.You could also use this tool to collect scrim (practice match) data, in order to better analyze a team's strengths and weaknesses, backed by data. 

So what is 'convenient'? Considering the target userbase consists of gamers, I've elected to build a Discord bot that will run the script for users, once given the `match_id`.

Once we have built our tool for data collection, we can start creating our visualizations. The objective here is to build a Tableau dashboard, which connects to our data source, and create interesting visuals for the tournament series. I will want to share this dashboard and make it interactable for the users, so they can explore the dataset themselves.

## Getting the Data

The first question you may ask is "Does Valorant have a publicly available API?" and the short answer is "no". The slightly longer answer is "not officially". Riot (the company that develops Valorant) has a Valorant API, but you have to apply to get a key, and they have been stingy with giving them out.

Luckily for me, there is the [Unofficial Valorant API](https://github.com/Henrik-3/unofficial-valorant-api). One of the endpoints, given a match's ID number, will fetch you a .json containing near everything pertaining to the match. 

I say 'near everything' because the API has no way of knowing the names of the team participating in the tournament. It identifies one team as 'Team Red' and the other as 'Team Blue', which will naturally get confusing if we leave it like that. We will need to make a small modification to our script to accomodate that. 

I would also like to keep track of 'when' the match happened, not the specific date, but rather the week of the series. This will help in creating animated visuals of how certain stats, team or individual performance, change over time. 

We will making use of a Python wrapper for this API, as Python is what we will be working with in large. Here is our modified pseudocode:

```python
def add_match_information(match_id, team_red_name, team_blue_name, week_number):

	# Use API to get match data
	response = fetch_match(match_id)

	## Add in the team names + week number

	update maps.csv ## 1 row = 1 map/match results
	update performances.csv ## 1 row = 1 player's performance in 1 given match
```

## Data Preprocessing

I mentioned that the API returns a JSON containing the match data. Here's a small sample of what that JSON contains:

```json
{
  "status": 200,
  "data": {
    "metadata": {
      "map": "Lotus",
      "game_version": "release-07.03-shipping-11-953184",
      "game_length": 2901,
      "game_start": 1692160211,
      "game_start_patched": "Wednesday, August 16, 2023 4:30 AM",
      "rounds_played": 26,
      "mode": "Competitive",
      "mode_id": "competitive",
      "queue": "Standard",
      "season_id": "0981a882-4e7d-371a-70c4-c3b4f46c504a",
      "platform": "PC",
      "matchid": "ff2b9221-1084-4f82-a436-456eec7fdd16",
      "premier_info": {
        "tournament_id": null,
        "matchup_id": null
      },
      "region": "na",
      "cluster": "US Central (Illinois)"
    },
```

I won't bother copying the rest here because the response for one match alone is **52000 lines**. There is a lot in the above snippet we aren't interested in at all. Before we can start transforming this JSON into workable data, we need to define what the transformed data should look like.

### Table 1: `maps.csv`

Each row/observation pertains to the stats for one given map in Valorant. What is a map? To the API, maps are the same as matches, but it gets confusing because in terms of the tournament, a match is a best of 3, which will consist of multiple maps. Since data specific to 'tournament matches' (map ban, map pick) will not be collected, I will be using the terms 'map' and 'match' interchangeably.

Here is the information I intend to extract from the JSON:

| Field| Description|
|----------|------|
| `match_id`     | Primary key, identifier for each map    |
| `map`          | The name of the map chosen for the game |
| `rounds_played`| # of total rounds played                |
| `team_blue`    | name of team blue                       |
| `team_red`     |  name of team red                       |
| `team_x_rounds`| # of rounds that team x won             |
| `team_x_attack`| # of rounds that team x on attack|
| `team_x_attack_total` | # of rounds that team was the attacker|
| `team_x_defense` | # of rounds that team won on defense|
| `team_x_defense_total` | # of rounds that team was the defender|
| `team_x_5v4s` | # of rounds where that team was up one player|
| `team_x_5v4s_won` | # of rounds where that team was up one player and won|
| `team_x_4v5s` | # of rounds where that team was down one player|
| `team_x_4v5s_won` | # of rounds where that team was down one player and won|
| `winner`| winning team name |
| `week`| what week the map was played in |

### Table 2: `performances.csv`

Each row/observation pertains to the stats for one given individual in a given map in Valorant. Valorant is a 5v5 game, meaning that there should always be 10 performances per `match_id`. 

| Field| Description|
|----------|------|
| `player_id`    | Primary key, identifier for each player    |
| `match_id`     | Primary key, identifier for each map   |
| `name`          | Player's ingame name|
| `agent`| # Player's agent for that map               |
| `acs`    | Player's average combat score for that map   |
| `kills`     |  # of kills                       |
| `deaths`| # of deaths           |
| `assists`| # of assists|
| `kast` | # of rounds that player got a kill, assist, survived or traded |
| `adr` | Player's average damager per round for that map|
| `hs` | headshot percentage|
| `bs` | bodyshot percentage|
| `ls` | legshot percentage|
| `fk` | # of first kills|
| `fd` | # of first deaths|
| `team`| team name |
| `judge_kills`| kills with the judge |
| `operator_kills`| kills with the operator|
| `1_vs_x_scenarios` | number of times the player was put in a 1v2, 1v3, 1v4 or 1v5 scenario|
| `1_vs_x_scenarios_won` | number of times the player was won a 1v2, 1v3, 1v4 or 1v5 scenario|

I won't get too specific on the coding aspect here, if you are curious on how on all of these were extracted you are free to look at the full code [in my repository](https://github.com/ray-cw/valorant-statbot), specifically the file [`datacollection.py`](https://github.com/ray-cw/valorant-statbot/blob/main/helper/datacollection.py). Some of it was easy to collect, some was less so.

## Making the Discord Bot

I now have the script necessary that makes use of the API to fetch data, and transform it into a usable form. The next step is to develop the tool to make this script accessible. I didn't have much experience building a Discord bot, but thankfully this step was actually quite easy.

```python

async def add_match(interaction, match_id: str, team_a: str, team_b: str, week: str):

    await interaction.response.send_message(f"Looking up [{match_id}]...")

    ## Check for duplicates in the database first
    maps_df = pd.read_csv('data/maps.csv',usecols=[0],names=['match_id'],header=None)

    if maps_df['match_id'].str.contains(match_id).any():
        await interaction.followup.send(f"Match exists in database, termininating operation.")
        return

    ## Fetch match
    try:
        match_response = vapi.get_match_details_v2(match_id)
    except: ## Case of error
        await interaction.followup.send(f"Valorant API Error: Match not found, terminating operation.")
        return

    clean_match_df(match_response, team_b, team_a, week)
    clean_performance_df(match_response, team_b, team_a, week)


```

The above snippet of code is the main command of the bot, `add_match` which takes 4 parameters and passes them to the functions `clean_match_df` and `clean_performance_df` from `datacollection.py`, which transforms the data and appends it to `maps.csv` and `performances.csv`.

The command can be used by users with the right permissions (primarily moderators) in the tournament's Discord server. In the case of mistakes/typos, I have also elected to add a `delete_match` function which deletes a match from both tables.

## Current Limitations & Future Improvements

- At the point of running the post, the bot is hosted on my computer, ideally I would like to get it set up remotely on a cloud service (Heroku, AWS, etc.). 
- There are definitely more stats from the API I could get, I simply haven't gotten around to implementing their collection. (i.e `multikills`)
- Similar to stat tracking websites for professional tournament, I want to implement some sort of `player_rating` stat to evaluate players, but figuring out a good formula for that can be tricky.

