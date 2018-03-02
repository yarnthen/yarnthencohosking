---
layout: post
title: "How to install jekyll on Windows 10 PC"
excerpt: How to install jekyll on Windows 10 PC
summary: This post teaches you how to install Jekyll on Windows 10 PC
categories: [How To]
tags: [Jekyll, Windows 10, Blog, Markdown, Bash, Command Prompt]
comments: true
---

Jekyll is one of the most popular Static Site generator which have been used around by a lot of people to create their blogs and website. Compared to sites/blogs created using Wordpress or Joomla, it has the advantage of speed(which means Search Engine likes it) because Jekyll do not need to call database for data loading and will simply need to serve static pages. In addition, it is more secured than Wordpress because there is no database to be hacked. The reason why it is not so popular compared to Wordpress is because:

1. The learning curve is much higher especially for people with minimal or no programming knowledge
2. To create a Jekyll website on the most popular OS (Windows) is difficult and more manual.

I am taking this opportunity to show in this post in step by step how you can install Jekyll on Windows 10 computer.

### Disclaimer: Guides already available on official sources
The steps to install Jekyll on Windows 10 is already available on [Official Source](https://jekyllrb.com/docs/windows/). I am creating this guide to show the most straightforward way based on the official source.

Prerequisites

Your PC must be running a 64-bit version of Windows 10 Anniversary Update or later (version 1607+).

To find your PC's architecture and Windows build number, open
Settings > System > About

Look for the Windows specification and Version field.


### Install Windows Subsystem for Linux

Launch Powershell as Adminsitrator to run the below command.

{% highlight powershell %}
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
{% endhighlight %}


After installing, it should prompt for restart. Type Y to restart the computer.

<img src="{{ site.baseurl }}/images/restartpowershell.png" alt="">

### Install a Linux distribution

Install [Ubuntu](https://www.microsoft.com/store/p/ubuntu/9nblggh4msv6)(a popular Linux Distro) from Windows Store.

Once install is completed, click on Launch. This will open up a command window to key in a username and password. You will need to remember those as it will be required sometime.

<img src="{{ site.baseurl }}/images/ubuntu.png" alt="">

### Launch bash from command prompt

Once Ubuntu is installed. Launch your command prompt and run command below.

{% highlight shell %}
bash
{% endhighlight %}

Your Command Prompt instance should now be a Bash instance. Now we must update our repo lists and packages.
Run the below command in the bash prompt to update the repository list and packages.

{% highlight shell %}
sudo apt-get update -y && sudo apt-get upgrade -y
{% endhighlight %}

It should prompt you for your password you created earlier when launching ubuntu. 
Make yourself a cup of coffee or do some pushups while waiting as it will take some time for this to complete.

### Install Ruby

Install Ruby in the bash prompt using the below 4 commands

{% highlight shell %}
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.3 ruby2.3-dev build-essential dh-autoreconf
sudo gem update
{% endhighlight %}
Make yourself a cup of tea or do some more pushups while waiting for the third command as it will take some time for this to complete.

### Install Jekyll (Finally!)
Run the below command in bash to install jekyll

{% highlight shell %}
sudo gem install jekyll bundler
{% endhighlight %}

### Run Jekyll

Go to your jekyll project folder from the bash prompt and run the below command

{% highlight shell %}
jekyll serve --watch --baseurl ""
{% endhighlight %}

Now your jekyll will be running and you can access your jekyll website from any internet browser fromm address http://127.0.0.1:4000/

### Bonus Tips (Create shortcut commands for running jekyll)

You will notice there will be two commands which you will be retyping often (jekyll serve --watch --baseurl "") & (cd to your project folder). Below is a way where you can save some hassle by creating a shortcut command to run these two.

From your bash prompt type the below.

{% highlight shell %}
cd
{% endhighlight %}

This will move the location to the root folder of bash. Type the below command to update the .bashrc file. This is the file to create shortcut commands in bash.

{% highlight shell %}
vi .bashrc
{% endhighlight %}

Using vi command, insert the below in the .bashrc script. (Note replace the your_jekyll_project_path to the location of your jekyll project)

{% highlight shell %}
alias qq='jekyll serve --watch --baseurl ""'
alias ww='cd your_jekyll_project_path'
{% endhighlight %}

Save the vi session. Quit your bash prompt and launch a new one.

Now you will be able to trigger command of jekyll serve --watch --baseurl "" by simply typing qq and go to your jekyll project directory by simply typing ww.










