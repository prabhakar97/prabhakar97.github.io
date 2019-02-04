---
layout: post
title: "Earthquake alert system in nodejs"
date: 2015-04-27 01:01
comments: true
tags: [nodejs]
---
Afflicted by the recent earthquake in Nepal which took so many lives in India as well, my mind under the shower spawned up an idea of an earthquake alert system that attempts to notify people a few tens of seconds before the quake strikes.
The premise of the idea is that *Electromagnetic waves travel way faster than seismic waves. 300,000 KM/s vs 10 KM/s*. Our communication systems (internet, sms) transmit on electronic waves and even after considering some added overhead (like buffers, processing etc) they can win the race with seismic waves easily. For instance, Kathmandu to New Delhi is around 800 kilometers. A quake epicentred in Kathmandu would be felt in Delhi after around 80 seconds. If we find some way to detect the quake in Kathmandu immediately after it strikes and transmit this information over the public communication infrastructure (web, SMS etc.), we could give the people more than a minute to move to safety. I made following assumptions:

* There must be some research organisations which place earthquake sensors all over the world
* At least one of those organisations, publishes such a feed in real-time
* I could hook into that feed and as soon as some quake gets triggered, I notify the registered people via SMS/Email/Push notifications etc.

Upon some quick googling, I found two such services. First was US Geological Survey.
Their [API page](http://earthquake.usgs.gov/fdsnws/event/1/) says that they make the feed available in various formats, but the feeds are refreshed every 5 minutes.
5 minutes is too long!
Next I found something from EU. [Seismic Portal](http://www.seismicportal.eu/realtime.html) broadcasts the feed on a web socket. Sweet! If I tap onto that feed, I would get near real time notifications.
I can connect this to some SMS gateway and I would be done. After some googling, I found out that buying a plan from SMS gateway providers have become tougher than simply clicking buy now and paying through a credit card.
This was done because of TRAI's strict regulations on bulk SMSes to be sent to Indian mobile numbers, to deter spammers.
I recalled reading about an app called Instapush which provides a REST API to send instant notifications to Android and iOS devices. I went through their documentation and found that wiring them was pretty easy. But I could just get notifications on my phone, not on everyone's. That is doable by writing an Android app which can get notified through [Google Cloud Messaging](https://developer.android.com/google/gcm/index.html). This one later, for now let's go with Instapush.

I created a [NodeJS](https://developers.openshift.com/en/node-js-getting-started.html) application on Openshift. The app opens a websocket, listens to the earthquake feed, processes it, makes an API call to instapush which instantly delivers the notification on my phone. There are just two files in the codebase.

#### Configuring instapush
Instapush needs you to define two things. An event, and trackers for the event. An event is simply an event name. For example: earthquake. Trackers are placeholders in the notification message, which are populated with the values that you pass for them in the REST call to instapush. Here is my configuration for event and trackers.

![InstaPush](/images/quakeinstapush.png)

#### Code

```javascript
var sjsc = require('sockjs-client-ws');
var request = require('request');

var client = sjsc.create("http://www.seismicportal.eu/standing_order");
client.on('connection', function () {
    console.log('Connected');
});

client.on('data', function (msg) {
    var message = JSON.parse(msg);
    var magnitude = message.data.properties.mag;
    var place = message.data.properties.flynn_region;
    var time = new Date(Date.parse(message.data.properties.time));
    console.log('Earthquake of magintude ' + magnitude + " in " + place + " at " + time);
    var options = {
        method: 'POST',
        url: 'https://api.instapush.im/v1/post',
        headers: {
            'x-instapush-appid': "MY_APP_ID",
            'x-instapush-appsecret': "MY_APP_SECRET",
            'Content-Type': "application/json"
        },
        json: true,
        body: { 'event': 'Earthquake', 'trackers': { 'magnitude': magnitude, 'location': place, 'time': time.toLocaleTimeString() } }
    };
    request(options);
});
client.on('error', function (e) {
    console.log('Some error occured');
});
```

The package.json declares the dependency on `sockjs` and `request`.

```json
{
    "name": "quake-prab",
    "version": "1.0.0",

    "engines": {
        "node": ">= 0.6.0",
        "npm": ">= 1.0.0"
    },

    "dependencies": {
        "request": "*",
        "sockjs-client-ws": "*"
    },
    "private": true,
    "main": "client.js"
}
```

#### Possible improvements
This project is quick and dirty. It is just re-wiring of stuff lying on the web. There are bunch of possible improvements here. Some of them are:

* Only notify if the quake has intensity greater than 4.5
* Only notify if the quake has struck in neighbouring countries like Nepal, Bhutan, Pakistan, India
* Provide an endpoint to let user register his/her mobile number on which SMSes can be sent.
