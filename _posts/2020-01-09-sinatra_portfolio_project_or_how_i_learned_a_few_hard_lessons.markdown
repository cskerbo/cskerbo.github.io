---
layout: post
title:      "Sinatra Portfolio Project or: How I Learned A Few Hard Lessons"
date:       2020-01-09 18:07:05 +0000
permalink:  sinatra_portfolio_project_or_how_i_learned_a_few_hard_lessons
---

When I started my Sinatra project, I originally gave myself a 5 day window to get it completed. Fast-forward to now, *one month later*, and I'm finally finished. What a rollercoaster this project turned out to be. Because this project is intended to be used for my portfolio, I wanted it to be above and beyond what the requirements listed. I also wanted it to be something that I can potentially turn into a more long-term project in the future. The original goal seemed simple: take the data from the previous Dead By Daylight web scraping gem I created and utilize it to create a CRUD MVC application that will allow a user to build a custom character loadouts. I searched around online to see if anything similar existed - some sites were close, but not quite what I had in mind. I was happy to have a somewhat original idea.

I started by whiteboarding out the pieces of the application to build - no sweat there. This came easy to me, as I'm familiar with the game and being able to relate game objects to Ruby code was a great way to help cement the Ruby language and MVC structure in my mind. Now it was time to decide on the layout of the site - here is where things got tricky. The layout to display all of the needed information was hard to invision, so I decided to utilize the layout the current game Wiki has. I don't recommend this at all - create your own layout from scratch, and design as you go. Don't attempt to utilize the same structure a similar site has used. What I ended up with was a complex html structure, and so many 'div' sections that it became almost impossible to apply any CSS changes to the site. After many, many hours of trying to wrap my head around why content was overflowing, not lining up, etc - scrapping that site's structure and creating my own is what was needed.

> **Lesson 1**: Create your own layout from scratch, and design as you go. Don't attempt to utilize the same structure a similar site has used.
> 

Capturing the game data I needed to display in the application proved to be quite difficult. The game developer doesn't have any public APIs available, and any open source APIs I found had *very* limited data that wasn't helpful. I ended up utilzing an advanced version of my previous scraper to capture everything I needed - or so I thought. A mistake I made in the beginning was assuming I knew all of the data I needed. However, as the application progressed I often found myslef saying "If I had this piece of data, I could display it here" or "I am going to have to go back and add this data to my scraper". For future reference, I learned that it's best to grab everything available in one swoop - that way you will have the data as the applicaiton progresses. And before you start coding, if you're going to have to scrape data, map out and test the method for obtaining the data you need. Know every field that is available to scrape, and capture everything possible.

> **Lesson 2**: Map out and test the methods for obtaining the data you need. Know every field that is available to scrape, and capture everything possible.
> 

Throughout the project, I struggled with feeling as if I had bitten off more than I could chew. Daily I would worry I was making this project too elaborate, trying to aim to far beyond the requirements. After going back and forth, I finally decided to compromise. There are many features I still want to add to the application - but I pushed those down the priority list to ensure I was able to check off all of the immediate project requirements. At the same time, I did what I could to expand upon functionality and style the application approprirately - in preparation for future enhancements. Be aware of "scope creep". Any time I found myself starting to work on something that wasn't required to complete this project, I would stop myself, make a note of the feature, and divert my focus to what I needed to complete *now*.

> **Lesson 3**: Be aware of "scope creep". Any time I found myself starting to work on something that wasn't required to complete this project, I would stop myself, make a note of the feature, and divert my focus to what I needed to complete *now*.
> 

This was a huge learning experience. I'm glad I decided to go the route I did for this project, because I feel like I ended up with a really great piece for my portfolio. I'm excited to take this application further in the future. Check it out here: [https://github.com/cskerbo/Sinatra-Portfolio-Project](http://)



