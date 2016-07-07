---
layout: post
title: Keystone to Keystone Proxy for Services
tags: [OpenStack]
---

This article explores a complementary and optional addition to the original idea about Resource Federation in OpenStack.

<!--more-->

# Resource Federation

[Resource Federation](http://info.massopencloud.org/blog/mixmatch-federation/) is a **Massachusetts Open Cloud** Project to enable a multi-landlord cloud model using OpenStack, where different service providers can setup competing services in a single cloud. Users can then federate the resources between the multiple service providers.

To implement this set of changes we’re using the **Keystone to Keystone (K2K)** capabilities of OpenStack, where a Keystone is able to assert the identity of one of its users to a second Keystone. We’re exploiting this by extending the capabilities of the individual OpenStack services like Nova with knowledge about K2K and the ability to query Keystone for an assertion to provide to the Service Provider so that they can get a token for it.

The above change, however, requires extending the API with information about which service provider to use and which project to scope to.

# Original Proxy Concept

Instead of letting the services know and do K2K, one idea was to abstract everything about K2K and the service provider location of the resources behind a proxy. This proxy would replace the service endpoint serving of the requested resource and would forward the API requests for it to the appropriate service provider (with the local non-federated service being a special case). This proxy service would therefore need to learn about the location of each resource. This is possible by exploiting the predictable behavior and design of REST APIs.

If a user is requesting information about the volume *volume_id* they will issue a GET request for /*project_id*/volumes/*volume_id* to the Volume API endpoint. In this case, the Volume API endpoint will be performed by the proxy. The proxy can then query the associated service providers, perform K2K to get a remote token, and query the remote Volume APIs for the specific resource. Given the uniqueness of the project ID, resource ID combination the proxy will receive many 404 NOT FOUND, but at most one 200 OK, which it will use to learn and cache the location of this resource for subsequent requests. The proxy will forward requests and responses as are, without any changes.

The proxy could also cache the tokens and the token they were mapped to after performing K2K, thus reducing the bottleneck, but the security of this needs further thought.

Additionally, it could aggregate the results from multiple providers in case the request is to list the resources.

# Revised Design

During discussions, we decided to move these features from a proxy service that runs in a separate web server into a “library” that is on the outgoing code path of requests from services.

This library would store the learnt resource mappings into an in-memory data structure. The main purpose of it would then be as a fallback in the case where our nova changes fail to populate the correct service provider id and project id fields. We hope this would lower the barrier to acceptance of our changes.

One disadvantage is that each of the separate nova processes will have a different in-memory data structure. One possible solution would be to use something like memcached.