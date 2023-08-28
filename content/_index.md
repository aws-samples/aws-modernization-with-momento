---
title: "AWS Modernization Workshop - Momento" # MODIFY THIS TO BE THE TITLE OF YOUR WORKSHOP
chapter: true
weight: 1
---

# AWS Modernization Workshop - Momento<!-- CHANGE THIS TO BE THE TITLE OF YOUR WORKSHOP -->

## Welcome

Caching is notorious as a particularly challenging architectural component to manage if you have
to take care of all the details like: scaling in instance-sized increments, replication, maintaining
hit rates during maintenance windows, etc. Because caches are intended to absorb traffic spikes and
smooth out load on databases, they are often over-provisioned and under-utilized. These pain points
have caused many applications that would benefit from caching to go without, driving up database
costs and delivering user experience that's not as responsive as it could be.


Now there is Momento Cache - a truly serverless caching service. It enables developers to focus on the specifics of delivering a great application without getting distracted by the
plumbing. Momento Cache pricing is based on consumption (and scales to zero), and it's as easy to deploy and manage as making a simple call to a CreateCache() API. Momento has also recently introduced Momento Topics - a very simple, high-throughput notification service (pub/sub) which scales to thousands of subscribers per topic. You can learn more about Momento's products at [gomomento.com](https://gomomento.co/46faC4h).

## Learning Objectives

In this workshop, discover how easy it is to implement a serverless cache for DynamoDB using Momento - and see measurably improved latencies in the application. Learn how to keep your cache data fresh and hit rates high as changes are made by tapping in to DynamoDB Streams. Still, your application users want more - and you'll step up to deliver real-time notifications to users With Momento Topics.


Workshop Goals:

* Learn how to implement a read-aside cache for DynamoDB<br/>
* Learn how to improve cache hit rates and maintain better consistency with automatic cache invalidation on data updates<br/>
* Learn how to broadcast notifications to end users when data changes<br/>




{{% notice warning %}}
The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how various AWS services can be architected to build a solution while demonstrating best practices along the way. These examples are not intended for use in production environments.
{{% /notice %}}