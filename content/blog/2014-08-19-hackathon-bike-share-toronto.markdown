+++
title = "Hackathon: Bike Share Toronto"
date = "2014-08-19 21:29:58 -0400"
tags = ["opendata", "python", "nodejs", "angularjs", "scikit", "toronto"]
type = "post"
aliases = [
  "/blog/2014/08/19/hackathon-bike-share-toronto/"
]
+++

This past weekend, I had the pleasure of being a participant at the [Bike Share Toronto Hackathon & Design Jam](https://bikesharearup-hackathonde.squarespace.com). This was my first time attending an Open Data related hackathon in Toronto and simply put, the experience was absolutely phenomenal. 36 hours of non-stop hacking on data related to usage of the Bike Share Toronto service combined with the data from the Cycling App made available by the City of Toronto with the singular aim of getting more people on bikes. 

## The Concept
[Team Fixie](https://twitter.com/abh1nv/status/501093674634448896) decided to appeal to the users' common sense by presenting a comparison of various modes of transportation before embarking on a trip. 

{{% screenshot "/images/2014-08-19-Fixie-UI-1.png" %}}

In a nutshell, the responsive web app asks for details about your trip, such as the reason for the trip, start / end locations and optionally a time of arrival. 

{{% screenshot "/images/2014-08-19-Fixie-UI-2.png" %}}

It then presents you with three options that include realistic time estimates (as opposed to average speed based estimates provided by Google Maps) by accounting for factors such as weather and average speeds typically achieved by cyclists during the time of day around the arrival time.

{{% screenshot "/images/2014-08-19-Fixie-UI-3.png" %}}

When assessing the Bike Share option, Fixie includes the time required to walk from your starting point to the nearest Bike Share station, the time required to bike from the first Bike Share station to the Bike Share station nearest to your destination and the time required to walk to your destination from the second Bike Share station.

{{% screenshot "/images/2014-08-19-Fixie-UI-4.png" %}}

Similarly, when calculating the time required for you to ride your own bike, Fixie includes the time required to bike from your starting point to the bike lock post nearest to your destination (so you can secure your bike) and the time required to walk to your destination from the bike post.

{{% screenshot "/images/2014-08-19-Fixie-UI-5.png" %}}

For public transit, driving and walking directions, it relies entirely on Google Maps.

## The Hack
The front end is backed by [NodeJS](http://nodejs.org/) / [ExpressJS](http://expressjs.com/) REST API and the client side is built using [HTML5](http://www.w3.org/TR/html5/), [SASS](http://sass-lang.com/) and [AngularJS](https://angularjs.org/). The auto-complete for address boxes is powered by the [Google Maps Javascript API v3](https://developers.google.com/maps/documentation/javascript/) and location detection is enabled using the [HTML5 Geo-Location API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/Using_geolocation). The entire javascript layer is uses [GulpJS](http://gulpjs.com/) to build and package itself.

The Javascript layer talks to a Python REST API written using [CherryPy](http://www.cherrypy.org/) to calculate the time for each type of trip. The python backend talks to the [Google Directions API](https://developers.google.com/maps/documentation/directions/) to generate waypoints for a trip and also encapsulates the [SciKit Learn](http://scikit-learn.org/stable/) model that predicts travel times between waypoints for a given time of day.

## Datasets Used

The machine learning system that predicts travel time was trained using data from City of Toronto Cycling App data and the Bike Share data. The difficulty here was that the Bike Share data did not provide the change in elevation for the trip and an analysis of the model's features indicated that altitude change was fairly important to ensure an accurate prediction. Using the [Google Elevation API](https://developers.google.com/maps/documentation/elevation/), we were able to approximate the change in altitude for each Bike Share trip and mix that data to enhance the provided dataset and help train the model.

The python service also used the [Bicycle Post and Ring Locations](http://www1.toronto.ca/wps/portal/contentonly?vgnextoid=d46e94ec9fbf3310VgnVCM1000003dd60f89RCRD) dataset provided by the City of Toronto as well as the Bike Share Station dataset to help plan the waypoints for bike trips. 

## Conclusion

At the end some amazing ideas, prototypes and implementations were presented by the 9 participating groups to a packed audience. All in all, the hackathon was a smashing success. I wanted to put this post up to catalogue the concept and implementation that our team came up with, just so I'd have something to refer back to, or in case someone else finds some use for it.

I'd like to thank [Alex Mansourati](https://twitter.com/alexmansourati), [Kent English](https://twitter.com/kentenglish) and [Kaye Mao](https://twitter.com/maozillah) for making Team Fixie awesome. Last but not least, a big thank you to all the organizers and sponsors for making this event happen. Shout outs to [Bianca Wylie](https://twitter.com/biancawylie), [Naomi Freeman](https://twitter.com/Naomi_Freeman), [Allison Buchan-Terrell](https://twitter.com/abuchanterrell), [Michael Markieta](https://twitter.com/MichaelMarkieta), [Matthew Browning](https://twitter.com/CabbagetownMatt) and Anelia.