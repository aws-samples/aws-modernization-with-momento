---
title: "Final performance results"
chapter: true
weight: 8
---

# Final performance results

To find out whether our DynamoDB Streams processor and write-aside has helped the performance of our get-order API, let's run the load generator again. You can refer to the instructions back [in this section](3_start-load-generator.html) to start the load generator. Follow along on the **MomentoPizza** dashboard ([jump directly to it here](https://console.aws.amazon.com/cloudwatch/home#dashboards/dashboard/MomentoPizza)).

When the load generator has completed its run, zoom in on the time segment for this third test, and take a look at the results. Has the hit rate for get-order improved? Did this also improve the average and p99 latency for that API to below that of the simple DynamoDB latency results?

Write-aside enabled by DynamoDB Streams is a powerful pattern for achieving excellent hit rates, minimizing inconsistency, and reducing both average and p99 latency. The combination of write-aside and read-aside keeps just the latest and most active data in memory, to give very predictable consistency and latencies that are great for user experience.

![Dashboard with cached results including Streams-based write-aside](/images/8_write_aside_cache_results.png)
___

Our caching implementation is a great success! What else can Momento do to help you with a pleasing user experience?