---
layout: single
title:  "Capstone: MTG Classifier (Part 1: EDA)"
date:   2022-08-05 11:10:31 -0400
categories: brainstation
---

![Project](/assets/img/project-cover.jpg)

## Prelude

For the last 3 months I've been taking a data science certification course with Brainstation. In a short span of time I've learn to work with numerous technologies in the realm of data analytics and machine learning. This capstone project is the summation of everything I've learned over the last three months, and I am happy to showcase it here.

You can find all the code related to this project [here](https://github.com/ray-cw/MTG-Color-Classifier).

##  Introduction

On August 5th, 1993, Wizards of the Coast released the first set of Magic: The Gathering, Limited Edition Alpha. This then unknown card game began with 295 cards. Today, it is one of the most popular trading card games, right up there with Yugioh and Pokemon TCG, with a card base exceeding 24,000 unique entries.

One of the integral components of MTG’s game design revolves around 5 colours: white, blue, black, red, and green, each representing a different realm of magic. Most cards have one colour, and the card is designed with that colour’s ‘identity’ in mind. Certain themes and mechanics that are strongly associate with each colour, encompassing this so called ‘colour identity’. Cards can also have multiple colours, and have themes and mechanics belonging to each colour.

The goal of my project is to build a classification model that can learn what ‘colour identity’ is and group cards by colour. I will employ natural language processing in order to tokenize the descriptions of each card, and post modelling, investigate to see which coefficients are strong indicators for or against a certain class.

In terms of value added, my initial goal is to prototype a tool that can assist in the creation of new Magic cards, by informing the designer which colour the card should belong to. This is to ensure new cards adhere to the design principles of previous cards.

## Obtaining the Dataset

Our data was pulled from MTGJSON.com, an open source project that catalogs all MTG data in a portable format. Using the requests library, I pulled the data for all 24733 unique Magic cards. The data came semi-structured, with each card having an inconsistent number of columns. I only extracted the features of interest which were:

<table>
 {% for row in site.data.mtg_dataset.details %}
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

Not all Magic cards were usable. We had to filter out some data:

* Remove all cards that were not legal in any format. This includes cards from ‘joke sets’ such as Unhinged that have cards that do not adhere to traditional colour design principles.
* Remove cards from the set ‘Conspiracy’ which includes cards with the type ‘Conspiracy’, a unique mechanic that falls out of the realms of normal Magic design.

After this initial preprocessing, we still have 23630 unique Magic cards.

## Data Preprocessing & EDA

I performed a few additional steps of data cleaning before modelling and exploring our data:

**Convert our dependant, colours, into 6 classes representing all 5 colours + colourless cards.** I have elected to drop all the other combinations, as this would result in too many classes, and create a large class imbalance. An early version of this project grouped all multicoloured cards into one category, but this proved to be too confusing for our model, as a blue-red multicoloured card will have little in common with a green-white card.

**Set the power and toughness of non-creatures cards to -1.** This is to avoid passing NaN values into the model. -1 is a good value as power and toughness of creature cards will never be negative.

**Replace any instance where power/toughness is a * with the card’s manaValue.** Cards with asterisks are few and far between, but typically have some special mechanic where asterisk represents some dynamic number (cards in hand, graveyard, etc.). In these situations, in order to preserve as much data as possible, we have elected to keep these cards but substitute asterisk with manaValue. Both power/toughness and manaValue are indicators of power level, and it makes sense to substitute one with the other.

**Merge name and text.** Name and text are both going to be subjected to natural language processing, and it simplifies the process of vectorizing them both together.

**Dummify our types column.** There are 8 primary types in MTG, with 9 additional combinations (Artifact Creature, Enchantment Creature, etc.) that have existed in previous cards.

**Encode our subtypes column using TransactionEncoder.** We cannot easily dummy our Subtypes column, as there are 1700 unique combinations of various subtypes. We instead use TransactionEncoder to detect the presence of a subtype. We now have about 300 Subtype fields instead.

## Natural Language Processing (NLP)

Our last step before modelling was vectorizing our merged name and text data. I used CountVectorizer to examine the most frequent tokens. I also created a list of explicit stop words for the vectorizer to ignore. These stop words are associated with my own domain knowledge of Magic, words that appear frequently across all colour and should not be a factor in determining the colour identity of a card.

The frequency chart aligns with my own domain knowledge of magic cards: red cards are strongly associated with dealing direct damage.
I made sure to perform the train-test split prior to vectorization in order to prevent data leakage. CountVectorizer was only fitted on the training set; any tokens appearing in the test set would be ignored.

I have elected to not scale my data as some early models with scaling have resulted in poorer performance.

## Final Notes

We now have cleaned and prepared our dataset in preparation for modelling. In my next blog post, I will go into the modelling process, hyperparamater optimization, model evaluationand analyzing our findings.


