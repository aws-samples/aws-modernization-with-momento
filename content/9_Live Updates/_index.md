---
title: "Live updates" 
chapter: true
weight: 9
---

# Adding Live updates

Now that our cache is fully implemented, it's time to make the *Momento Pizza* app even cooler with live updates! To do that, we'll be incorporating [Momento Topics](https://docs.momentohq.com/introduction/momento-topics) into our solution. 

To do this, we will need to make a few updates:

1. Publish to a topic when data changes for an order
2. Subscribe to a topic in the Order Web Application
3. Subscribe to a topic in the Admin Web Application
1. Create a token vending machine (TVM) to distribute short-lived tokens for use in our UI

## So, what are topics?

You might be wondering to yourself, "what are topics and how does implementing them give me live updates?"

*Great question, I'm glad you asked!*

Momento Topics is a serverless services that enables real-time communication between two components of an application. Technically speaking, *subscribing* to a topic for updates will open a long-term connection with Momento, akin to a WebSocket. 

When someone *publishes* data to the topic, Momento will push it automatically to all the subscribers, including browser-based connections. From an architecture point of view, it will look something like this :backhand_index_pointing_down:

![Architecture diagram pointing from API to Momento to UI for Topic Workflow](/images/9_live_architecture.png)

___

Ready to start building?