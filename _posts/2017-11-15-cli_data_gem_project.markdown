---
layout: post
title:      "CLI Data Gem Project"
date:       2017-11-16 00:07:37 +0000
permalink:  cli_data_gem_project
---


Club Nights is an app that provides weekly listings of electronic music events. My data was sourced from the online magazine and community platform Resident Advisor. Resident Advisor provides a list of top regions for electronic music (London, UK; Berlin, DE; New York, US etc.) which I thought was a good starting point for my app. The user can select a region, view events from that region on a particular day of the week specified and then view details of that event if desired. Simple enough. But of course this app wouldn’t be complete without some unexpected obstacles along the way. So here is a selection of some of the obstacles I encountered and what I did to solve them.

After I stubbed out the data with dummy text and established a rough flow for the app my immediate next step was scraping. The first challenge was returning a url that corresponds with the users input. Each Nokogiri url could potentially differ depending on the region, day and event the user selects. There were too many variables to scrape from fully hardcoded urls, so my solution came in the form of class instance variables to fill in the blanks. Even though class instance variables seem to have a reputation as weird or unorthodox, I was able to store the scraped data and then interpolate it within another url for use in different scraper methods and cli methods. Probably not the best solution but here are some of these class instance variables in action:

```
  def self.event_href_title(input)
    doc = Nokogiri::HTML(open("https://www.residentadvisor.net#{@dayname_href}"))

    @event_href = doc.search("#items > li > article > div > h1 > a").map {|link| link.attribute('href').to_s }[input.to_i - 1]
    @event_title = doc.search("#items > li > article > div > h1 > a").map {|event| event.text.to_s }[input.to_i - 1].upcase
  end

  def self.scrape_event_details
    puts "\n\e[4m#{@event_title}\n\e[0m"
    ClubNights::Event.all.clear
    Nokogiri::HTML(open("https://www.residentadvisor.net#{@event_href}")).search("section.contentDetail").each do |details|
      event = ClubNights::Event.new
      event.date = details.xpath('//*[@id="detail"]/ul/li[1]/a','//*[@id="detail"]/ul/li[1]/child::text()').map(&:text).join(" / ")
      event.venue = details.xpath('//*[@id="detail"]/ul/li[2]/a[1]','//*[@id="detail"]/ul/li[2]/child::text()').map(&:text).join(" /")
      event.lineup = details.xpath('//*[@id="event-item"]/div[3]/p[1]').text
      event.description = details.xpath('//*[@id="event-item"]/div[3]/p[2]').text
    end
  end
```


Another problem I encountered was repeating data from a previous listing. Since the user can select another region, event day and corresponding details, the attribute values in the Location and Event classes would need to be #cleared each round through. Again this seems a bit unusual but proved to be the most logic solution for my app. This is how I implemented #clear in one of my Scraper methods:

```
  def self.region_stats
    ClubNights::Location.all.clear

    Nokogiri::HTML(open("https://www.residentadvisor.net/guide/#{@country}/#{@city}")).search("ul.stats-list").each do |stats|
      location = ClubNights::Location.new

      location.venues = stats.search("li:nth-child(1) div.large").text
      location.events = stats.search("li:nth-child(2) div.large").text
      location.population = stats.search("li:nth-child(3) div.large").text
    end
  end
```


When the app is started the user is given a list of 15 regions to choose from but what about prompting a user to select from days of the week, not to mention various abbreviations like Thu, Thur or Thursday? Ruby’s Date::DAYNAMES constant holds the names of the days in an array. So instead of hardcoding each day of the week I could simply ask if DAYNAMES include? input, which accounts for any abbreviated spelling as well. Below is the method that included DAYNAMES constant.

```
  def day
    puts "\n-------------------------------------------------------"
    puts "Enter a day (Mon-Sun) you would like to go to the club:\n\n'region' for a different location\n'exit' to quit"

    input = gets.strip.downcase
    day_names = Date::DAYNAMES.join.downcase

    if day_names.include?(input)
      ClubNights::Scraper.scrape_events(input)
    elsif input == "region"
      restart_list_region
    elsif input == "exit"
      goodbye
    else
      puts "\nPlease enter a valid command."
      day
    end
  end
```


The CLI class required some input prompts (like ‘events’, ‘day’, ‘region’ or ‘exit’) to allow the user to navigate the app. I decided to exclude the use of keyword ‘back’ to help keep the prompts consistent. What wasn’t immediately apparent to me however was jumping into a previous method could set off other CLI methods. My solution was to use methods that ‘restarted’ the chain of activity much like how my initial call method worked. You can view #call and one of the ‘restart’ methods below.

```
  def call
    puts "\n •-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•"
    puts "               Welcome to Club Nights             "
    puts "                       • • •                      "
    puts "  your weekly listings of electronic music events "
    puts " •-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•-•\n\n"
    list_region
    select_region
    day
    event_list
    event_details
    goodbye
  end

  def restart_list_region
    list_region
    select_region
    day
    event_list
    event_details
    goodbye
  end
```


Although these may appear to be minor problems, finding a solution to each of them was necessary to having a fully functional app. Along the way I also improved my scraping abilities, gained a better understanding of github repos and finally got comfortable with using pry, a completely indispensable tool!


