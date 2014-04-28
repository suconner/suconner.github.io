---
title: “The Quest for Automated Interrogation pt 2”
layout: post
---
<br>

<a href="http://www.vagrantup.com/"><img src="{{ site.url }} /assets/index.jpg" height="150"  hspace="30"></a>

<br>
<p class="lead">
This is part two of a multi-part series of a building block project aimed at finding the perfect set of ingredients for automating and mobilizing vulnerability research.
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
pt 2 Vagrant + The Vagrant Pentester setup
</p>
<br>
Concisely, Vagrant acts as a virtualization management tool that allows you to create, configure, destroy and rebuild virtual machines in minutes. At its core are Vagrantfiles which are syntactically Ruby and handle the configuration of the machine with the help of provisioners of which could be Puppet, Chef, Docker, Salt or Bash.
<br>

Please read up on Vagrant here: <i class="fa fa-long-arrow-right"></i><a href="http://www.vagrantup.com/"> http://www.vagrantup.com/</a>
<br>

Vagrant relies on publicly available base boxes to build off of.
<br>

Current Vagrant Base Box repositories can be found at: 
<br>
<i class="fa fa-long-arrow-right"></i><a href=" https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Boxes">https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Boxes</a><br>
<i class="fa fa-long-arrow-right"></i><a href=" https://vagrantcloud.com/discover/">https://vagrantcloud.com/discover</a><br>
<i class="fa fa-long-arrow-right"></i><a href=" http://www.vagrantbox.es/">http://www.vagrantbox.es</a>


<p>
While you can pull a base box and launch a virtual machine of any type, one of the biggest strengths of Vagrant (at least within the scope of this project) is its ability to turn any VirtualBox machine into a base box quite effortlessly. 
Its definitely important for a web developer, sysadmin or security researcher to be able to launch a pre-configured environment and concentrate on their essential duties, but I also see a huge benefit in being able to make quick and dirty snapshots of the machines in different phases to export and share with others. 
Think of situations where you would want others to test a super specific proof-of-concept or master a penetration test of a critical asset prior to an actual red team exercise.
<br>
<br>
*** While I focus on VirtualBox here, Vagrant has added support for most other virtualization platforms such as VMware and Hyper-V
</p>
<br>
<p class="lead">
Install Vagrant
</p>
<i class="fa fa-long-arrow-right"></i><a href="https://www.vagrantup.com/downloads"> https://www.vagrantup.com/downloads</a>

<br>
If you have an older version of Vagrant that was installed as a Ruby gem you need to remove it before installing. If you use RVM you may notice issues when executing Vagrant commands. Removing RVM path variables from your bash profiles (~/.bashrc, ~/.profile, ~/.bash_profile) should correct this.

<br>
Verify that Vagrant is installed and working by typing in the terminal:
{% highlight bash %}
$ vagrant -v
# This will print out the Vagrant version number
{% endhighlight %}
 

<br>
<p class="lead">
Download Ubuntu LTS Precise 32 basebox:
</p>
{% highlight bash %}
$ vagrant box add precise32 http://files.vagrantup.com/precise32.box
{% endhighlight %}
<br>
Check to make sure the Precise32 box installed correctly:
{% highlight bash %}
$ cd ~/.vagrant.d/boxes
{% endhighlight %}


<br>
<p class="lead">
Create directory for Vagrant environment:
</p>
{% highlight bash %}
$ mkdir ~/vagrant-test
$ cd vagrant-test
{% endhighlight %}

<br>
<p class="lead">
Initialize directory with Vagrantfile:
</p>
{% highlight bash %}
$ vagrant init precise32
{% endhighlight %}

<br>
<p class="lead">
Import box and start Vagrant virtual machine:
</p>
{% highlight bash %}
$ vagrant up
{% endhighlight %}

<br>
Notice that there is now a running Vagrant virtual machine with the name of your directory (“vagrant-test”). You can log-in using the GUI by opening the machine as you would any other VirtualBox machine. The user  is “vagrant” and the password is also “vagrant.”

<br>
<p class="lead">
Basic Usage:
</p>

SSH into the vagrant operating system:
{% highlight bash %}
$ vagrant ssh
{% endhighlight %}
<br>
Shared directory between host and box can be accessed while inside vagrant box by going here:
{% highlight bash %}
$ cd /vagrant
{% endhighlight %}
<br>
To suspend or shutdown open a new terminal:
{% highlight bash %}
$ vagrant suspend
{% endhighlight %}
<br>

To destroy, exit the vagrant box and type:
{% highlight bash %}
$ vagrant destroy
{% endhighlight %}

<br>
...and to recreate:
{% highlight bash %}
$ vagrant up
{% endhighlight %}

<br>
Its just that simple. 

<br>
The provisioners in the Dockerfile will only run once. If you make changes to it you must call:
{% highlight bash %}
$ vagrant provision
{% endhighlight %}

<br>
The above will run all provisioners. In the case that you only want to run one set of provisioners such as the shell, call:
{% highlight bash %}
$ vagrant provision --provision-with shell
{% endhighlight %}

<br>
<p>
After you are comfortable with all of the general uses of Vagrant you can begin creating custom boxes from pre-existing virtual machines. It could be a VirtualBox machine you have been working on or a Vagrant machine you have been testing, it doesnt matter. 
Once you have chosen a virtual machine to clone enter the root of the VirtualBox directory.
</p>
On OS X it looks something like this:
{% highlight bash %}
$ cd ~/VirtualBox\ VMs/
{% endhighlight %}
<br>
To Create the box you need to copy the full VirtualBox machine name and place it after <strong>--base</strong>.
After <strong>--output</strong> you should place the name you wish the new box to be called with a .box extension:

<p class="lead">
Creating the Base Box:
{% highlight bash %}
$ vagrant package --base vagrant-pentester_default_1397852517894_35688 --output my_new_test.box
{% endhighlight %}
<p>
Say you want to give this new box to a friend or send it to a colleague but you don’t want them to have to create their Vagrantfile and or download it from github. If you’re picky like me you will want that Vagrantfile packaged up with the new box so that when the person you are sending hits <strong>vagrant init</strong> the box will be configured with the default Vagrantfile and the Vagrantfile you specified. 
</p>

<br>
<p class="lead">
Create Base Box specifying the Vagrantfile:
{% highlight bash %}
$ vagrant package --base vagrant-pentester_default_1397852517894_35688 --vagrantfile ~/VirtualBox\ VMs/Vagrantfile --output my_new_test.box
{% endhighlight %}

<br>
<p>
How this works is, during <strong>vagrant init</strong> or any other command for that matter, Vagrant looks in a series of a paths for a Vagrantfile to use. It executes a merge on all the Vagrant files configuration settings as it moves along. Now, you may get a little confused when you open up the Vagrantfile that is created in the root of your directory during <strong>vagrant init</strong> because it will just be the plain vanilla default file. If you dig a little deeper you can find the Vagrantfile that was copied into the box during creation. 
<br>
{% highlight bash %}
$ cd ~/.vagrant.d/boxes/my_new_box/0/virtualbox/include/_Vagrantfile
{% endhighlight %}
</p>

<br>
When you run <strong>vagrant up</strong> in the terminal you should see any custom configurations or port forwardings fire off while its loading. 
<br>

<br>
<p class="lead">
The Vagrant Pentester
</p>

The idea for The Vagrant Pentester was to put 9 of the most well known vulnerable web application training exercises together and cut out the monotonous setup and configuration that keeps some of the beginners away.
<br>
In creating it i used all of the methods described above and added a few Puppet and Shell provisioners. In future releases i will be removing many of the clunky Bash scripts in favor of modular Puppet manifests for each web application. I just didn't have the development time to do the conversion right now...
<br>
If you have been following along you should already have Vagrant installed and have a pretty good working relationship with it. 

<br>
<p class="lead">
Install The Vagrant Pentester
</p>
Simply download the release from Github:<br>
<i class="fa fa-long-arrow-right"></i><a href="https://github.com/p00gz/vagrant-pentester/releases/tag/v1.0">https://github.com/p00gz/vagrant-pentester/releases/tag/v1.0</a>

<br>
<p class="lead">
Add The Vagrant Pentester to the list of available boxes:
</p>
{% highlight bash %}
$ cd ~/Downloads
$ vagrant box add vagrant-pentester vagrant-pentester.box
{% endhighlight %}

Make sure the box installed correctly:
{% highlight bash %}
$ cd ~/.vagrant.d/boxes
{% endhighlight %}

<br>
<p class="lead">
Create working directory:
</p>
{% highlight bash %}
$ cd ~
$ mkdir directory_name
$ cd directory_name
$ vagrant init vagrant-pentester
$ vagrant up
{% endhighlight %}

<br>
<p>
And Huzzah. You should now have a fully configured penetration testing training suite platform at your disposal. 
The Vagrantfile will forward Apache served ports to localhost:8888 and Tomcat served ports to localhost:8999 on your host machine. This allows you to access the exercises from your host browser rather than having to log-in to your virtual machine. 
<br>
In part 4 we will actually pit Kali Linux against the Vagrant Pentester and judge its effectiveness and ease of use. 
For now, follow the instructions below for accessing each of the web applications:
</p>

<p>
<h3>Bodge It Store</h3>
https://code.google.com/p/bodgeit/<br>
<br>
access the application at:<br>
localhost:8999/bodgeit<br>
</p>


<p>
<h3>bWAPP (Buggy Web Application)</h3>
http://www.mmeit.be/bwapp/<br>

<br>
access the application at:<br>
localhost:8888/bWAPP/install.php<br>
<br>
Follow the "click here to install bWAPP" link<br>
<br>
login: bee<br>
password: bug<br>
</p>


<p>
<h3>DVWA: Damn Vulnerable Web Application</h3>
http://www.dvwa.co.uk/<br>

<br>
access the application at:<br>
localhost:8888/DVWA/setup.php<br>
<br>
select create/reset database<br>
<br>
Login at:<br>
localhost:8888/DVWA/index.php<br>
<br>
username: admin<br>
password: password<br>
</p>

<p>
<h3>Exploit KB</h3>
http://exploit.co.il/blog/<br>

<br>
access the application at:<br>
<br>
localhost:8888/exploit/index.php<br>
</p>


<p>
<h3>Mutillidae Project</h3>
http://www.irongeek.com/i.php?page=mutillidae/mutillidae-deliberately-vulnerable-php-owasp-top-10<br>

<br>
access the application at:<br>
localhost:8888/mutillidae<br>
<br>
Click "Setup/Reset DB" link
</p>


<p>
<h3>Puzzlemall</h3>
https://code.google.com/p/puzzlemall/<br>
<br>
install the application at:<br>
localhost:8999/puzzlemall/install/initialize.jsp<br>
<br>
username: root<br>
password: mysql<br>
<br>
access the application:<br>
localhost:8999/puzzlemall/<br>

</p>


<p>
<h3>SQLi Labs</h3>
https://github.com/Audi-1/sqli-labs<br>

<br>
access the application:<br>
localhost:8888/sqli-labs/index.html<br>
<br>
Click "Setup/reset Database for labs<br>
</p>


<p>
<h3>Wavsep</h3>
https://code.google.com/p/wavsep/<br>
<br>
install the application at:<br>
localhost:8999/wavsep/wavsep-install/install.jsp<br>
<br>
username: root<br>
password: mysql<br>
host: localhost<br>
port: 3306<br>
<br>
access the application:<br>
localhost:8999/wavsep/<br>
</p>


<p>
<h3>OWASP Web Goat</h3>
https://www.owasp.org/index.php/Category:OWASP_WebGoat_Project<br>

<br>
access the application:<br>
localhost:8999/WebGoat/attack<br>
<br>
username: webgoat<br>
password: webgoat<br>
<br>
If you get locked out for entering the wrong password close the browser completely and reload page.
</p>


<br>
<br>
Next: The Quest for Automated Interrogation pt. 3: Heartbleed with Docker & Packer
<hr>
<br>
<br>
<br>