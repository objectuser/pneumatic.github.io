---
layout: post
title:  "Vagrant Provisioning"
date:   2015-03-21 10:00:00
categories: vagrant
comments: true
---

Recently, I was setting up MySQL for a specific Pneumatic.IO test. I was about to install MySQL when I remembered reading about [Vagrant](https://www.vagrantup.com/). Now having used it, it's pretty insane that I hadn't used it before.

Anyway, I searched for a Vagrant config and found one using shell provisioning. I used to install and run MySQL and it was fine. But this got me thinking about all of those other provisioning options that Vagrant supports: Ansible, Chef, Puppet, Salt ... it would be interesting to compare provisioning with a shell script vs. provisioning with those tools. I had been reading about them but hadn't used them yet. It seemed like a good opportunity to at least begin to understand the basics of some of those tools.

So that's what I did. There are obvious limitations in terms of thinking of this as any sort of exhaustive comparison. I'm just doing this in Vagrant with a single box and only provisioning MySQL. So take from it what you care to.

As a reference point, I used this [vagrant-lamp](https://github.com/mattandersen/vagrant-lamp) repository (thanks, dude), which got me up and running quickly with Vagrant shell provisioning. The only modifications I made were to reduce the configuration to MySQL.

I also did a bit of timing on the setup. This is for my own purposes in using Vagrant and may be useless to you, but you don't have to use the information.

Here's my Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "forwarded_port", guest: 3306, host: 3306
  config.vm.provision "shell", path: "provision.sh"
end
```
You can see the provision.sh from the link above. As I said, I just removed the PHP and Apache stuff to focus on MySQL.

Here are the time outputs. First `vagrant up`:

```
real	1m26.335s
user	0m4.364s
sys	0m2.044s
```

Then `vagrant halt` and then here is the next `vagrant up`:

```
real	0m22.137s
user	0m3.112s
sys	0m1.277s
```

In my next post, I'll talk about [Puppet](https://puppetlabs.com/).
