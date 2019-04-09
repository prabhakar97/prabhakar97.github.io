---
layout: post
title: "Run your webapps on Kubernetes for cheap"
date: 2019-01-29 22:18:53 +0530
comments: true
tags: [cloud, kubernetes, docker]
---

### TL;DR

This post details how to spin off a 3 node Kubernetes cluster on Google Cloud, paying close to 7 dollars a month for it and host multiple database backed dynamic websites and APIs.

### Intro

Kubernetes is the new kid on the block (bye bye blockchain). Listing this as an skill in your LinkedIn profile will definitely get you some recruiter attention.
It is *the* solution for all your dev-ops woes.

I used to run all of my hobby projects in docker containers with lifecycle managed by docker-compose files. I had a VM from [scaleway](https://scaleway.com) with 4 gigs of memory,
100 gigs of SSD storage and dual core x86 CPU for which I used to pay less than 5 Euros a month. I was happy until I learnt about Kubernetes and then I figured *ignorance is bliss*.
After reading this post, you will be able to spin off a 3 node Kubernetes cluster on Google Cloud with 3.75GB memory each and 1 vCPU each and pay roughly ~40 dollars a month for it!
If you do not want to pay as much, you can choose f1-micro VM size and pay close to ~7 dollars a month. It might not be as beefy as the 25 dollar cluster, but it can do the job of
hosting a few hobby webapps. With 11.25GB of memory at your disposal and three compute cores, you can do wonders - host several moderate traffic webapps backed by different databases!
Adding to that, with Google Cloud's free trial program you can have this cluster running for free for an year!

My three node cluster currently runs `mongodb`, `postgres`, `nginx`, two rails web applications, a nodeJS web application, a dotnetcore web application, a spring boot based Java REST API application
which powers an Android app, and still has a lot of resources remaining to do 3-4 times more of what it currently does. All the web apps are fronted by nginx and configured as virtual hosts,
and have their own domain names with letsencrypt https certificates automatically renewed every three months. Two of the apps use mongodb and the other two postgres.

### What is Kubernetes?

Kubernetes's official website tells this:

> Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.

This might not be quite an approachable description for everybody. Let me take a shot at it:

> Kubernetes is like an operating system, for your distributed computing cluster that lets you run your scalable applications without being bothered about nodes in the cluster going down, request routing, load balancing, networking and other devops hassles.

### Let's get started

#### Step 1: Setup a 3-node cluster on Google Cloud

Register on Google Cloud, if you already haven't. Create a project, setup billing. Click the hamburger menu icon (![menu](/images/hamburger.png)) on the top left, and in *Compute* section,
click *Kubernetes Engine*. Click the `Create Clusters` button. Make sure that in the left pane, cluster template is set to `Standard cluster`. In the right pane, name your cluster;
for example *my-awesome-cluster*. Fill in the settings as listed below:

* *Location type*: Zonal
* *Zone*: us-central1-a

Under node pools:

* *Number of nodes*: 3
* *Machine type*: `n1-standard-1` (this will cost roughly $40/month for 3 nodes). Choose `f1-micro` for $7/month instead.

Click *More node pool options* and ensure that the following are set:

* *Enable autoscaling*: off
* *Boot disk type*: Standard persistent disk
* *Boot disk size*: 10 GB
* *Enable pre-emptible nodes*: Yes - you must do this for cost cutting

Click *Availability, networking, security, and additional features* and ensure the following:

* *Enable HTTP load balancing*: No - this is costly! We will use Kubernetes's default balancer
* *Enable Stackdriver Logging service*: No
* *Enable Stackdriver Monitoring service*: No

Leave all the other settings to their sane defaults. Changing some of them might cost you extra money.

#### Step 2: Connect to your newly minted cluster

Install `kubectl` command line utility for your OS. In the *Clusters* page, click *Connect* and it will show you a `gcloud` command which downloads the necessary authentication information
for the `kubectl` command.

Once you have done that, test your connection by running `kubectl get nodes` which should return you a list of three nodes.

#### Step 4: Prepare your first application

Let's prepare a rails application that uses a `postgresql` database to deploy on this cluster. We will name it `Portal`, you are free to name whatever you want.

```bash
gem install bundler
gem install rails
rails new Portal
bundle exec rails s
```

Last command should start a local server at port 3000 and you should be able to access it at `localhost:3000`. Open `config/database.yml` and around the end change the production db connection
settings so that they look like this:

```yml
production:
  <<: *default
  adapter: postgresql
  url: <%= ENV['DATABASE_URL'] %>
```

Also, edit the `Gemfile` and add a line `gem 'pg'`. The above changes will configure the production instance of app to use postgresql instead of sqlite and will let us pass the `postgresql`
connection string through an environment variable that Kubernetes will setup for us later.

Let's create a scaffold with two fields and see whether things are working fine.

```bash
bundle exec rails g scaffold User name:string about:string
bin/rails db:migrate RAILS_ENV=development
```

This will generate necessary views and controllers which we can use to insert some data and ensure that things are working fine. It will also commit pending database migrations. Access the url
`http://localhost:3000/users` and click `New User` and add a new record.

Now that our test application is ready, let's containerize it. If you do not have docker installed, now is the time to install it and start the docker service.
Once we build the container, we want to push it to a public docker repository. [Docker Hub](https://hub.docker.com) lets you store containers for free. Register on their website then connect
your `docker` command to your docker hub account by running the following command:

```bash
docker login --username=DockerHubUsername --email=`DockerHubEmail`
```

Now let's build the container. For building it, we need to create a file named `Dockerfile` which tells docker how to build the container. Create this *Dockerfile* in rails root directory.

```docker
FROM ruby:2.6.2-alpine

RUN apk add --no-cache --update nodejs postgresql-dev libpq tzdata imagemagick
RUN apk add --no-cache --update --virtual build-deps build-base git
RUN cp /usr/share/zoneinfo/UTC /etc/localtime

COPY . /app

WORKDIR /app
RUN echo 'gem: --no-document' > /etc/gemrc
RUN bundle install --jobs 4 --without development test

RUN apk del build-deps
```

Now build and push the container to docker hub by running following commands:

```bash
docker build -t DockerHubUsername/portal .
docker push DockerHubUsername/portal
```

