---
layout: post
title:      "Dead By CLI"
date:       2019-10-24 22:00:32 +0000
permalink:  dead_by_cli
---


After a grueling 9 caffeine-filled days - I've just completed my CLI Data Gem project! I was nervous about writing my first program from scratch, so I decided to base my gem off of something that is interesting and personal to me. That was the best decision I could have made and if you are nearing your own CLI project I highly recommend you do the same. Having a thorough understanding of the data you are working with, and being interested in the subject, is *so helpful* when learning.


**So what does this gem do and why?**

<center>![](https://steamcdn-a.akamaihd.net/steam/apps/381210/header.jpg?t=1570547542)</center>


My gem is a CLI for the video game "[Dead by Daylight](http://deadbydaylight.com/en)". If you haven't ever heard of it, check it out at that link! Being a horror movie fan, this game is right up my alley and is one of the most unique games I have ever played. 

I've enjoyed watching the game take shape over the last few years, and I stay very close to news from the developer. It's been so interesting to see the developer grow in talent and size as the game continues to gain popularity. I've wanted to be more involved in the game's community for a while now, and I have handfuls of ideas on programs that could be created utilizing their APIs, so this was the perfect time to start.

My gem scrapes the Official Dead By Daylight Wiki and allows the user to view character and perk information for all available Survivors and Killers in the game. The interface will guide you through selection of a Survivor or Killer, allow you to view their biographies, show you available perks with descriptions, and allow you to create a full character loadout to review. I see this gem as a starting point for a tool I plan to develop later on down the road that will allow players to interact with their current in-game characters at a greater level, especially through social media paths like Reddit.

**While making your first gem, how many times did you put your face in your hands and curse the world?**

Great question! I lost count when I hit triple digits. This was definitely the most challenging thing I have done so far at Flat Iron. Jumping away from having rspec to guide you through errors is quite literally terrifying.

There were many challenges parts, but I would say the most difficult piece was building the web scraper. It took me 3-4 days of trial and error, a couple of sleepless nights, and the entire stock of cold brew at Strabucks to build. I naively thought pulling from a game wiki would be easy. I assumed pages would all be uniform and set up fairly similar, because why wouldn't it be? No, that would be too easy wouldn't it.

What I instead found was each page, even though they all housed similar data, was vastly different. Scraping was a bit complex to dive into to begin with, but when you are dealing with tags levels and levels deep it gets so difficult to navigate. On top of that, telling the scraper only certain `<p>` tags within a huge set each need to be their own objects was brain-killing. Here is an example:

```
def self.scrape_perks
    page = Nokogiri::HTML(open("https://deadbydaylight.gamepedia.com/Perks"))
    all_perks = []

    perk_extract = page.css('div.mw-parser-output')
    counter = 1
    perk_extract.each do |item|
      perk_name_extract = item.css('table.wikitable.sortable tr th[2] a[1]')
      perk_name = perk_name_extract.map {|name| name.attribute('title').text.gsub("\n", "")}
      perk_description_extract = item.css('table.wikitable.sortable tbody tr td')
      perk_description = perk_description_extract.map {|description| description.text.gsub("\n", "")}
      perk_hash = Hash[perk_name.zip(perk_description.map {|i| i.include?(',') ? (i.split /, /) : i})]
      perk_list = perk_hash.each do |name, description|
        perk_complete = {:name => name, :description => description, :count => counter}
        all_perks << perk_complete
        counter += 1
      end
    end
    all_perks
 end

```

All that being said, I made it through and I learned a **TON** about web scraping. 

**What's next?**

After this blog post, I'll be creating a video walkthrough of my CLI and then I should be ready to submit the project. From there, it's technical review time.

<center>![](http://giphygifs.s3.amazonaws.com/media/LRVnPYqM8DLag/giphy.gif)</center>

I'm a little nervous, so hopefully it goes well!

After that, I am going to go outside (if I can remember where that is) and enjoy the five minutes of sunlight we might be graced with here in Seattle.




