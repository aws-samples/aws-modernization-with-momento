---
title: "Admin App List Page" 
chapter: true
weight: 5
---

# Admin App List Page Updates

Our admin app gets a bit more of the cool realtime communication updates. We want our list page to update automatically when a new order is added or canceled. This allows the team at the store to react as quickly as possible and get delicious pizzas to customers in record time.

## Before we make changes

Just to make sure things are already in working order, let's compile and run the solution to see what it looks like. You should already be in the `/order-ui` directory, we need to run the following command in a terminal:

```bash
cd ../admin-ui
npm install 
npm run build
npm run dev
```

Much like the other web app we were just in, this will install dependencies, build the solution, then start the local host. If you did not stop the local server for the order app, this should start up another instance, but this time on port 3001 (http://localhost:3001). To easily test the updates we're about to make, having both instances running will come in handy.

After the local server boots up, you can log into the *localhost* url and you should see something like this (you should be able to see any pizza orders you've created already).

![Home page of the admin app](/images/9_live_admin_home.png)

## Code Changes

Our first change is on the `/admin-ui/app/page.js` page. We will be refreshing the order list whenever we receive a message from the *new-order* or *order-canceled* topics. First, let's import the necessary components from the Momento SDK.

```javascript
import { TopicClient, Configurations, CredentialProvider } from '@gomomento/sdk-web';
```

Add this line directly underneath the rest of the import statements in the `page.js` file.

Next, we're going to add a function similar to what we did in the Order app to initialize the *TopicClient* and subscribe it to two different topics :backhand_index_pointing_down:

```javascript
const subscribeForUpdates = async () => {
  if (!topicClient) {
    const response = await fetch(`${process.env.NEXT_PUBLIC_ADMIN_API}/tokens`);
    const data = await response.json();
    topicClient = new TopicClient({
      configuration: Configurations.Laptop.latest(),
      credentialProvider: CredentialProvider.fromString({ authToken: data.token })
    });
  }

  await topicClient.subscribe('pizza', 'new-order', {
    onItem: () => { fetchOrders() },
    onError: (err) => console.error(err)
  });

  await topicClient.subscribe('pizza', 'order-canceled', {
    onItem: () => { fetchOrders() },
    onError: (err) => console.error(err)
  });
};
```

Initializing the *TopicClient* is exactly like what we did before, except this time we're calling the **Admin API**. Note the difference in the environment variable we use to call the token vending machine.

Next, we'll subscribe to the *new-order* and *order-canceled* topics separately. When we receive data to our subscription, we will call the existing `fetchOrders` method, which hits our Admin API endpoint that loads active orders.

Lastly, we'll need to declare our global *topicClient* variable and make a call to `subscribeForUpdates` to set us up when the page loads. Update the `useEffect` call to look like this:

```javascript
let topicClient;

useEffect(() => {
  fetchOrders();
  subscribeForUpdates();
}, []);
```

We were already doing the initial `fetchOrder` call on page load, so now we added our initialization step in there as well. Now we can test it out! With both the order app and the admin app open on the home page, click the *+ New Order* button. Watch as your new order immediately appears in the admin app. Super cool :pizza:

___

Now that the list page updates live, let's work on the detail page :backhand_index_pointing_right: