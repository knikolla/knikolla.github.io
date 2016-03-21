---
layout: post
title: Federated OpenStackClient
---

<div class="message">
  OpenStackClient proof of concept to send commands to federated Service Providers.
</div>
<!--more-->


## Why?
Currently there is no way to performs calls to the federated Service Providers using OpenStackClient or the various python-\*client.

With a new command line option `--os-service-provider`, users should be able to direct that call to the specified service provider.

[Here](http://paste.openstack.org/show/491281/) is a short demo on which we query Glance for the image list in the Service Provider and boot an instance there using one of the images.

There is no cross-service federated communication yet, which is something we are currently working on in the [Massachusetts Open Cloud](http://info.massopencloud.org) as the [Mix & Match Federation](http://info.massopencloud.org/blog/mixmatch-federation/) project.  

## How?
A new `--os-service-provider` command line option parameter is added. This is the federated service id as registered in Keystone.

{% gist aae43a75e1304170b97b shell.py %}

A new auth object is created and assigned as `self.auth`. I'm currently doing this in `auth_ref` but a more appropriate place would be `setup_auth`.

{% gist 97d3a0562c410c038100 clientmanager.py %}

The entire diff can be viewed [here](https://gist.github.com/knikolla/5985bd18e2f075d9e3af).
