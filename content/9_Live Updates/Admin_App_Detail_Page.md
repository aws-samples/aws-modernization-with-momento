---
title: "Admin App Detail Page" 
chapter: true
weight: 6
---

# Detail Page Live Changes

Now it's time for the cool part - updating the app to watch the user change their order live. As the user changes things like number of pizzas, toppings, sauce, crust, or size - workers at Momento Pizza can view them in realtime to deliver the pizza as fast as possible. Let's take a look at what we need to do.

## Code updates

First, let's find the order detail page. From the root directory, it lives in `/admin-ui/pages/orders/[id].js`. So let's open that file up and get ready to make some changes.

### Import the topic client

Just like we did on the list page, we need to import the *TopicClient* from the Momento Web SDK. To do that, add the following line underneath the existing imports in the `[id].js` file:

```javascript
import { TopicClient, Configurations, CredentialProvider } from '@gomomento/sdk-web';
```

If that looks familiar, that's because this is the same song, different verse.

### Initialize and subscribe

Now, we have the same initialization as the list page with one minor detail - we are subscribing do a *dynamic topic*. A *dynamic topic* is a variable-named topic used to represent a specific entity. Since we want updates specifically for one pizza, we need to subscribe to the update topics for it.

```javascript
const subscribeForUpdates = async (id) => {
  if (!topicClient) {
    const response = await fetch(`${process.env.NEXT_PUBLIC_ADMIN_API}/tokens`);
    const data = await response.json();
    topicClient = new TopicClient({
      configuration: Configurations.Laptop.latest(),
      credentialProvider: CredentialProvider.fromString({ authToken: data.token })
    });
  }

  await topicClient.subscribe('pizza', `${id}-updated`, {
    onItem: () => { fetchOrderDetail() },
    onError: (err) => console.error(err)
  });

  await topicClient.subscribe('pizza', `${id}-status-updated`, {
    onItem: () => { fetchOrderDetail() },
    onError: (err) => console.error(err)
  });
};
```

You see here we are subscribing to the `{id}-updated` and `{id}-status-updated` topics. Whenever data is changed on the order, our stream publishes to these topics. By subscribing, we can pick up those changes immediately and update the page in the user interface. 

When we recieve data on either topic, we call the existing `fetchOrderDetail` function which calls the API to pull the recently updated cache information and updates the screen with the latest values.

### Connect the dots

Of course, we have one final part - declaring our `topicClient` variable and calling the *subscribeForUpdates* function on page load. To do this, you can update the `useEffect` function call.

```javascript
let topicClient;

useEffect(() => {
  if (id) {
    fetchOrderDetail();
    subscribeForUpdates(id);
  }
}, [id]);
```

The `id` parameter comes from the URL path. When we land on a page, our React app will take the id out of the path and populate that variable for us automatically via the `router` object. Then it's up to us to subscribe!

## Let's try

That's all the code changes! Let's have a go on the next page and walk through the entire experience :backhand_index_pointing_right: