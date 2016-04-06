---
layout: post
title: K2K OpenStackClient
---

<div class="message">
  OpenStackClient proof of concept to send commands to federated Service Providers using K2K.
</div>
<!--more-->


## Why?
Currently there is no way to performs calls to the federated Service Providers using OpenStackClient or the various python-\*client.

With a new command line option `--sp`, users should be able to direct that call to the specified service provider using K2K.

[Here](http://paste.openstack.org/show/491281/) is a short demo on which we query Glance for the image list in the Service Provider and boot an instance there using one of the images.

There is no cross-service federated communication yet, which is something we are currently working on in the [Massachusetts Open Cloud](http://info.massopencloud.org) as the [Mix & Match Federation](http://info.massopencloud.org/blog/mixmatch-federation/) project.

A [demo](https://www.youtube.com/watch?v=K6XcJpz4B0M) which includes the OpenStackClient (among federated volume attachment) change can be found [here](https://www.youtube.com/watch?v=K6XcJpz4B0M).

## How?
A new `--sp` command line option parameter is added. This is the federated service id as registered in Keystone.

{% gist aae43a75e1304170b97b shell.py %}

A new auth object is created and assigned as `self.auth`. This is done in the  `setup_auth` method, after the local `self.auth` object has been built.

{% gist 97d3a0562c410c038100 clientmanager.py %}

A WIP patch will be proposed soon with the full changes.
