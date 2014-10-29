---
layout: post
title: Speeding up JRuby/Rails on Vagrant and VirtualBox
author: Joseph Turner
tags: vagrant devops
category: DevOps
---
After much head banging (and not the [good
kind](https://www.youtube.com/watch?v=389DkzjHpus)) over the slow
startup time of Rails, I finally decided to bite the bullet and figure
out what was going on. I have been plagued by multi-minute startups in
my development environment for a while, and I just couldn't take it
anymore. Because we develop
[Interlock](http://www.mobilesystem7.com/interlock/) in a virtual machine
set up by [Vagrant](http://www.vagrantup.com/) atop [VirtualBox](https://www.virtualbox.org/), and because the slow-to-start parts of our
software use [Rails](http://rubyonrails.org/) atop [JRuby](http://jruby.org/) atop the venerable [Java](https://www.java.com/en/), there were quite
a few moving pieces to isolate and try to speed up.

My first thought was that it was OpenJDK that was slowing things down,
so I installed the Oracle JDK. If this yielded any speedups, they
weren't large enough to notice qualitatively.

Next, I thought JRuby itself was the culprit. I found [a handy
guide](https://github.com/jruby/jruby/wiki/Improving-startup-time) the
project owners maintain, and tried a number of the suggestions. I found
that using the JRuby flags `--dev` and `-J-Djruby.launch.inproc=true`
did yield some significant speedups, probably 10-15%. This wasn't going
to alleviate the bruise on my forehead though, so I had to dig deeper.

As an aside, you can always pass in these flags to JRuby using the
JRUBY_OPTS environment variable. For example, if you wanted to start the
Rails console with the above flags, you could use
    
    JRUBY_OPTS="--dev -J-Djruby.launch.inproc=true" rails c

After reading one of the last-ditch suggestions in the article above, 
I decided that the problem might be in the Ruby code itself. I added the
`--profile` flag to a `rails c` call to get some information about where the Ruby functions
were spending their time. Conspicuously, almost all of the time was
being spent in IO functions like `Dir.[]`. This suggested that the
problem was perhaps in VirtualBox itself.

I started Googling for things like `slow virtualbox io`, but that didn't
bear much fruit. There were lots of suggestions like 'use SATA and not
IDE' which my setup was already doing. So I tried doing some disk IO
inside the VM to see what the throughput rates were, and got some
surprising results.

    vagrant@precise64:/vagrant$ time dd if=/dev/zero of=/tmp/file bs=1M
    count=1024
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 1.03046 s, 1.0 GB/s 

That's not quite as fast as my SSD-havin' Mac host, but it's pretty
quick. What's going on here? Oh right, my project lives in /vagrant
which is not really a part of the VM filesystem but is a mounted shared
directory! It must be shared directory performance that is the dog. A
quick search yielded [this Vagrant documentation
page](http://docs.vagrantup.com/v2/synced-folders/nfs.html). TL;DR:

> In some cases the default shared folder implementations (such as
VirtualBox shared folders) have high performance penalties.

I also found [this page](http://docs-v1.vagrantup.com/v1/docs/nfs.html)
with some older Vagrant documentation that suggests that the problem is
with folders with lots of files:

> It’s a long known issue that VirtualBox shared folder performance
degrades quickly as the number of files in the shared folder increases.
As a project reaches 1000+ files, doing simple things like running unit
tests or even just running an app server can be many orders of magnitude
slower than on a native filesystem (e.g. from 5 seconds to over 5
minutes).

A ha! How many files do we have in our project?

    vagrant@precise64:/vagrant$ find | wc -l
    44086

Looks like we found our culprit. To fix, I enabled private networking
and NFS for the shared folder, as suggested in [the documentation linked
above](http://docs.vagrantup.com/v2/synced-folders/nfs.html). YMMV, but
for me, on a Mac host running an Ubuntu guest, it involved adding these
lines to my Vagrantfile:
    
    config.vm.network :private_network, ip: "10.11.12.13"
    config.vm.synced_folder ".", "/vagrant", type: "nfs"

Afterwards, I got a DHCP error when trying to reload my VM. I found a
[bug report](https://github.com/mitchellh/vagrant/issues/3083) that suggested that VirtualBox had a default DHCP instance
that was colliding, and suggested this a workaround. Run this on the
host machine:

    VBoxManage dhcpserver remove --netname HostInterfaceNetworking-vboxnet0

With that working, Vagrant will get to a point where it needs to edit
`/etc/exports`, your list of NFS exports on the host. You'll need to
provide your password to edit this file.

Afterwards, I was getting a timeout error from the VM while trying to
mount the NFS drive. I tried a few more things, but sometimes the [old
magic is best](http://www.penny-arcade.com/news/post/2008/05/19):
I rebooted the host and the VM booted like a charm.

I tested my Rails startup time, and got between a 5X and 10X speedup. My
forehead feels better already.

