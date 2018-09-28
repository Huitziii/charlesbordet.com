---
layout: single
permalink: /en/shiny-aws-3/
title: "How to deploy a Shiny app on AWS - Part 3"
date: 2018-09-01
lang: en
ref: shiny-server-aws-part3
header:
    image: /assets/images/shiny-server-aws-part3-featured.png
---

{% include shiny-server-aws-en.markdown %}

Last time, we stopped right after the creation of your first server on AWS.

In this new article, we'll learn:

* how to access the server with SSH
* how to install R and R Shiny Server on it

## How to access your new server?

Once your *instance* is created, AWS will display a **Connect** button. If you click on it, you'll get something like that:

![Bouton Connect]({{ "/assets/images/shiny-server-aws-part3-connect.png" | absolute_url }}){: .align-center}

These are the instructions to connect to your server.

But it's not *that* easy, so let's take it slowly. Plus, it's different depending on your OS.

**For Windows users**

If you're on Windows, you'll need to download an SSH client, since the OS doesn't have one natively.

The most common one is PuTTY, which you can download on [https://www.putty.org/]("https://www.putty.org/").

Next, you'll have to transform the key you downloaded in the previous article. Remember, the file with the *pem* extension.

![PuTTYgen]({{ "/assets/images/shiny-server-aws-part3-puttygen.png" | absolute_url }}){: .align-center}

1. Run PuTTYgen (type "PuTTYgen" in the start menu)
2. Choose RSA.
3. Select the *pem* file that you downloaded (you'll need to activate the option to see all file types).
4. Save the private key with the same name of your *pem* file (except the extension will be different, with *ppk*).

Then, run PuTTY and configure it with what AWS told you:

![PuTTY config]({{ "/assets/images/shiny-server-aws-part3-putty-config-en.png" | absolute_url }}){: .align-center}

1. In Host Name (or IP address), copy/past the value you'll find at the 4th step of the above screenshot (for me, it's *ec2-52-59-238-223.eu-central-1.compute.amazonaws.com*).
2. Port: 22
3. Connection type: SSH
4. Go to Connection/SSH/Auth, and load the *ppk* key that you saved a few seconds ago.

Finally, click "Open", type "ubuntu" when you're asked **login as**, say "Yes", and a console will open!

That's it.

You're in!

![PuTTY console]({{ "/assets/images/shiny-server-aws-part3-putty-console.png" | absolute_url }}){: .align-center}

**For Ubuntu and Mac users**

If you're not on Windows, things are way easier.

Open a console, change the directory to be in the one that contains the *pem* key, and type the instruction given by AWS.

For example, for me it's:

{% highlight shell %}
charles@blue:~$ cd Downloads/
charles@blue:~/Downloads$ ssh -i "tutorial.pem" ubuntu@ec2-52-59-238-223.eu-central-1.compute.amazonaws.com
{% endhighlight %}

Say "yes", and you're in!

If you get an error message such as "WARNING: UNPROTECTED PRIVATE KEY FILE!", it's normal. You have to change the rights of the file by typing:

{% highlight shell %}
chmod 400 tutorial.pem
{% endhighlight %}

## How to install Shiny Server

Next step is to install Shiny Server.

And of course, before that, we'll need to install R!

The Ubuntu repos usually contain an outdated version of R. So instead, we'll first add the CRAN repo:

{% highlight R %}
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
sudo add-apt-repository 'deb [arch=amd64,i386] https://cran.rstudio.com/bin/linux/ubuntu xenial/'
{% endhighlight %}

**Important note**: At the time of the writing of this article, AWS still uses Ubuntu 16.04 LTS (i.e. the stable version of April 2016). But Ubuntu 18.04 LTS is now out and I guess that AWS will make the switch at some time in the future. 

You can easily check your version of Ubuntu with the welcome message when you log in. For me it's written:

{% highlight shell %}
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1065-aws x86_64)
{% endhighlight %}

If you're using Ubuntu 18.04, then you will need to change 'xenial' by 'bionic' in the above instructions.

Once it's done, the CRAN repo is added and you can install R:

{% highlight shell %}
sudo apt update
sudo apt install r-base-dev
{% endhighlight %}

Once R is installed, we also need to install the *shiny* package before installing Shiny Server.

Run R in admin mode and install the package:

{% highlight shell %}
sudo R
> install.packages("shiny")
{% endhighlight %}

You can quit R by typing *q()* and *n*.

Ok.

That's progress.

Final step: Download the last version of Shiny Server.

Go on [https://www.rstudio.com/products/shiny/download-server/](https://www.rstudio.com/products/shiny/download-server/) and scroll down at the bottom of the page to see the instructions, with the version number.

For me, it's:

{% highlight shell %}
sudo apt-get install gdebi-core
wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.7.907-amd64.deb
sudo gdebi shiny-server-1.5.7.907-amd64.deb
{% endhighlight %}

The last two lines might be different if a new version is released.

Once you launch everything, say "yes", and everything will get installed!

That's it!

You don't know it yet, but you already have a Shiny app running on your server now.

*How do I see it?*

That's what we'll see in the next part...

[Part 4 - Deploy the app on the server]({{ "/deploy-shiny-app-on-aws-part-4" | absolute_url }})