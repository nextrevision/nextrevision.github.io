---
title:  "Up and Running with OpenSwitch"
date:   2015-11-11
description: Get started with OpenSwitch using Vagrant and Docker
---

## An Introduction to OpenSwitch

HP [recently announced](http://www8.hp.com/us/en/hp-news/press-release.html?wireId=1989827) the arrival of [OpenSwitch (OPS)](http://openswitch.net/), a new addition to the Network Operating System space that runs entirely on top of Linux. The project has open sourced all of their underlying components in a [self-hosted Git repository](http://git.openswitch.net/cgit/), where you can find and clone any of these components directly.

There are some philosophical differences from that of competitor [Cumulus Linux](https://cumulusnetworks.com), namely the inclusion of an operator CLI to manage all switch configuration in addition to a REST API. Similar, however, to Cumulus, they offer a way to get started with the OS by using [Vagrant](https://www.vagrantup.com).

For those wanting to dive a bit deeper into the internals of OpenSwitch, I highly recommend watching the following videos from the [ONUG Conference](http://opennetworkingusergroup.com) held recently: [OpenSwitch Architecture](https://vimeo.com/144749345) and [Discussion and Demo](https://vimeo.com/144750531).

## Installation

Getting an environment setup running OpenSwitch is relatively easy. The project has some [documentation tucked away](http://git.openswitch.net/cgit/openswitch/ops-docs/tree/quick-start.md) on how to do just that using Vagrant.

First, download the [supporting Vagrant files](https://archive.openswitch.net/vagrant/ops-vagrant.zip) from the OpenSwitch archive and unpack them to a directory:

{% highlight bash %}
curl -o ops-vagrant.zip https://archive.openswitch.net/vagrant/ops-vagrant.zip
unzip ops-vagrant.zip
cd vagrant
{% endhighlight %}

Then we need to fix a potential issue with a UID mismatch by overwriting to our own:

{% highlight bash %}
echo $(id -u) > host/.vagrant/machines/default/virtualbox/creator_uid
{% endhighlight %}

Next, we will go ahead and boot the Vagrant VM and start the OPS build. You must have Vagrant installed as well as the Vagrant plugin `vagrant-reload`. If you do not already have Vagrant, you can download an installer from their [downloads page](https://www.vagrantup.com/downloads.html). The Vagrantfile also uses [VirtualBox](https://www.virtualbox.org/) as the backend, so ensure you have that installed as well.

Let's install the vagrant plugin:

{% highlight bash %}
vagrant plugin install vagrant-reload
{% endhighlight %}

Then boot the VM using VirtualBox (this will take a while):

{% highlight bash %}
vagrant up
{% endhighlight %}

What you will see happening next is the `ubuntu/trusty64` base box importing and Vagrant booting the VM. Then a few build dependencies will begin to be installed in the VM followed by a build of OPS and import as a Docker image. This will take a little while due to building OPS and once it's complete, we can login to our box running OPS.

{% highlight bash %}
vagrant ssh
{% endhighlight %}

Now that we are logged in, we are presented with a BASH prompt. To access the OPS CLI, we can simply enter:

{% highlight bash %}
vtysh
{% endhighlight %}

You can then explore the CLI with familiar commands such as `?`, `conf t`, and `show run` to get started.
