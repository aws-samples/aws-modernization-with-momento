---
title: "Scenario and setup"
chapter: true
weight: 1
---

# Scenario and setup

## Our pizza business is busy and expanding to more stores

In this workshop, discover how easy it is to implement caching to improve the performance of DynamoDB and enable real-time notifications using Momento Cache and
Momento Topics. The scenario for the application is a pizza shop's online order processing system. We'll build a fully serverless solution using DynamoDB, API Gateway, and
Lambda - with customer and staff interfaces implemented using Next.js. Then, we'll update our application to add caching and order update notifications for the customer.
We'll order some pizzas to test our solution and to generate some latency metrics that we'll use to measure the improvement. Finally, we will celebrate with some
delicious virtual pizza!

![Architecture Diagram](/images/mom1.png)

## Ordering a Pizza

Momento Pizza has two websites: one for customers to order pizzas and another for employees to accept and work on orders as they come in. But it needs some enhancements, what we have works, *but it could be better*!

### What We Have

This is the current process for ordering a pizza :backhand_index_pointing_down:

1. User creates an order in the Order app
2. The Momento Pizza employee refreshes the Admin app and sees a new order
3. The Momento Pizza employee opens the order and waits for the customer to submit
4. The Momento Pizza employee starts the order, cooks the pizza, and marks it as complete
5. User waits patiently at their machine, refreshing the page to watch status changes

### What We Want

That process works but it would be so much better with realtime updates and faster API responses. Let's see what we're going to build.

1. User creates an order in the Order app
2. Momento Pizza employee notices new order pops into the orders list page in Admin app automatically
3. Momento Pizza employee opens the order
3. User updates order with toppings
4. Momento Pizza employee watches as pizza is modified in realtime
5. Momento Pizza employee starts order
6. User notices they can no longer modify the order now that it is in progress
7. User can track pizza and watch in real time as the status changes

___

With this in mind, let's get setup with Momento and AWS, and make sure we've got any prerequisites installed.

[Momento Account Setup](/1_scenario-and-setup/momento_setup.html)