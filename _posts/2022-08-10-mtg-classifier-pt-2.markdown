---
layout: single
title:  "Capstone: MTG Classifier (Part 2: Modelling)"
date:   2022-08-10 11:10:31 -0400
categories: brainstation
---

![Project](/assets/img/project-cover.jpg)

[In my last post](../mtg-classifier) I explained how I went about cleaning and preparing the dataset for modelling. In this post I will be going into detail on my thought process, model evaluation, hyperparamater optimization and key takeaways from the results.

##  Model Evaluation

We tried out a variety of different models: logistic regression, KNN, decision trees, random forest and XGBoost. I used normalized confusion matrices to compare the model performance.

<img src='https://www.dropbox.com/s/g15a2v3729xbc9d/lr.png?raw=1' width='49%' />
<img src='https://www.dropbox.com/s/44imt9pmwp8huto/dt.png?raw=1' width='49%'/>
<img src="https://www.dropbox.com/s/yvtivqvfrzok99c/knn.png?raw=1" width='49%'/>
<img src='https://www.dropbox.com/s/81jc28544hcq3oa/rf.png?raw=1' width='49%'/>
<img src='https://www.dropbox.com/s/ze39zu0xp5hzvfo/xgb.png?raw=1' width='49%'/>

Our best performing model appears to be logistic regression. We used GridSearch to find the optimal hyperparamaters and found that LR with C=1 was the best performing. (Notably, KNN performs rather poorly with our dataset.)

Accuracy is a bit of a misleading metric. In the world of Magic design, there are quite a few cards that are designed 'incorrectly', commonly referred to as color design breaks. Take a look at these three cards:

<img src='https://c1.scryfall.com/file/scryfall-cards/large/front/1/f/1f44e0cc-68bd-49c6-97d2-be9b353d1579.jpg?1580013766' width='32%' />
<img src='https://c1.scryfall.com/file/scryfall-cards/large/front/9/7/974e84ce-5b51-4bd7-9a4d-b64d8f8f62d4.jpg?1572489802' width='32%' />
<img src='https://c1.scryfall.com/file/scryfall-cards/large/front/a/5/a5e14b62-c050-4d43-aeee-873f46d1e295.jpg?1562654028' width='32%'/>

The former two cards are both classified as white by our logistic regression model, but the third is also predicted to be white, when it was actually a blue card. This doesn't mean the model is necessarily wrong; its simply identifying what the color of the card should be. This is what I mean by accuracy being a misleading metric: the model can be right even when it's wrong.

This does make model evaluation confusing, and may require a more qualitative approach. Instead of looking at accuracy, it makes more sense to look at the coefficients - how the model weighs the different features when deciding what color a card should be. Take a look at the coefficients from red:

<img src='https://www.dropbox.com/s/1742dmb5kj575i4/top_Red.png?raw=1'>

We can compare what our model sees [to this article about the mechanical color pie](https://magic.wizards.com/en/articles/archive/making-magic/mechanical-color-pie-2021-10-18): haste, drawing and discard, and menace are all mechanics heavily connected to red. 

Another key observation is that **the most common misclassification for each colour is one of its ally colours**. Every colour in MTG has 2 ally colours, and colours will often share mechanics with their ally colour. These misclassifications are a good indicator that the model is picking up on this. 

For example:
- Green cards are most often mistaken for White cards. (Common mechanic: gaining life)
- Black cards are most often mistaken for Red cards. (Common mechanic: menace)
- White cards are most often mistaken for Blue cards. (Common mechanic: flying)

## Results

I now have a classification model that performs at roughly 80% accuracy that can classify a card provided it’s name, text, and other aforementioned features. From this model, we can obtain a list of coefficients and how they positive or negatively impact the probability of a card being a certain colour. In terms of potential business value added:

- Card designers can run future cards through the model and check whether or not it was designed ‘appropriately’ (If the card is red but the model thinks its white, something might be wrong there.)
- Card designers may also want to go in the opposite direction and make an innovative card that model will almost certain misclassify. They can look at the most negative coefficients from each class (maybe goblins have never been blue before) and explore designing unique cards in that design space.
- Instead of predicting the colour identity, this project could be inverted using more advanced model to see if it is possible to train an AI to make custom magic cards. (In this case, name + text are the dependant variables).
On a broader scale, this project aims to showcase how natural language processing can be used to ensure consistency between a new product of one category and older products of that same category. 

A massive shoutout to Brian Yee's own color classifier project, which can you find documented in [this repository](https://github.com/Yee-Brian21/MTG-Color-Classifier).