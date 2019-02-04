---
layout: post
title: "Ship a REST API with Node, ES6 and MongoDB - Part 1"
date: 2017-03-22 11:22:40 -0500
comments: true
tags: [linux, nodejs, mongodb, es6]
---
## Introduction
NodeJS has been gaining a lot of traction recently, especially for web applications. Lots of new web applications
are now being built with tools from  the Node ecosystem. A possible reason is the umpteen number of javascript
developers out in the wild. Other one is  the blazing speed of
[V8 engine](https://en.wikipedia.org/wiki/V8_(JavaScript_engine)). Heck, even shiny desktop apps are now being built
with Node viz. Popcorn Time.

In this post I will walk through everything I did to ship an API app. Ship as in - ready to serve production traffic.

### About the stack
We will be using [Express](https://expressjs.com/) as our web app framework. Express is minimalistic, fast and quietly
gets out of our way when you want to detour through the dirt road. It is unlike convention over configuration style in
[Rails](http://rubyonrails.org/) and [Play framework](https://playframework.com/) where you are good as long as you
follow the conventions recommended by the framework. Going gets tough, if you start deviating. There are frameworks
built specifically for facilitating API building. An example would be [Restify](http://restify.com/). The reason of
not choosing such a framework for this exercise is to understand the intricacies of building an API. Restify would
shield us from understanding lots of stuff.

Also, we wil be writing our javascript in [ES6](http://es6-features.org/) which is cleaner and hands down a better
language than the javascript specced in ES5. Latest version of node still doesn't fully support ES6, so we will use a
transpiler named [Babel](http://babeljs.io/) which transpiles code written by us into code conforming to ES5 for which
node has full support. In the future when node catches up, we can turn off transpilation and run our ES6 code directly
with node.

[MongoDB](https://www.mongodb.com/) will be our database. It is a document oriented NoSQL database. What that means
is - it is unlike the RDBMSes like MySQL or Oracle which let you model the world by defining a schema in form
of tables recording their properties and then linking them with primary and foreign keys. With MongoDB we model our
worldly entities in form of documents which are similar to JSON objects . The biggest advantage of NoSQL databases like
MongoDB is that they are designed to be horizontally scalable - whenever we need to increase performance, we just
add few more machines to the DB cluster and the database engine takes care of redistributing data and query traffic
across all available machines. RDBMSes are limited by theory, in terms of horizontal scalability. A discussion of this
is beyond the scope of this article, but if you are interested in knowing more about it you can browse the interwebs
for this nuisance called CAP theorem.

### About the project
We are going to build an API which can power a web application and/or a mobile app. The app is a social code snippet
sharing system where the users can share frequently encountered code snippets that are used in a developer's day to
day life for solving mundane problems which they are too lazy to code themselves. Usually, as developers we google for
our problem, find link to a stackoverflow question and copy the snippet from there and use from. In our case we let
the users share the snippet, enter a description, select the language, optionally add some tags and publish it. A user
can also thumb up a snippet or mark it as junk. We also allow some experienced users to edit the snippets and/or
descriptions for clarity. We define experience with a formula derived from number of upvotes to their questions by
other users, their age on the site and their time spent on the site. Also, we do away with logins through passwords. 
We provide OAuth based logins through Google, Facebook, Twitter etc. Apart from these functional features, we will
also do some non-functional stuff like generate an SEO friendly URL for each snippet so that Googlebot can find our
snippets and list in search results. Let's wear the product manager hat for a moment and write down the high level
product requirements for our application.

#### As a guest user I should be able to

* search for snipppets through description and/or code matches
* list snippets by language, and sort them by
  - most voted
  - most viewed
* login using Google, Facebook or Twitter auth

#### As a logged-in user I should be able to

* perform snippet operations
  - post a snippet
  - edit my posted snippets
  - view a snippet and see all previous versions of a snippet, if they have been edited
* perform profile operations
  - view my profile
  - refresh upstream information in the profile
  - view others' profiles
  - view activity of a user, like snippets posted, snippets upvoted, snippets edited etc.
* perform friendship operations
  - send friend requests to people
  - accept/reject others' friend requests
  - ignore/block a user
* perform feed operations
  - view a feed which shows my friends' activity on the website
  - like and comment on activities in the feed
* perform leaderboard operations
  - view leaderboard of snippets
  - view leaderboard of users
  - filter leaderboard by language, user location etc.
* perform messaging operations
  - send private messages to my friends
  - send private messages to non-friends
  - view my inbox, categorized with messages from friends/non-friends

### Why API?
Let's take a step back and understand why should we write an API? Why not a simple MVC web application? APIs are great
for scaling, technically as well as logistically. With an API, we define a set of operations, which cover all the
interactions of users and other systems with our application. Then we write client apps, which talk to the API and get
the job done. So, next time you swipe left in gmail to archive an email, you should know that an API call has been
made by the GMail app residing on your phone, to the GMail servers along with some parameters which did the actual
archiving of the email.

What is logistic stability? Ok, I just made up this term! With an API as contract in place, the client app can
independently scale with the actual functional implementation. We can have different people (or teams) working on the
different client apps. As long as they have the API spec they can work independently. Also, the same API can power a
native iOS app, an Android app, a mobile website and a web application. These client apps make API calls to fetch and
display relevant data and perform operations upon user interactions.

In terms of technical scalability, the people working on the backend can independently scale the most used operations
in the API. For instance, if we are getting millions of calls for our compute intensive `findBugs` operation in a
day, we can intelligently increase the resources it needs. One way would be to have a dedicated independent machine of
high configuration just for this operation in the API. And we leverage the power of distributed systems and horizontal
scalability! We will get into the details of this when we talk about shipping once we are code complete.

## Dev Env setup
Before we design our data model, let's get our hands dirty a bit, write a hello world app with Express and get MongoDB
setup on our workstation.

I am an [Arch Linux](https://www.archlinux.org/) devotee. The instructions here should work perfectly on Arch. The
onus of changing the commands to suit your package manager(apt, yum, emerge, brew) is upto you.

### Install node, npm and mongodb
The following commands will install `node`, `npm` and `mongodb` for us. `node` is the javascript runtime executable
which is actually going to run our code. `npm` is the package manager which takes care of resolving dependencies on 
third party libraries, downloading them and setting up their paths correctly so that we have a lot of pre-written
code in form of modules available at our disposal.

Mongodb will also be started and enabled for autostart. If you don't want it to be auto started everytime you boot,
don't execute the last command.

```sh
sudo pacman -Syu node npm mongodb

sudo systemctl start mongodb

sudo systemctl enable mongodb
```

As of this writing, I got node 7.2, npm 4.0 and mongodb 3.2. The javascript ecosystem is a very fast moving one. Some
of the code mentioned hereon might get obsolete in a couple of years but the general idea remains the same.

### Setup the initial package.json
In the node world `package.json` is the file which contains the details of our dependencies and run configurations.
In Rails world, it is analogous to `Gemfile` but on steroids. In java world it is kind of analogous to `build.gradle`
or a `pom.xml`.

```sh
mkdir -p ~/Documents/Projects/js-snipcode
cd !$
npm init
```
 The second command might seem unfamiliar. !$ represents the last paramter of previous command in bash and zsh. So it
will just take you to the newly created directory from the previous step.

The `npm init` is interactive and it will first ask you for a project name. Then configure the version and description.
Next it asks for the entry point for the app. You can leave it as `index.js`. Keep pressing enter to accept the
defaults for other fields or change if you like. For now we ignore the test command. A `package.json` file will be
created in the end.

### Install Express
Express is the web framework library that we are going to use. Now let's add a dependency on Express, add babel
transpilation of ES6 and get from zero to "Hello World!".

```sh
npm install express --save
npm install --save-dev babel-cli babel-preset-latest
```

Running the above commands will download Express, babel-cli and babel's latest preset. `--save` flag makes an entry in
the `dependencies` section of package.json. Guess what does the `--save-dev` flag do? You got it right! It makes an
entry in `devDependencies` in `package.json`. Dev dependencies are different from dependencies in that they
are not required for running the app. They are only used during development, for instance for transpilation during
build. Let's also add two target scripts named `build` and `start` in the `package.json`. Running `npm run build` will
tell babel to transpile everything in `src` directory recursively and output the transpiled files in `dist` directory.
`npm run start` in the root directory of our project will start our web server by running the command
`node dist/index.js`. Also, don't forget `mkdir src; mv index.js src`. Here's the updated `package.json`.

```json
{
  "name": "js-snipcode",
  "version": "1.0.0",
  "description": "API in NodeJS for the Snipcode project.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "babel src -d dist --presets latest",
    "start": "node dist/index.js"
  },
  "author": "Prabhakar Kumar",
  "license": "ISC",
  "dependencies": {
    "express": "^4.14.0"
  },
  "devDependencies": {
    "babel-cli": "^6.18.0",
    "babel-preset-latest": "^6.18.0"
  }
}
```

### Zero to Hello World!
Let's edit our `src/index.js` and write relevant code to make a Hello World app.

```javascript
import express from 'express';

const app = express();

app.get('/', (req, res) =>
  res.send('Hello World!')
);

app.listen(3000, () => 
  console.log('Server is up on port 3000!')
);
```

Now let's build and run.

```sh
npm run build
npm run start
```

If everything went well as we discussed, you will see a message on the console that says server is up on port 3000.
Now if you fire up firefox or chromium and go to [localhost:3000](http://localhost:3000), you will see the evergreen
'Hello world!' message staring at you on a white screen.

Before we start writing more code, I'll explain the meaning of the lines in our Hello World web app. But, before that
let's pivot and focus on learning our database and design a basic data model.

### Play with mongo console
Let's play with mongo console a bit to gain some familiarity with it.

```text
$ mongo
MongoDB shell version: 3.2.10
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
Server has startup warnings:
2016-12-04T07:40:24.545-0500 I CONTROL  [initandlisten]
2016-12-04T07:40:24.545-0500 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-12-04T07:40:24.545-0500 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2016-12-04T07:40:24.545-0500 I CONTROL  [initandlisten]
> show databases
local  0.000GB
> use snipcode
switched to db snipcode
> db.users.insert({name: "John Doe", country: "United States", "sex": 'M', dob: ISODate('1970-01-01')})
WriteResult({ "nInserted" : 1 })
> db.users.find()
{ "_id" : ObjectId("58441a27d01af6e5b1ae6f74"), "name" : "John Doe", "country" : "United States", "sex" : "M", "dob" : ISODate("1970-01-01T00:00:00Z") ers.insert({name: "Jane Doe", country: "United States", "sex": 'F', dob: ISODate('1970-01-01')})
WriteResult({ "nInserted" : 1 })
> db.users.find()
{ "_id" : ObjectId("58441a27d01af6e5b1ae6f74"), "name" : "John Doe", "country" : "United States", "sex" : "M", "dob" : ISODate("1970-01-01T00:00:00Z") }
{ "_id" : ObjectId("58441a5bd01af6e5b1ae6f75"), "name" : "Jane Doe", "country" : "United States", "sex" : "F", "dob" : ISODate("1970-01-01T00:00:00Z") }
> db.users.find({sex: 'F'})
{ "_id" : ObjectId("58441a5bd01af6e5b1ae6f75"), "name" : "Jane Doe", "country" : "United States", "sex" : "F", "dob" : ISODate("1970-01-01T00:00:00Z") }
```

Similar to MySQL and friends (MariaDB, Aurora), Mongodb organizes it's data in databases. We don't need to explicitly
create a database unlike MySQL. It gets created as soon as we try to write something to it. Inside a Mongodb database,
the database is organized in collections. A collection is analogous to tables in RDBMS, and more flexible. A
collection doesn't have a fixed schema. So, one document (record/row in RDBMS) can have fields different from others
in the same collection. In the above example, we switched to database named `snipcode`. Then we inserted a document
describing a user named 'John Doe'. Upon doing a `find()` (which is analogous to SQL's SELECT) on that collection we
got the record, along with a unique record ID called ObjectID. Then we inserted another record for `Jane Doe`. Then we
ran a query which returns all female users. For this, we provided the query condition `sex: 'F'` which is similar to
WHERE sex = 'F' in SQL.

As of now, this much familiarity is enough for us to get started. We will dive deeper as and when we need.


## Next part

In this part we got bootstrapped and got our hands dirty. In the next part we will start with data modeling and
continue further. Stay tuned for it.
