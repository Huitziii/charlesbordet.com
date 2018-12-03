---
layout: single
permalink: /en/shiny-aws-1/
title: "Deploy Shiny on AWS - Create a Shiny app"
description: "Learn how to deploy a Shiny app on an AWS server. It's simple, free, and fully explained in this article! Part 1: We prepare ourselves with a simple shiny app ready to be deployed!"
date: 2018-08-22
lang: en
categories: [shiny, en]
ref: shiny-server-aws-part1
header: 
    image: /assets/images/shiny-server-aws-part1/featured.jpg
---

{% include tableofcontents/shiny-server-aws-en.markdown %}

Recently, a client asked me to build an app with [R Shiny](https://shiny.rstudio.com/). This app was supposed to be used to visualized plenty of indicators and also to interact with the data.

Rather basic for a Shiny app.

So I did my magic, created the dashboard, and the client was super happy!

Except... when he asked me to deploy it on his servers.

That's when things got complicated.

![From Shiny to AWS]({{ "/assets/images/shiny-server-aws-part1/shiny-to-aws.jpg" | absolute_url }})

As a data scientist, any R script has become a piece of cake. And since Shiny is getting more and more traction, I get more and more clients who ask me for automatic reporting dashboards where the users can visualize the data and interact with it.

But the problem is when we need to host the app.

I need a server, I need to configure it, update it, administrate it, and... well, it's just not my job.

## The "stupid" method

The first time I faced the issue, our team and I decided for a completely different solution. 

We had no idea how to setup a server, nor the skills to do it anyway, so... we had to find another way.

It's not pretty.

**We installed R and the Shiny app on each individual computer for each user.**

Yep.

That's what we did.

Every time we wanted to give access to a new user, we had to go to him, install R, the packages, everything, create a shortcut written in BATCH...

When you think about it, that's kind of creative!

![Sad Face]({{ "/assets/images/shiny-server-aws-part1/sad-face.jpg" | absolute_url }}){: .align-right}

But also incredibly stupid...

* First, we still needed a way to share the data
* Then, it takes forever to install the app for each user. Such a waste of time!
* And, to maintain the app, build new features and deploy updates, it's just hell.

But we had no choice.

Not the skills. And no help from the IT department.

Today, I know there exists a wayyy better solution.

## The "smart" method

It's easy:

1. Create a server in the cloud
2. Send the app there
3. Profit.

That's all.

.

Ok, ok... I may be oversimplifying it just to make a point.

At first at least, it's not that easy.

But once you know how to do it, you can make it work in half an hour.

And today, with this series of articles, **my goal is to teach you exactly how to host a Shiny app on a server**.

So, where to start?

First, we'll use AWS.

AWS are the Amazon Web Services, i.e. all the services from Amazon in the cloud.

Of course, you could use other services, such as Azure from Microsoft, Digital Ocean, Rackspace, etc. 

Doesn't matter.

But I only know AWS, so that's what we'll use.

Important point to note: When you create a new account on AWS, you have *free access* to some services for a whole year.

Hence, everything we'll do in these articles is potentially free.

![Happy Face]({{ "/assets/images/shiny-server-aws-part1/happy-face.jpg" | absolute_url }}){: .align-right}

The benefits of this smart method are:

* You can deploy the app without moving from your chair and give your client access.
* It's easy to maintain and update, once everything is set up.
* You can easily change the server capabilities (for example if you suddenly need to handle big data queries) for a more powerful machine.
* You can add layers of security and restrict access to the app.

## What should you expect?

My goal is to create reference articles, so that every time you need to deploy a Shiny app on AWS, you can come back here and find the solution. 

Until you don't need to come back anymore.

But I know from experience it's easy to forget. So don't hesitate to come back when you need.

I will explain each step with plenty of details, screenshots, drawings, maybe even videos!

What steps? 

1. **Create a Shiny app**
2. **Create an AWS server**
3. **Install R and R Shiny on your new server**
4. **Deploy the app on the server**

These steps are enough to get started!

But if you want to go further, I have created these new extra steps:

1. **Extra: Create a domain name to have a nice URL**
2. **Extra: Secure your app with HTTPS**
3. **Extra: Protect your app with a password**

And then we'll be good!

.

Let's start right away with step 1: **Create a shiny app**

Actually, it's not exactly the goal of this article to teach you the Shiny framework. So instead of spending time on that, let's use an already existing app.

I chose the movie explorer from the Shiny gallery:

![Movie Explorer]({{ "/assets/images/shiny-server-aws-part1/movie-explorer.jpg" | absolute_url }}){: .align-center}

And I created a Github repo so that you can clone it easily: <a href="https://github.com/Huitziii/movie-explorer" target="_blank">https://github.com/Huitziii/movie-explorer</a>

Note that you'll need git and a console for this tutorial. You can download git here: <a href="https://git-scm.com/downloads" target="_blank">https://git-scm.com

Ok, now we're ready!

To continue to the next step, click here:

[Part 2 - Create an AWS server]({{ "/en/shiny-aws-2" | absolute_url }})