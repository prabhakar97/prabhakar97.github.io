---
layout: post
title: "Run your webapps on Kubernetes for cheap"
date: 2019-01-29 22:18:53 +0530
comments: true
tags: [cloud, kubernetes, docker]
---
Kubernetes is the new kid on the block (bye bye blockchain). Listing this as an skill in your LinkedIn profile will definitely get you some recruiter attention. It is the solution for all your dev-ops woes.
I used to run all of my personal projects in docker containers managed by several docker-compose files. When I learnt what Kubernetes can do automatically for free, I was amazed.
In this post I will take a shot at helping you getting your webapps up and running on Kubernetes. This post assumes that you have some familiarity and/or experience with docker. I will take an example
of a NodeJS Express webapp here, but the concepts directly translate to other popular frameworks like Ruby on Rails.
