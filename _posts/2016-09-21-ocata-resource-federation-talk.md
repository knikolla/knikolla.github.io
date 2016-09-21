---
layout: post
title: OpenStack Barcelona Summit vBrownBag on Resource Federation
tags: [OpenStack]
---

We got accepted for a vBrownBag talk for the OpenStack "Ocata" Summit in Barcelona on "Resource Federation in a Multi-Landlord Cloud". 

<!--more-->

> We present a solution for a multi-landlord cloud where resources can be combined across deployments.
>
> This model allows each organization to maintain its autonomy by operating its own OpenStack deployment with federated identity. A proxy will replace the Cinder and Glance endpoints in Keystone. When the proxy receives an API call, it determines the location of any objects the API call acts on, and forwards the request to the appropriate deployment using Keystone-to-Keystone authentication. An agent will listen to the messagebus for notifications of newly created resources, learning their location. This allows Nova and Horizon to access resources such as volumes, images and snapshots in other OpenStack deployments.
>
> This solution enables the Open Cloud eXchange model that the Massachusetts Open Cloud (MOC) project is built on. The MOC is a collaboration between higher education (BU, NEU, Harvard, MIT and UMass), government, and industry.

The talk will be Wednesday, October 26, 2016 at 5.45pm. Full schedule for all vBrownBag talks is available [here](http://vbrownbag.com/2016/09/vbrownbag-techtalks-at-openstack-barcelona/). 