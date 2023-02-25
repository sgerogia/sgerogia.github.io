---
layout: post
title: 'Starting up: Your first few days as a leader'
author: stelios
tags: []
categories: []
featured: true
description: "."
image: assets/images/starting-up-leader/yuta-koike-ICgqQsDYndc-unsplash.jpg
---

A new technical endeavour comes with uncertainty. I will share an approach to help you (a founding technical leader) 
lift "the fog of war".

# First few days 

Uncertainty takes many forms 
new team 
new domain 
a startup seeking funding 

Also a "technical leader" takes many forms
Could be the technical co-founder in a startup
a department director in a larger company 
a team lead 

A leader is not necessarily someone with the title  
but it is someone to whom all eyes are turned to for direction

# Your mission, should you decide to accept it...  

![New mission](../assets/images/starting-up-leader/nathan-dumlao-vxHX2qLltdw-unsplash.jpg "New mission")
> Photo by Nathan Dumlao on Unsplash

> Fictional scenario
> The business idea is here only as a McGuffin to help us focus on the thought process

You are the technical co-founder in an order startup for same-day farm-to-plate-to-door meals.  
Or in VC-speak: "Uber for farm fresh-cooked food".  
Your co-founder (CEO) has been working on the business model for the past few months. The market opportunity is just 
amazing, customers cannot wait to log on and start ordering, investors are forming a disorderly queue to invest, this 
is the next unicorn etc etc. 

Here is the outline of the functionality.  
* Urban farmers register with our platform and list their micro-produce on it.
* Customers browse the website and select their desired produce. 
* They also select their preferred recipe to cook the produce in (e.g. salad, soup, etc.) in a central kitchen location
* ...and finally their desired time to deliver the ready meal to their door.
* On the backend the platform allows freelance drivers to sign up, and 
* ...pick up the produce from the farmers to deliver to the central kitchen, and
* ...pick up the cooked orders from the central kitchen and deliver them to the customers.
* The whole system is available as a mobile app and a website.

The founder CEO has added some additional requirements for the platform.  
* You have secured a kitchen partner to cook the meals.  
  The partner not only provides the cooking facilities (shadow kitchen), but they also have their own order management 
  and logistics system (check-in ingredients, check-out cooked orders). Your platform should integrate with this system.
* To save on capex, you should initially be ready to utilise an external Digital Dispatch Platform to manage drivers 
  pickup and delivery (e.g. https://www.truxnow.com/, https://www.qupworld.com/,).
* The system should be in public beta within 3 months of seed funding.   
  There are really-really good business & investor reasons. No point arguing about it, this is a hard requirement.

## ...and some more context

![Putting the numbers together](../assets/images/starting-up-leader/scott-graham-5fNmWej4tAA-unsplash.jpg "Putting the numbers together")
> Photo by Scott Graham on Unsplash

The above might be enough to get started: it is our **What**.  

But we need to get some additional context on the **Why** you, as a technical co-founder, are about to embark on the 
exercise we describe in this post. 

The business is currently in pre-seed stage, bootstrapped by the CEO.  
Having validated the idea and business model with potential customers and partners, the business is now looking to raise 
a seed round to get things off the ground.

But how much seed funding do you need?  
The CEO has a firm grasp of the costs of marketing, sales, and operations; these are already in the financial projection. 
What about product development?   
Will it cost $100k or $1M? Can the team build the MVP in-house? Will the MVP be ready on time for public beta as needed? 

The answer to these questions (and more) can have a dramatic impact on a number of fronts:  
* the "ask" to the prospective investors 
* the product roadmap (is the go-live date really non-negotiable? is the MVP scope non-negotiable?)
* the viability of the business model (What is the company's burn-rate? When will it break even? When will it be 
  profitable?)
* the hiring plan (Do you need a team of 10 or 100? Do you need to go looking for outsourcing agencies for the first few 
  months?)

Now that we have some context, let's get to work!

# Our approach 

As an engineer, most likely your muscle-memory reaction is to jump in, pick the first problem and do http://programming-motherfucker.com/.      
Not to worry! There will be plenty of time for that in the days and months to come!

But first, it is really worth taking a step back and spend a couple of days to flesh out your approach.  

Going from the abstract to the concrete, let's start by defining...

## 1. The problem space

![Problem space](../assets/images/starting-up-leader/charlesdeluvio-OWkXt1ikC5g-unsplash.jpg "Problem space")
> Photo by charlesdeluvio on Unsplash

In our imaginary system we have a number of product- and system-related questions & decisions.  
Just like with any system.    

Here is a (definitely non-exhaustive!) list to consider:   
* Will we be multi-channel? Which distribution channels will we support?
* Is there a target date we aim for?
* What is the MVP scope?
* Do we have external systems to integrate with?
* What are our projected volumes and load?
* Do we handle sensitive data? Are there applicable regulations?

and many more.

Some of these questions have already been answered by the CEO, some are to be decided by you, and some well,... who knows!
The answers give us a high level understanding of all the different "problems" we will need to tackle.  

The best way to represent a fuzzy problem is to break it down into a mind map.  

![Mind map of our imaginary system](../assets/images/starting-up-leader/mind-map.png "Mind map of our imaginary system")
> Mind map of our imaginary system

This exercise can be done collaboratively and allows the team to  
* ...brain-dump all possible aspects and required components of the system,
* ...from a logical (or jobs-to-be-done) PoV
* ...continue drilling down in each tree branch. 

With the problem space laid out, it is time to consider... 

## 2. The system

![System design](../assets/images/starting-up-leader/campaign-creators-IKHvOlZFCOg-unsplash.jpg "System design")
> Photo by Campaign Creators on Unsplash

Questions
* Can we use SaaS like Airtable?
* Can we use a PaaS like Heroku or do we need heavier cloud resources?
* What would the infrastructure footprint look like on Day 1?



![System design](../assets/images/starting-up-leader/Imaginary-System.png "System design")
> System design for our platform


## 3. The plan 

![Building a plan](../assets/images/starting-up-leader/volodymyr-hryshchenko-L0oJ4Dlfyuo-unsplash.jpg "Building a plan")
> Photo by Volodymyr Hryshchenko on Unsplash

![](..%2Fassets%2Fimages%2Fstarting-up-leader%2Fgantt-1.png)


![gantt-2.png](..%2Fassets%2Fimages%2Fstarting-up-leader%2Fgantt-2.png)

## 4. The team & the budget
 
![Teamwork](../assets/images/starting-up-leader/randy-fath-ymf4_9Y9S_A-unsplash.jpg "Teamwork")
> Photo by Randy Fath on Unsplash



# Discussion & Parting thought

![Sailing forward](../assets/images/starting-up-leader/marc-wieland-U9YrT6trizs-unsplash.jpg)
> Photo by Marc Wieland on Unsplash


Until next time, excelsior!

# Footnotes

1. <a name="footnote_1"></a>



  [1]: 