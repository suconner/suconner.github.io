---
title: “The Quest for Automated Interrogation pt 1”
layout: post
---
<br>
<a href="http://www.kali.org/"><img src="{{ site.url }} /assets/kali-site-logo2.png" width="120"  hspace="30"></a>
<a href="https://www.virtualbox.org/"><img src="{{ site.url }} /assets/VirtualboxIcon.png" height="150" hspace="30"></a>

<br>
<p class="lead">
This is the beginning of a multi-part series of a building block project aimed at finding the perfect set of ingredients for automating and mobilizing vulnerability research. 
</p>

<p>
<h4>Currently the series stacks up like this:</h4>
</p>

<p>
<h4>
pt 1: Kali Linux VirtualBox setup<br>
pt 2: Vagrant + The Vagrant Pentester setup<br>
pt 2 Supplement: Packer & Vagrant <br>
pt 3 Attacking The Vagrant Pentester<br>
pt 4 Vagrant botnet panel (Murdering Dexter, BitBot, Ra1nx1ng Bots)<br>
pt 5 Drupal pentesting shooting gallery with Docker<br>
pt 7 LXC container security <br>
pt 8 Docker, Hadoop & BinaryPig for malware traffic analysis... is it even possible?<br>
</h4>
</p>


<br>
<p class="lead">
pt 1 Kali Linux VirtualBox setup:
</p>

In order to carry out any testing you must first install and configure the attacker machine. Detail and care should be taken as this is the one virtual machine that we will not be tearing down and rebuilding repeatedly.

At this point it must be made clear that any and all testing should only be done against the localhost on the Host-only Adapter that is shared with the attacker machine. The only reason we are using NAT for the attacker machine is to pull repositories or updates. 

<br> 
<p class="lead">
Download Kali Linux iso (for this setup will be 32 bit):
</p>
<i class="fa fa-long-arrow-right"></i><a href="http://www.kali.org/downloads/"> http://www.kali.org/downloads</a>

<br>
<p class="lead">
Download and install VirtualBox:
</p>
<i class="fa fa-long-arrow-right"></i><a href="https://www.virtualbox.org/wiki/Downloads"> https://www.virtualbox.org/wiki/Downloads</a>

<br>
<p class="lead">
Create Virtual network:
</p>
Open VirtualBox<br>
In the VirtualBox menu select:<br>
VirtualBox > Preferences > Network >Host-only networks<br>
Click the <strong>+</strong> sign<br>
By default this will create the vboxnet0 adapter<br>
Click <strong>OK</strong><br>

<br>
<p class="lead">
Create Kali Linux virtual machine:
</p>

In VirtualBox click the <strong>New</strong> button<br>
Enter Name: Kali Linux<br>
Memory size: at least 1024MB<br>
Click the <strong>Create Button</strong><br>
Set File Size: at least 12GB (I usually shoot for around 20GB)<br>
Click the <strong>Create Button</strong><br>
A Kali Linux panel should have been created and now visible in your list.<br>

<br>
<p class="lead">
Configuration:
</p>
Select the Kali Linux virtual machine that was just created<br>
Click the <strong>Settings</strong> button<br>
Click the <strong>System</strong> button<br>
Select <strong>Processor</strong><br>
check <strong>Enable PAE/NX</strong> (your virtual machine will fail to boot otherwise)<br>

<br>
<p class="lead">
Setup Kali’s network adapters:
</p>
While still in <strong>Settings</strong><br>
Click the <strong>Network</strong> button<br>
Set Adapter 1 attached to <strong>NAT</strong> (so Kali can connect to the internet and the outside world)<br>
Click Adapter 2 and check the  <strong>Enable Network Adapter</strong> box<br>
Set Adapter 2 attached to <strong>Host Only Adapter</strong><br>
Under <strong>Name</strong> select <strong>vboxnet0</strong>  (recommended method for connecting multiple hosts to test or attack while segregated from the outside world)<br>

<br>
<p class="lead">
Shared resources configuration:
</p>
While still in <strong>Settings</strong><br>
Click the <strong>General</strong> button<br>
Select <strong>Advanced</strong><br>
Set <strong>Shared Clipboard</strong> & <strong>Drag and Drop</strong> to <strong>Bidirectional</strong><br>

<br>
<p class="lead">
Create a shared folder for the host machine and Kali:
</p>
While still in <strong>Settings</strong><br>
Click <strong>Shared Folders</strong> button<br>
Click the  <strong>+</strong> folder button<br>
Under <strong>Folder Path</strong> select <strong>Other</strong> and then specify a directory on your host to use<br>
Click <strong>Choose</strong><br>
Check <strong>Auto-Mount</strong>box<br>
Click <strong>OK</strong><br>
Click <strong>OK</strong> again<br> 

<br>
<p class="lead">
Installing Kali Linux:
</p>
Double click the Kali Linux panel in the VirtualBox manager list<br>
When the dialog box pops up looking for boot media, click the folder icon and browse to the iso image you<br> downloaded<br>
Click <strong>Start</strong><br>
The iso image should boot to the Kali Linux install menu<br>
Complete the installation as normal.  <br>

<br>
<p class="lead">
Post Installation Configuration:
</p>
After the Kali Linux virtual machine boots successfully:<br>
Open up the Terminal and enter: <br>
$ cd /etc/apt
$ nano sources.list

Add the following to the sources file or Virtualbox Additions will not work
{% highlight bash %}
deb http://http.kali.org/kali kali main non-free contrib
deb-src http://http.kali.org/kali kali main non-free contrib
deb http://security.kali.org/kali-security kali/updates main contrib non-free
{% endhighlight %}

<br>
Install Linux Kernel headers:
{% highlight bash %}
$ apt-get update && apt-get install -y linux-headers-$(uname -r)
{% endhighlight %}

<br>
Mount the VirtualBox Additions media:
In the Virtualbox VM menu select <strong>Devices</strong><br>
Select <strong>Insert Guest Additions CD image<strong><br>
The image will try to Autorun, it will not work so just ignore it.<br> 

<br>
Manually install the VirtualBox Additions media:
{% highlight bash %}
$ cd /media/cdrom
$ cp VBoxLinuxAdditions.run /root/
$ chmod 755 /root/VBoxLinuxAdditions.run
$ cd /root
$ ./VboxLinuxAdditions.run
$ reboot
{% endhighlight %}

<br>
Add DNS resolver:
{% highlight bash %}
$ echo "nameserver 8.8.8.8" > /etc/resolv.conf
{% endhighlight %}

<br>
Update and Upgrade:
{% highlight bash %}
$ apt-get update
$ apt-get upgrade
{% endhighlight %}


<br>
Add Host-only network adapter to interfaces file:
{% highlight bash %}
$ cd /etc/network/interfaces
$ nano interfaces
{% endhighlight %}

<br>
Add to the bottom of the file:
{% highlight bash %}
auto eth1
iface eth1 inet dhcp
{% endhighlight %}


when finished type:
Ctrl x<br>
y to save<br>
Enter to write to the file and return to the terminal.<br>

<br>
Restart the system:
{% highlight bash %}
$ reboot 
{% endhighlight %}

<br>
Check to make sure that both interfaces are working correctly:
open terminal and check to make sure both eth0 and eth1 are assigned ip addresses. 
{% highlight bash %}
$ ifconfig
{% endhighlight %}

<br>
<br>
Next: The Quest for Automated Interrogation pt. 2: The Vagrant Pentester
<hr>
<br>
<br>
<br>




