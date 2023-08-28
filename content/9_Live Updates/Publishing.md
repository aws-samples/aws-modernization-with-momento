---
title: "Publishing" 
chapter: true
weight: 1
---

# Code changes for publishing

Whenever a change is made to an order we're watching, we want to notify people watching it. In our back-end solution we have one place where all data changes go: *the stream-handler*.

We will be making changes to our DynamoDB Stream handler to publish to a topic to indicate certain events have occurred. These new events are:

* *new-order*
* *canceled-order*
* *order-status-updated*
* *order-updated*

Let's open up the `stream-handler` Lambda function code and get to work!

## Function updates

### Imports

First, let's import the `TopicClient` from the Momento SDK.

```javascript
const { unmarshall } = require('@aws-sdk/util-dynamodb');
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');
const { CacheClient, CredentialProvider, Configurations, CacheGet, TopicClient } = require('@gomomento/sdk');
const secrets = new SecretsManagerClient();
let cacheClient;
let topicClient;
```

Here we've added a single item to our existing import statement and also globally declared the `topicClient` variable. We define the topic client globally to adhere to [AWS best practices](https://docs.aws.amazon.com/lambda/latest/operatorguide/global-scope.html). 

When defining clients outside of the main handler, you allow them to be reused across function invocations. This means that the next time the `stream-handler` function is called (and it's not a cold start) we don't have to create the clients again, they are already initialized! This results in faster performance of our function and fewer SDK calls to AWS services - both of which lower the overall operating cost of the function.

### Client Initialization

Next we need to initialize the client similarly to the CacheClient.

```javascript
const initializeMomento = async () => {
  if (cacheClient && topicClient) {
    return;
  }

  const secretResponse = await secrets.send(new GetSecretValueCommand({ SecretId: process.env.SECRET_ID }));
  const secret = JSON.parse(secretResponse.SecretString);
  cacheClient = new CacheClient({
    configuration: Configurations.InRegion.Default.latest(),
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento }),
    defaultTtlSeconds: 60
  });

  topicClient = new TopicClient({
    configuration: Configurations.InRegion.Default.latest(),
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento })
  });
};
```

We updated the short-circuit condition to check for existence of the `topicClient`. If one or both of the clients do not exist, we go ahead and initialize both of them. You can see the `TopicClient` constructor is similar to the `CacheClient`, minus the *defaultTtlSeconds* argument. Since Momento Topics is a pub/sub messaging service, there is no time to live to consider.

### Handling new or updated items

We now need to make a distinction between a new item and an update to an existing one because we need to know which event (aka topic) to fire. To do this, we need to update the method signature of the `handleNewOrUpdatedCacheItem` function.

```javascript
const handleNewOrUpdatedCacheItem = async (record, oldRecord) => {
  await updateOrderRecord(record);
  if (!oldRecord) {
    await topicClient.publish('pizza', 'new-order', JSON.stringify({ id: record.pk }));
  } else {
    const topicName = (record.status == oldRecord.status) ? `${record.pk}-updated` : `${record.pk}-status-updated`;
    await topicClient.publish('pizza', topicName, JSON.stringify({ status: record.status }));
  }
};
```

You see the function now accepts an *oldRecord* argument. This is the DynamoDB record prior to the update. If *oldRecord* is undefined, that means this record was just created.

If the record was just created, we use the `topicClient` to publish to the *new-order* topic and pass along the id of the order. This takes care of our *new-order* event :check_mark_button:

If this is an update to an existing record, we need to check what changed. If the status changed, we need to publish to the `{orderId}-status-updated` topic. If it was anything else, publish to the `{orderId}-updated` topic. We want the distinction because users of the *Order app* will be notified of a status change, but not if something in the order itself changes. However, users of the *Admin app* will be notified of all changes.

### Handling deleted items

When a user deletes their order, we need to send a notification immediately so the pizzas don't get made in error. Let's update the `handleDeletedCacheItem` function to publish to a topic.

```javascript
const handleDeletedCacheItem = async (record) => {
  await Promise.allSettled([
    await cacheClient.delete('pizza', record.pk),
    await cacheClient.delete('pizza', `ADMIN-${record.pk}`),
    await deleteArrayCacheItem('allOrders', record.pk),
    await deleteArrayCacheItem(record.creator, record.pk),
    await topicClient.publish('pizza', 'order-canceled', JSON.stringify({ id: record.pk }))
  ]);
};
```

We've only made one change here, and that's on line 7 where we add `topicClient.publish`. Simple as that!

## All together

If you want to see the entire `stream-handler` function with all the updates, see below.

```javascript
const { unmarshall } = require('@aws-sdk/util-dynamodb');
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');
const { CacheClient, CredentialProvider, Configurations, CacheGet, TopicClient } = require('@gomomento/sdk');
const secrets = new SecretsManagerClient();
let cacheClient;
let topicClient;

exports.handler = async (event) => {
  try {
    await initializeMomento();
    const record = event.Records[0];
    switch (record.eventName) {
      case 'INSERT':
      case 'MODIFY':
        await handleNewOrUpdatedCacheItem(unmarshall(record.dynamodb.NewImage), record.dynamodb.OldImage ? unmarshall(record.dynamodb.OldImage) : undefined);
        break;
      case 'REMOVE':
        await handleDeletedCacheItem(unmarshall(record.dynamodb.OldImage));
        break;
    }
  } catch (err) {
    console.error(err);
  }
};

const handleNewOrUpdatedCacheItem = async (record, oldRecord) => {
  await updateOrderRecord(record);
  if (!oldRecord) {
    await topicClient.publish('pizza', 'new-order', JSON.stringify({ id: record.pk }));
  } else {
    const topicName = (record.status == oldRecord.status) ? `${record.pk}-updated` : `${record.pk}-status-updated`;
    await topicClient.publish('pizza', topicName, JSON.stringify({ status: record.status }));
  }
};

const updateArrayCacheItem = async (key, item) => {
  let cachedArray;
  const arrayResponse = await cacheClient.get('pizza', key);
  if (arrayResponse instanceof CacheGet.Hit) {
    cachedArray = JSON.parse(arrayResponse.valueString());
    const index = cachedArray.findIndex(i => i.id == item.id);
    if (index < -1) {
      cachedArray[index] = { ...cachedArray[index], ...item }
    } else {
      cachedArray.unshift(item);
    }
  } else {
    cachedArray = [item];
  }

  await cacheClient.set('pizza', key, JSON.stringify(cachedArray));
};

const handleDeletedCacheItem = async (record) => {
  await Promise.allSettled([
    await cacheClient.delete('pizza', record.pk),
    await cacheClient.delete('pizza', `ADMIN-${record.pk}`),
    await deleteArrayCacheItem('allOrders', record.pk),
    await deleteArrayCacheItem(record.creator, record.pk),
    await topicClient.publish('pizza', 'order-canceled', JSON.stringify({ id: record.pk }))
  ]);
};

const deleteArrayCacheItem = async (key, itemId) => {
  const arrayResponse = await cacheClient.get('pizza', key);
  if (arrayResponse instanceof CacheGet.Hit) {
    const cachedArray = JSON.parse(arrayResponse.valueString());
    const index = cachedArray.findIndex(i => i.id == itemId);
    if (index > -1) {
      cachedArray.splice(index, 1);
    }

    await cacheClient.set('pizza', key, JSON.stringify(cachedArray));
  }
};

const initializeMomento = async () => {
  if (cacheClient && topicClient) {
    return;
  }

  const secretResponse = await secrets.send(new GetSecretValueCommand({ SecretId: process.env.SECRET_ID }));
  const secret = JSON.parse(secretResponse.SecretString);
  cacheClient = new CacheClient({
    configuration: Configurations.InRegion.Default.latest(),
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento }),
    defaultTtlSeconds: 60
  });

  topicClient = new TopicClient({
    configuration: Configurations.InRegion.Default.latest(),
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento })
  });
};

async function updateOrderRecord(record) {
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

___

Next, we're going to be covering the token vending machine (TVM) :backhand_index_pointing_right: