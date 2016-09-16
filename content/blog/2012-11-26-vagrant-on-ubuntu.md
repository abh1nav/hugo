+++
title = "Vagrant on Ubuntu"
date = "2012-11-26T21:21:00-05:00"
tags = ["linux", "ubuntu", "vagrant"]
type = "post"
aliases = [
    "/blog/2012/11/26/vagrant-on-ubuntu/"
]
+++

Getting vagrant installed on ubuntu isn’t as easy as it should be (as of v1.0.5 on 12.10). Here are a few solutions to make your life easier:<!--more-->

*Bug 1*:  
I noticed that ruby + gems didn’t exist on a vanilla 12.10 install and that installing the official .deb from vagrantup.com resulted in a vagrant not found error.
<br><br>
*Fix 1*: 

<pre>
sudo apt-get install ruby1.8 ruby1.8-dev rubygems1.8 virtualbox-ose
sudo gem install vagrant
</pre>
  
<br><br>
*Bug 2*:    
After running my first vagrant up with a precise64 base box, I noticed that the guest VM was having trouble with resolving domain names. Peeking at the `/etc/hosts` file, I saw an entry that read:`127.0.1.1  precise64`.  
  
After correcting this to `127.0.0.1 precise64`, it still froze when pip installing some python modules. This was strange considering the guest was configured to run in bridge mode. 
  
*Fix 2*:  
The following settings cured it:

<pre>
VBoxManage modifyvm "myvagrantbox_23901293" --natdnshostresolver1 on
VBoxManage modifyvm "myvagrantbox_23901293" --natdnsproxy1 on
</pre>

Here, **myvagrantbox_23901293** is the name assigned to the guest in virtualbox by vagrant and can be found via the virtualbox UI.