---
title: "Understanding the code"
chapter: true
weight: 7.2
---

# Code Breakdown

We just wrote a fairly complex Lambda function. Let's talk about what it's doing so we don't just copy/paste and rely on magic! :magic_wand:

## Event Source Mapping

Before we get into the function code, it's important to take a look at the resource that triggers it to run - the *EventSourceMapping*.

```yaml
OrderTableStream:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt OrderTable.StreamArn
    FunctionName: !GetAtt StreamHandlerFunction.Arn
    StartingPosition: TRIM_HORIZON
    BatchSize: 1
```

There are two properties here worth talking about: `StartingPosition` and `BatchSize`

* The `StartingPosition` property of *TRIM_HORIZON* tells the resource to feed the Lambda function the oldest record. We want to process events as they occur, so this functionality will give us the chronological order.

* `BatchSize` is set to 1. When order doesn't matter, you can batch record sets up and process them in parallel. But order does matter to us. We want to put the most recent value of our items in the cache - which means we don't want parallel processing. So setting the batch size to 1 will force us to process them one by one in the order they are queued.

## Lambda Function Code

### Main handler

Execution starts here. Let's take a look.

```javascript
exports.handler = async (event) => {
  try {
    await initializeMomento();
    const record = event.Records[0];
    switch (record.eventName) {
      case 'INSERT':
      case 'MODIFY':
        await handleNewOrUpdatedCacheItem(unmarshall(record.dynamodb.NewImage));
        break;
      case 'REMOVE':
        await handleDeletedCacheItem(unmarshall(record.dynamodb.OldImage));
        break;
    }
  } catch (err) {
    console.error(err);
  }
};
```

Since we've configured our batch size to 1, we can safely grab and process the first record in the `Records` array, which is what we're doing on line 4.

After we have our record to process, we check what [type of action occured](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_streams_Record.html) in DynamoDB. Based on the action that was taken, the record will have an event name of `INSERT` (for an add), `MODIFY` (for an update), or `REMOVE` (for a delete). On an add or update to the database, we want to update the cache. But when an item is deleted, we also need to delete it from the cache. 

Since we have split behavior based on the actions from the, we implement the `switch` statement to conditionally branch our logic.

### Updating a cache item

When an order record is created or updated, we run the following code:

```javascript
const handleNewOrUpdatedCacheItem = async (record) => {
  let item = {
    id: record.pk,
    createdAt: record.createdAt,
    status: record.status,
    numItems: record.numItems,
    items: record.items,
    ...record.lastUpdated && { lastUpdated: record.lastUpdated }
  };

  await Promise.allSettled([
    await cacheClient.set('pizza', item.id, JSON.stringify(item)),
    await cacheClient.set('pizza', `ADMIN-${item.id}`, JSON.stringify({ ...item, creator: record.creator }))
  ]);

  delete item.items;

  await Promise.allSettled([
    await updateArrayCacheItem('all-orders', item),
    await updateArrayCacheItem(record.creator, item)
  ]);
};
```

This code formats the item into its return value, then updates both the user and admin versions of the cached item detail. Remember, the only difference between the user and admin view is the `creator` property.

Then we update the list views. We have the admin *all-orders* cache item and the user *my-orders* cache item we need to update. The only difference in the data between these views is that they do not list the individual pizza items inside the order.

### Deleted DynamoDB Record

When an order is deleted, we need to update the cache in a few places.

```javascript
const handleDeletedCacheItem = async (record) => {
  await Promise.allSettled([
    await cacheClient.delete('pizza', record.pk),
    await cacheClient.delete('pizza', `ADMIN-${record.pk}`),
    await deleteArrayCacheItem('allOrders', record.pk),
    await deleteArrayCacheItem(record.creator, record.pk)
  ]);    
};
```

First, we delete the detail records for the user and admin. Then we update the two list cache items. You can see all these actions are in a `Promise.allSettled` command. This will execute them concurrently, saving us on execution time and data consistency.

___

Pretty cool right? We should go see how this improves our hit rate and average latencies :backhand_index_pointing_right: