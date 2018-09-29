---
layout: single
permalink: /en/shiny-aws-2/
title: "How to deploy a Shiny app on AWS - Part 2"
date: 2018-08-30
lang: en
ref: shiny-server-aws-part2
header:
    image: /assets/images/shiny-server-aws-part2/featured.jpg
---

{% include tableofcontents/shiny-server-aws-en.markdown %}

We stopped last time at the creation of the Shiny app. Just as a reminder, you can find it on Github: [https://github.com/Huitziii/movie-explorer]("https://github.com/Huitziii/movie-explorer")

Today, we are going to **create an AWS server to host our app**.

## Create a free AWS account

The first step is to create an account on AWS.

Go on [https://aws.amazon.com]("https://aws.amazon.com/") and create an account. The steps are easy to follow so I trust you can do it by yourself.

Note that Amazon will ask for a payment method, since for every paid service you used, you'll need to pay for it.

But don't worry. When you open a new account, some services are free so that you can try without burning any cash!

Also note that getting a server can be very cheap. 

I'm using AWS to host my website at the cost of $10/month. Plus, I can also host my Shiny app, an RStudio Server, a Jupyter Notebook, and many more stuff.

## Create your first server on EC2!

Once your account is created, log in and you'll land on a page with all the available services:

![Services AWS]({{ "/assets/images/shiny-server-aws-part2/aws-services.jpg" | absolute_url }}){: .align-center}

In the search box, type *EC2*.

![Kesako]({{ "/assets/images/shiny-server-aws-part2/kesako.jpg" | absolute_url }}){: .align-right}

EC2 means Elastic Compute Cloud.

It's basically a service in the cloud that is used for computation. Plus, it's elastic, meaning you can stretch its power as much as you want. It's flexible.

You can use it to:

* create web services (like hosting a website, or .... a Shiny app!)
* do computations (such as training a machine learning model)
* store data

It's better to use S3 to store data, but you'll still need to put a bit of data of your EC2 machine (like your website files, your R scripts for the Shiny app, etc.)

The huge advantage of this service, compared for example to owning your own server, is that you can change the config anytime.

Today we won't need a big one for our Shiny app.

But say that tomorrow you want to run a heavy script on multiple cores and you need many GB of ram. No prob. Simply upgrade the machine to a powerful one. Run the script. Downgrade the machine.

That's it.

You'll pay the powerful machine only for the couple of hours you used it.

Super convenient.

But anyway.

We're not here for that. 

**Let's create your first server!**

In the search box, type *EC2* and enter the service.

To the left, in the menu, click on Instances. An instance is a server.

And then, click on *Launch Instance*.

## Step 1: Choose your AMI

![Choose your AMI]({{ "/assets/images/shiny-server-aws-part2/ami.jpg" | absolute_url }}){: .align-center}

An AMI, for *Amazon Machine Image*, is a sort of pre-packaged server with plenty of things ready-to-use inside.

Some AMIs are prepared by Amazon.

You can create your own.

And you can even rent some to third-party companies (for example, with proprietary software inside).

We'll use a very basic AMI, the *Ubuntu Server 16.04 LTS*.

Click on Select.

## Step 2: Choose your instance type

![Choose your instance type]({{ "/assets/images/shiny-server-aws-part2/instances.jpg" | absolute_url }}){: .align-center}

The instance type corresponds to the power of your machine.

Do you need a small one or a very powerful one?

In this guide, no need for a costly one. 

We will use the *t2.micro* type since it's the only one that you can use for free when you have a new account.

It contains a CPU with only 1 core, only 1 GB of RAM, and an okay internet connection.

That will be enough for us. But if at some point you want to upgrade and you're not sure which one to pick, simply visit the [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/).

Then, click on *Next: Configure Instance Details*.

We won't modify anything in the next screen, so directly click on *Next: Add Storage*.

## Step 3: Choose the storage size

![Choose the storage size]({{ "/assets/images/shiny-server-aws-part2/ebs-en.jpg" | absolute_url }}){: .align-center}

By default, AWS gives you 8 GB of space

It's up to you to decide whether you'll need more or not.

Note that to stay within the free offer of a new account, you can to keep it below 30 GB.

For this tutorial, you can keep it at 8 GB.

Now, config is done!

You can click on *Review and Launch*!

A final screen will summarize all the options you picked. Give it a look and validate the launch of your instance by clicking on *Launch*!

## Step 4: Create a key pair

![Create a key pair]({{ "/assets/images/shiny-server-aws-part2/keypairs.jpg" | absolute_url }}){: .align-center}

Oh well... Yet another screen. What's this thing with keys?

It's simply a way to protect the access to your server.

Since it's connected to the Internet, you can't really keep the door open, or anyone will mess up with your server.

Not cool.

Instead, you'll create a key pair in the form of a file that you'll keep on your computer. 

Every time you want to connect to your instance, you show up this file and you'll get access.

Important: Do not lose this file. If you do, you will lose access to your server at the same time and you'll have to restart from scratch!

Pick a name for you key, and download it.

## Congrat's! You have created your first server!

After a few seconds, your instance will be ready and you'll see it in the Instances menu.

![New instance]({{ "/assets/images/shiny-server-aws-part2/newinstance-en.jpg" | absolute_url }}){: .align-center}

In the next chapter, we'll see how to connect to your server with SSH and we'll install R and R Shiny on it!

Keep reading:

[Part 3 - Install R and R Shiny on your new server]({{ "/en/shiny-aws-3" | absolute_url }})
