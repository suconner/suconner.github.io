---
title:  "Docker + Drupal + Amazon EC2"
layout: post
---

<br>
<a href="http://www.docker.io/"><img src="{{ site.url }} /assets/homepage-docker-logo.png" height="150"  hspace="30"></a>
<a href="https://drupal.org/"><img src="{{ site.url }} /assets/druplicon.large_.png" height="150" hspace="30"></a>
<a href="https://aws.amazon.com/ec2/"><img src="{{ site.url }} /assets/amazon_300.png" height="150" hspace="30"></a>

<br>
<h4>
A Whale walks into a bookstore and wants to read up on some Drupal. </h4>

<br>
<p class="lead">
For my obligatory first post i am going to walk through my experience running a turnkey dockerized
Drupal container on an Amazon EC2 instance.
</p>
First things first, you need to create an account with Amazon AWS. The free tier is 
available for a full year and you get 750 hours of micro instances a month. 
You will need a credit card to sign up.<br>
Head over to: 
<p class="lead">
<i class="fa fa-long-arrow-right"></i><a href="https://aws.amazon.com/"> https://aws.amazon.com/</a>
</p>
After you have signed up you can use the Instance Launch Wizard:

<p class="lead">
<i class="fa fa-long-arrow-right"></i><a href="https://console.aws.amazon.com/ec2/v2/home?#LaunchInstanceWizard"> https://console.aws.amazon.com/ec2/v2/home?#LaunchInstanceWizard</a>
</p>

Choose the 64 bit Ubuntu 12 LTS version (12.04.3 as of writing)
Click <strong>Configure Instance Details</strong><br>
Leave everything default and scroll to the bottom of the page<br>
Click <strong>Advanced Details</strong><br>	
Under user-data check <strong>As Text</strong>
<br><br>
Paste the following into the text box:
<pre>#include https://get.docker.io</pre>
<br>

<p class="lead">
This will install Docker while it is launching your instance.
</p>

Click <strong>Next Add Storage</strong><br>
Leave everything default<br>

Click <strong>Next Tag Instance</strong><br>
Change the <strong>Name</strong> value to Docker so that you can identify it from other instances<br>

Click <strong>Next Configure Security Group</strong><br>
Select <strong>Create New Security Group</strong><br> 
Click <strong>Add Rule</strong> and open SSH using TCP on port 22 only to your ip address (will auto populate)<br>
Click <strong>Add Rule</strong> and open HTTP using TCP on port 80 only to your ip address (will auto populate)<br>
Click <strong>Review and Launch</strong><br>
Click <strong>Launch</strong>

Select <strong>Create New Keypair</strong><br>
Give the new keypair a name<br>
Click <strong>Download Key Pair</strong>

Move the key to a secure location on your hard drive 

Click <strong>launch instances</strong><br>
Click <strong>View Instances</strong>

<br>
<p class="lead">
You should see the Docker instance that was created in the EC2 dashboard. 
</p>
<br>
Wait until the instance has finished initializing<br>
Check the box next to the tag name<br>
Click <strong>connect</strong>

Open up a terminal session<br> 
Navigate to the folder/directory that your key is located in<br>
In the terminal type:

{% highlight bash %}
$ sudo chmod 400 your_keypair_name.pem
{% endhighlight %}
 
This makes the keypair file read-only<br>

<br>
Copy the details of the <strong>Connect</strong> dialog box and paste into your terminal:
{% highlight bash %}
$ sudo ssh -i your_keypair_name.pem ubuntu@54.200.30.10
{% endhighlight %}

(The keypair and ip address should be unique to your case, do not copy those details from above)

<br>
<p class="lead">
You should now be running a shell session inside the Ubuntu Amazon Instance
</p>


In the Terminal type:
{% highlight bash %}
$ docker
{% endhighlight %}

This is a check to ensure that <strong>lxc-docker</strong> was installed and should return a list of
Docker command options. If not, make sure that you
selected the 64 bit LTS version of Ubuntu for your micro instance
and that the syntax for the include was correct. To get Docker installed at this point
you would need to follow the <u><a href="http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise-12-04-lts-64-bit">
installation</a></u> documentation provided.


<br>
<p class="lead">
We need to get Git!
</p>

Install Git:
{% highlight bash %}
$ sudo apt-get install git
{% endhighlight %}

<br>
<p class="lead">
Pull Dockerfiles....
</p>

Github user <u><a href="https://github.com/ricardoamaro">Ricardo Amaro</a></u> has been kind enough
to write a DockerFile that pulls a base Ubuntu image and installs a LAMP stack, Drupal, Drush 
and configures the web server and mysql database. 
<br>

Do some homework and read up on <u><a href="http://docs.docker.io/en/latest/">Docker</a></u> 
and its <u><a href="http://docs.docker.io/en/latest/use/builder/">DockerFiles</a></u>

<br>
Clone the docker-drupal repository 

{% highlight bash %}
$ sudo git clone https://github.com/ricardoamaro/docker-drupal.git
$ cd docker-drupal
$ sudo docker build -t ubuntu/drupal .
{% endhighlight %}

<br>
Grab some <i class="fa fa-coffee fa-2x"></i> and wait for everything to install. 

<br>
<p class="lead">
Fire up the container...
</p>

{% highlight bash %}
$ sudo docker run -d -p 80:80 -name TEST ubuntu/drupal 
{% endhighlight %}

<code>-name TEST</code> will give the container a name so that you can call TEST instead of 
the long string of numbers Docker uses by default. 

<br>
Visit the public ip address that your Amazon instance is running on

<br>
<p class="lead">
You should see a basic Drupal Bartik themed landing page
</p>

Username: <strong>admin</strong><br>
Password: <strong>admin</strong>
<br>

Go ahead and play around with your Drupal installation, add a theme, create a node, whatever you want to do.

<br>
When you are finished head back to the Terminal and type:

{% highlight bash %}
$ sudo docker stop TEST 
{% endhighlight %}

<br>
This command will stop the container from running. If you navigate to your Amazon instance's ip address
you will notice that the web server is no longer running. <br>

<br>
When you wish to continue working in your containerized environment simply execute:

{% highlight bash %}
$ sudo docker start TEST
{% endhighlight %}
<br>


Dont forget to shutdown your Amazon micro instance when you are all finished so that you dont waste your uptime allowance.<br>
If you are still in the EC2 Instances Dashboard check the box next to the instance(s) that are in a running state.<br>

Click <strong>Actions</strong><br>
Under <strong>Actions</strong> click <strong>Stop</strong> 

<br>
Obviously this is a very condensed tutorial and is only meant to help get you started. As you can
see Docker is a powerful development tool that bears some serious attention. You should start looking into 
how to commit changes to your base image, automating tasks with DockerFiles and shipping your containers. 

<br>
<p class="lead">
The rest is up to you!
</p>
