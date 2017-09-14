---
published: true
layout: post
title: 'I Came, I saw, I published...My First Ruby Gem!'
author: "Sergio"
date: '2017-04-13 16:16:01 -0600'
permalink: /2017/04/13/my-first-published-ruby-gem
---

16 hours and then some! I forgot to link WakaTime to my new MacBook. >:)
There were some laughs, there were some tears, and there was plenty of caffeine. But I'm finally finished with my first Ruby Gem ( Version 0.1.0 at least ;] )!

To start off this project I had the idea of building a CLI that was capable of getting game information for A LOT of sports. NFL, MLB, MLS, NASCAR, you name it. But after a few hours of setting up my object structures, I realized that the sports scores had entirely different structures themselves. Baseball had 9 innings, Hockey has 3 periods, NASCAR has....laps? I honestly don't even know. As much as I wanted a challenge to become a better programmer by struggling, due to the time constraints I chose to go all-in on the sport I love watching the most (besides football, but we're a ways out from the 2017 season :( ), the NBA! Also, I figured once I completed the project and got it working, it would be a huge step towards implementing all other sports.

I created a simple CLI Gem called NBA Today that looks up and returns the latest NBA scores and game information all right through your terminal. For a quick video walk-through you can check it out below or click here to see the gem on RubyGems.org or here for the Github Repo

[rubygem youtube VIDEO]

# [](#header-3)Planning

To be quite honest, I was feeling myself a bit too much at the beginning of this project. Coming off hot from a few other Object Oriented Ruby labs, I thought I would be able to breeze through this with little to no planning. BOYE WAS I WRONG...

[erroneous IMAGE]

Turns out there was a lot more involved than just a little Nokogiri scraping and I ran into a plethora of issues. But we'll get to that later...

I started off the planning phase by writing about as much of my idea as I knew. I knew I would be dealing with Games, Teams, Scores, and Players. From the little paragraph I wrote, I was able to set up a basic structure of classes involving those listed objects.

But to start off, I used the bundle gem command. This bootstrapped my project for a quick-start and set up all the files I would need.

```
bundle gem nba_today_gem
```
Next, I set up my bin file to write down "the code I wish I had"

```ruby
#!/usr/bin/env ruby

require "bundler/setup"
require "nba_today_gem"

NbaTodayGem::CommandLine.new.call
```

Inferring from this code, I knew I would need a CommandLine class that would be responsible for interfacing with the terminal. But first, let's walk through the other classes of the program for some context.



# [](#header-3)Game.rb

The center piece of our program. NBA Today revolves around the Game class to get all of the game information. It has the following attributes:

Status
Score
Venue
Top Performers
Home Team
Away Team
URL
Recap
Date

```ruby
class NbaTodayGem::Game
  attr_accessor :status, :score, :venue, :top_peformers, :home_team, :away_team, :url, :recap, :date

  @@all = []

  def initialize
    @top_peformers = {}
    @@all << self
  end

  def away_team=(away_team)
    if !away_team.is_a?(NbaTodayGem::Team)
      raise TypeError, "must be a Team object"
    else
      @away_team = away_team
    end
  end

  def home_team=(home_team)
    if !home_team.is_a?(NbaTodayGem::Team)
      raise TypeError, "must be a Team object"
    else
      @home_team = home_team
    end
  end

  def self.find(index)
    @@all[index]
  end

  def self.all
    @@all
  end

  def totalscore
    self.home_team.score[0].to_i + self.away_team.score[0].to_i
  end

  def self.sort_by_total_score
    # binding.pry
    NbaTodayGem::Game.all.sort! {|game1, game2| game2.totalscore <=> game1.totalscore}
  end

end
```

All of these attributes will be populated via the Scraper class we'll make in a second. When a new Game is initialized, the instance will be pushed into the Game.all array, and its top_performers will be an empty hash we'll populate later with the Scraper class as well. In order to enforce Type on the away and home teams, I coded a conditional that will only set the team if it is a Team object. If you were to try and set the home_team as a string, for example, you would get a custom TypeError.

# [](#header-3)Team.rb

Next, we have our Team class...

```ruby
class NbaTodayGem::Team
  attr_accessor :name, :score

  def initialize(name)
    @name = name
    @score = [0,0,0,0,0]
  end
end
```

It's initialized with a name and a scoreboard of an array. @score[0] represents the final score while the rest of the items represent the score of each quarter of the game.

# [](#header-3)Nba_Player.rb

Next we have our Players class...

```ruby
class NbaTodayGem::NbaPlayer
  attr_accessor :name, :points, :rebounds, :assists

  def initialize(name)
    @name = name
  end
end
```

Each player is instantiated with a name, and has the option to have points, rebounds, and assists that we can add via the Scraper class.

# [](#header-3)Scraper.rb

FINALLY, THE PIECE DE RESISTåNCE...

```ruby
class NbaTodayGem::Scraper
  include CommandLineReporter

  attr_accessor :doc

  def initialize
    # CommandLineReporter progress
    self.formatter = 'progress'

    @doc = Nokogiri::HTML(open("https://www.msn.com/en-us/sports/nba/scores")).css("div.sectionsgroup div.section")[0..3]
    @games = []
  end

  def games
    @games
  end

  def scrape_nba_scores

    report do
      self.doc.css("table tbody").each do |game|
        new_nba_game = NbaTodayGem::Game.new

        new_nba_game.status = game.css("td.matchstatus.orangeText.hidedownlevel.size4.alignleft div").text
        new_nba_game.venue = game.css("td.venue.size4").text
        new_nba_game.url = "https://www.msn.com/#{game.attribute("data-link").value}"

        new_nba_game.away_team = NbaTodayGem::Team.new(game.css("td.teamname.teamlineup.alignright.size234").text)
        new_nba_game.away_team.score[0] = game.css("td.totalscore.teamlineup").text.gsub(/[\n _]/, "") # Total score
        new_nba_game.away_team.score[1] = game.css("tr:nth-child(1) td:nth-child(7)").text.gsub(/[\n _]/, "") # Quarter 1 score
        new_nba_game.away_team.score[2] = game.css("tr:nth-child(1) td:nth-child(8)").text.gsub(/[\n _]/, "") # Quarter 2 score
        new_nba_game.away_team.score[3] = game.css("tr:nth-child(1) td:nth-child(9)").text.gsub(/[\n _]/, "") # Quarter 3 score
        new_nba_game.away_team.score[4] = game.css("tr:nth-child(1) td:nth-child(10)").text.gsub(/[\n _]/, "") # Quarter 4 score

        new_nba_game.home_team = NbaTodayGem::Team.new(game.css("td.teamname.teamlinedown.alignright.size234").text)
        new_nba_game.home_team.score[0] = game.css("td.totalscore.teamlinedown").text.gsub(/[\n _]/, "") # Total score
        new_nba_game.home_team.score[1] = game.css("tr:nth-child(2) td:nth-child(5)").text.gsub(/[\n _]/, "") # Quarter 1 score
        new_nba_game.home_team.score[2] = game.css("tr:nth-child(2) td:nth-child(6)").text.gsub(/[\n _]/, "") # Quarter 2 score
        new_nba_game.home_team.score[3] = game.css("tr:nth-child(2) td:nth-child(7)").text.gsub(/[\n _]/, "") # Quarter 3 score
        new_nba_game.home_team.score[4] = game.css("tr:nth-child(2) td:nth-child(8)").text.gsub(/[\n _]/, "") # Quarter 4 score

        # scrape_nba_game_info(new_nba_game)

        #games << new_nba_game
        # CommandLineReporter progress
        progress
      end
    end
    #games
  end

  def self.scrape_nba_game_info(game)
    get_game_info = Nokogiri::HTML(open(game.url))

    game.date = get_game_info.css("#gamematchupsummary div.basicinfo div.datetime.hide1").text

    if game.date == ""
      game.date = "LIVE NOW"
    end

    game.recap = get_game_info.css("#matchuprecapmodule header h1").text

    point_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.points a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.points a:nth-child(5) div div.board span div.playername span").text)
    ]

    point_leaders[0].points = get_game_info.css("div.leaders div.points a:nth-child(3) div div.board span div.stats span").text
    point_leaders[1].points = get_game_info.css("div.leaders div.points a:nth-child(5) div div.board span div.stats span").text

    rebound_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.rebounds a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.rebounds a:nth-child(5) div div.board span div.playername span").text)
    ]

    rebound_leaders[0].rebounds = get_game_info.css("div.leaders div.rebounds a:nth-child(3) div div.board span div.stats span").text
    rebound_leaders[1].rebounds = get_game_info.css("div.leaders div.rebounds a:nth-child(5) div div.board span div.stats span").text

    assist_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.assists a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.assists a:nth-child(5) div div.board span div.playername span").text)
    ]

    assist_leaders[0].assists = get_game_info.css("div.leaders div.assists a:nth-child(3) div div.board span div.stats span").text
    assist_leaders[1].assists = get_game_info.css("div.leaders div.assists a:nth-child(5) div div.board span div.stats span").text

    game.top_peformers[:points] = point_leaders
    game.top_peformers[:rebounds] = rebound_leaders
    game.top_peformers[:assists] = assist_leaders
  end
end
```

Our Scraper is initialized with a @doc instance variable that contains a Nokogiri object of the MSN NBA Scores page. It's also initialized with a @games variable that will contain instances of Games as they are scraped from the page. It's also initialized with a progress formatter from our CommandLineReporter Gem, more on that later.

[IMAGE]

The Scraper has two main methods, #scrape_nba_scores and #scrape_nba_game_info.

```ruby
  def scrape_nba_scores

    report do
      self.doc.css("table tbody").each do |game|
        new_nba_game = NbaTodayGem::Game.new

        new_nba_game.status = game.css("td.matchstatus.orangeText.hidedownlevel.size4.alignleft div").text
        new_nba_game.venue = game.css("td.venue.size4").text
        new_nba_game.url = "https://www.msn.com/#{game.attribute("data-link").value}"

        new_nba_game.away_team = NbaTodayGem::Team.new(game.css("td.teamname.teamlineup.alignright.size234").text)
        new_nba_game.away_team.score[0] = game.css("td.totalscore.teamlineup").text.gsub(/[\n _]/, "") # Total score
        new_nba_game.away_team.score[1] = game.css("tr:nth-child(1) td:nth-child(7)").text.gsub(/[\n _]/, "") # Quarter 1 score
        new_nba_game.away_team.score[2] = game.css("tr:nth-child(1) td:nth-child(8)").text.gsub(/[\n _]/, "") # Quarter 2 score
        new_nba_game.away_team.score[3] = game.css("tr:nth-child(1) td:nth-child(9)").text.gsub(/[\n _]/, "") # Quarter 3 score
        new_nba_game.away_team.score[4] = game.css("tr:nth-child(1) td:nth-child(10)").text.gsub(/[\n _]/, "") # Quarter 4 score

        new_nba_game.home_team = NbaTodayGem::Team.new(game.css("td.teamname.teamlinedown.alignright.size234").text)
        new_nba_game.home_team.score[0] = game.css("td.totalscore.teamlinedown").text.gsub(/[\n _]/, "") # Total score
        new_nba_game.home_team.score[1] = game.css("tr:nth-child(2) td:nth-child(5)").text.gsub(/[\n _]/, "") # Quarter 1 score
        new_nba_game.home_team.score[2] = game.css("tr:nth-child(2) td:nth-child(6)").text.gsub(/[\n _]/, "") # Quarter 2 score
        new_nba_game.home_team.score[3] = game.css("tr:nth-child(2) td:nth-child(7)").text.gsub(/[\n _]/, "") # Quarter 3 score
        new_nba_game.home_team.score[4] = game.css("tr:nth-child(2) td:nth-child(8)").text.gsub(/[\n _]/, "") # Quarter 4 score

        # scrape_nba_game_info(new_nba_game)

        #games << new_nba_game
        # CommandLineReporter progress
        progress
      end
    end
    #games
  end
  ```

# [](#header-3)#scrape_nba_scores
Our #scrape_nba_scores method create a new instance of a Game, then pipes in all of the data we scrape from the Scores page. At the end of that method we call on the #scrape_nba_game_info and pass in our new game to get additional information for that game.

```ruby
  def self.scrape_nba_game_info(game)
    get_game_info = Nokogiri::HTML(open(game.url))

    game.date = get_game_info.css("#gamematchupsummary div.basicinfo div.datetime.hide1").text

    if game.date == ""
      game.date = "LIVE NOW"
    end

    game.recap = get_game_info.css("#matchuprecapmodule header h1").text

    point_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.points a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.points a:nth-child(5) div div.board span div.playername span").text)
    ]

    point_leaders[0].points = get_game_info.css("div.leaders div.points a:nth-child(3) div div.board span div.stats span").text
    point_leaders[1].points = get_game_info.css("div.leaders div.points a:nth-child(5) div div.board span div.stats span").text

    rebound_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.rebounds a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.rebounds a:nth-child(5) div div.board span div.playername span").text)
    ]

    rebound_leaders[0].rebounds = get_game_info.css("div.leaders div.rebounds a:nth-child(3) div div.board span div.stats span").text
    rebound_leaders[1].rebounds = get_game_info.css("div.leaders div.rebounds a:nth-child(5) div div.board span div.stats span").text

    assist_leaders = [
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.assists a:nth-child(3) div div.board span div.playername span").text),
      NbaTodayGem::NbaPlayer.new(get_game_info.css("div.leaders div.assists a:nth-child(5) div div.board span div.playername span").text)
    ]

    assist_leaders[0].assists = get_game_info.css("div.leaders div.assists a:nth-child(3) div div.board span div.stats span").text
    assist_leaders[1].assists = get_game_info.css("div.leaders div.assists a:nth-child(5) div div.board span div.stats span").text

    game.top_peformers[:points] = point_leaders
    game.top_peformers[:rebounds] = rebound_leaders
    game.top_peformers[:assists] = assist_leaders
  end
```

# [](#header-3)#scrape_nba_game_info

Here, we use the url instance variable of the game object to open the MSN Sports page of the selected game and scrape more additional information like top performers, the date, and a little Recap headline. Next, we can wrap up all of our data and execute it through our CommandLine class.

# [](#header-3)CommandLine.rb

For this class I wrote a few methods I knew I would be calling on throughout the CLI:

#call
#menu
#list_games
#pick_game_menu
#show_game
#goodbye

# [](#header-4)#call

```ruby
  def call
    puts "Hello, Welcome to the NBA Today Gem!"
    menu
    goodbye
  end
```

The #call method simply welcomes the user and displays the menu. Once that menu is exited, we display our #goodbye

# [](#header-4)#menu

```ruby
  def menu
    input = nil

    until input == "exit"
      puts
      puts "Type in 'nba today' for the latest NBA scores"
      puts "or type 'exit' to quit!"
      puts
      print "> "
      input = gets.strip.downcase

      if input == "nba today"
        list_games
        pick_game_menu
      end
    end
  end
```

Seeing that our program is fairly simple, and I didn't want to have a heavy initial load time, I wanted to have a little "main menu" where you could start the scraping process.

# [](#header-4)#list_games

```ruby
  def list_games
    if NbaTodayGem::Game.all.size == 0
      NbaTodayGem::Scraper.new.scrape_nba_scores

      header :title => 'NBA SCORES', :width => 75, :align => 'center', :rule => true, :color => 'green', :bold => true, :timestamp => true

      NbaTodayGem::Game.all.each.with_index(1) do |game, index|
        table(:border => true) do
          row do
            column('GAME NUM.', :width => 10, :align => 'center')
            column(game.date, :width => 30, :align => 'center')
            column('SCORE', :width => 10, :align => 'center')
          end
          row do
            column(index)
            column(game.away_team.name)
            column(game.away_team.score[0])
          end
          row do
            column('')
            column(game.home_team.name)
            column(game.home_team.score[0])
          end
        end
      end
    else
      header :title => 'NBA SCORES', :width => 50, :align => 'center', :rule => true, :color => 'green', :bold => true, :timestamp => true

      NbaTodayGem::Game.all.each.with_index(1) do |game, index|
        table(:border => true) do
          row do
            column('GAME NUM.', :width => 5)
            column(game.date, :width => 30, :align => 'center')
            column('SCORE', :width => 10, :align => 'right')
          end
          row do
            column(index)
            column(game.away_team.name)
            column(game.away_team.score[0])
          end
          row do
            column('')
            column(game.home_team.name)
            column(game.home_team.score[0])
          end
        end
      end
    end
  end
```

I was proud of the #list_games method I wrote because of that first line. Before I added the if !self.games line of code, every time I typed in 'list games' in my program it would have to reload and rescrape the web pages for data, resulting in a terrible user experience. This way, the if statement checks to see if self.games is populated first, before scraping the page. If we already have data, it simply displays it. No need for scraping!

# [](#header-4)#pick_game_menu

```ruby
  def pick_game_menu
    game_no = nil

    until game_no == 'back' || game_no == 'exit'
      puts
      puts "Type in a GAME NUM. to view more stats and info"
      puts "Type 'list games' for a list of games"
      puts "Type 'back' to go back to the main menu."
      puts
      print "> "
      game_no = gets.strip.downcase

      if game_no == "list games"
        list_games
        pick_game_menu
      elsif game_no.to_i > 0 && game_no.to_i <= NbaTodayGem::Game.all.length
        index = game_no.to_i - 1
        game = NbaTodayGem::Game.find(index)
        NbaTodayGem::Scraper.scrape_nba_game_info(game)
        show_game(index)
      end
    end
  end
```

This method displays a little menu for the user to either 'list games' or type in the Game Number of the game they wish to see more information on. The input is ran through a conditional to make sure they are selecting a valid number (and not a random one) or wanting to see the list of games again.

# [](#header-4)#show_game

```ruby
  def show_game(index)
    table(:border => true) do
      row do
        column(NbaTodayGem::Game.all[index].date, :width => 30, :align => 'center')
        column(NbaTodayGem::Game.all[index].venue, :width => 55, :align => 'center')
      end
    end

    puts

    if NbaTodayGem::Game.all[index].recap != ""
      table(:border => true) do
        row do
          column(NbaTodayGem::Game.all[index].recap, :width => 80, :align => 'center')
        end
      end

      puts
    end

    header :title => 'BOX SCORE', :width => 100, :align => 'center', :rule => true, :color => 'red', :bold => true

    table(:border => true) do
      row do
        column('TEAM', :width => 25, :align => 'center')
        column('QTR 1', :width => 9, :align => 'center')
        column('QTR 2', :width => 9, :align => 'center')
        column('QTR 3', :width => 9, :align => 'center')
        column('QTR 4', :width => 9, :align => 'center')
        column(NbaTodayGem::Game.all[index].status, :align => 'center')
      end
      row do
        column(NbaTodayGem::Game.all[index].away_team.name)
        column(NbaTodayGem::Game.all[index].away_team.score[1])
        column(NbaTodayGem::Game.all[index].away_team.score[2])
        column(NbaTodayGem::Game.all[index].away_team.score[3])
        column(NbaTodayGem::Game.all[index].away_team.score[4])
        column(NbaTodayGem::Game.all[index].away_team.score[0])
      end
      row do
        column(NbaTodayGem::Game.all[index].home_team.name)
        column(NbaTodayGem::Game.all[index].home_team.score[1])
        column(NbaTodayGem::Game.all[index].home_team.score[2])
        column(NbaTodayGem::Game.all[index].home_team.score[3])
        column(NbaTodayGem::Game.all[index].home_team.score[4])
        column(NbaTodayGem::Game.all[index].home_team.score[0])
      end
    end

    puts
    header :title => 'TOP PERFORMERS', :width => 100, :align => 'center', :rule => true, :color => 'green', :bold => true

    table(:border => true) do
      row do
        column("#{NbaTodayGem::Game.all[index].away_team.name}", :width => 35, :align => 'center')
        column('', :align => 'center')
        column("#{NbaTodayGem::Game.all[index].home_team.name}", :width => 35, :align => 'center')
      end
      row do
        column("#{NbaTodayGem::Game.all[index].top_peformers[:points][0].name}: #{NbaTodayGem::Game.all[index].top_peformers[:points][0].points}", :width => 35, :align => 'center')
        column('POINTS', :align => 'center')
        column("#{NbaTodayGem::Game.all[index].top_peformers[:points][1].name}: #{NbaTodayGem::Game.all[index].top_peformers[:points][1].points}", :width => 35, :align => 'center')
      end
      row do
        column("#{NbaTodayGem::Game.all[index].top_peformers[:rebounds][0].name}: #{NbaTodayGem::Game.all[index].top_peformers[:rebounds][0].rebounds}", :width => 35, :align => 'center')
        column('REBOUNDS', :align => 'center')
        column("#{NbaTodayGem::Game.all[index].top_peformers[:rebounds][1].name}: #{NbaTodayGem::Game.all[index].top_peformers[:rebounds][1].rebounds}", :width => 35, :align => 'center')
      end
      row do
        column("#{NbaTodayGem::Game.all[index].top_peformers[:assists][0].name}: #{NbaTodayGem::Game.all[index].top_peformers[:assists][0].assists}", :width => 35, :align => 'center')
        column('ASSISTS', :align => 'center')
        column("#{NbaTodayGem::Game.all[index].top_peformers[:assists][1].name}: #{NbaTodayGem::Game.all[index].top_peformers[:assists][1].assists}", :width => 35, :align => 'center')
      end
    end
  end
```

The #show_game method is responsible for using our CommandLineReporter Gem to display data in neatly organized tables and headers. For more info on how to use the Gem you can look through the wiki on their GitHub page. It was a pretty simple process of laying out columns and rows and plugging in the scraped data.

When this is all wrapped up and executed through the bin file, you can get the latest NBA Scores through your terminal! I struggled a bit with this project but ending up with a fully functioning CLI made it so worth it in the end. I learned so much and I can't wait to learn more in my future projects.

Thanks for taking the time to check out my first Ruby Gem, I hope there was some information to help you get started on yours!
