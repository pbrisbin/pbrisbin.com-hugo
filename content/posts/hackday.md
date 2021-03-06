---
title: Hackday
date: 2011-02-28
tags:
---

The idea was that 100 developers, 50 designers, and 50 "others" would 
all get together Friday night, team up and develop a phone or web app to 
make life better in the city of Boston.

Each team would have until 2:30 Sunday afternoon to get something 
demo-able. Then they would present and be judged.

## The Team

Friday night was a bit of a social mixer with drinks, food, and mingling 
up until about 7PM. Then, anyone with an idea was asked to come forward 
and give a 30 second pitch. Those that didn't could then go stand with 
the idea they liked best.

We decided to team up with a Harvard Business School student who wanted 
to help her Mom get less parking tickets. The issue was that the signs 
were so confusing in Boston, that she didn't know she couldn't park 
there.

We picked up a designer and a rails dev and got started.

## The App

We created [CanHazParking][canhaz] which aggregates your location, the 
current time, and the many confusing parking laws in the area to give 
you a simple "can haz" or "no can haz" answer.

The app was written using ruby on rails and jQuery. We used a few [open 
APIs][civic], lots of jQuery, html scraping, and one nifty email hack to 
get the parking information.

I focused mainly on the Snow Emergency logic and helped out a bit with 
Street Cleaning (I determine if even addresses are on the right or left 
side of the vehicle).

Throughout the weekend we had varying levels of functionality with some 
unfortunate breakages just about demo time (pretty sure I contributed 
to that, sorry guys).

Despite these issues, we took the prize for "Best Boston-centric" 
application. For this we get Blue Man Group tickets as well as blurbs in 
the various boston.com articles about the event. The first of such 
articles can be seen [here][article].

Many thanks go to my team members: Beth, Dave, Kara, and Ken. Certainly 
couldn't have had so much fun and been so successful without such a 
smart and easy-going team.

[canhaz]:  http://canhazparking.com "Can Haz Parking?"
[civic]:   http://civicapi.com "The Civic API"
[article]: http://www.boston.com/business/technology/innoeco/2011/02/winners_of_the_first-ever_bost.html
