---
title: Momento account setup
chapter: true
weight: 2
---

# Momento account setup

This workshop requires you to have an active Momento account. Don't worry, it's completely free to try and we will be well within the free tier today.

## Initial Setup

1. Navigate to the [Momento Console](https://console.gomomento.com) - you might like to open this in a new tab or window.
2. Create an account using single sign-on (Google or another provider) if convenient for you, or sign up via email confirmation.

Congratulations, you now have a free Momento account!

## Create Your First Cache

We need to create a cache to hold our pizza orders. We can do this via the user interface in the Momento Console. 

1. From the landing page in the Momento Console, click on the **Create Cache** button.
  ![Create Cache button on the main dashboard](/images/1_momento_create.png)
2. On the detail page, type in `pizza` for the *cache name* and select the *region* that's closest in proximity to you.
3. Hit the **Create** button to create your first cache!
  ![Complete form for creating a new cache](/images/1_momento_detail.png)

Woohoo! Cache is ready to go! :check_mark_button:

## Generate an API token

To enable programmatic access to your new `pizza` cache, we must generate an API token. To do this, navigate to the [token page](https://console.gomomento.com/tokens) by clicking on the lock icon via the console.

1. Choose the same *region* you selected when you created the `pizza` cache and opt for a "Super User Token".
2. Select an expiration date. For our workshop, we can leave the default of 30 days.
3. Select Type of Token "Super User Token"
4. Click on the **Generate Token** button to create your API token. 
  ![Complete form for creating an API token](/images/1_momento_apikey.png)
5. Copy the value from the **Auth token** row in the table that pops up to your clipboard. We will need this value in an upcoming step. You might want to keep this Momento Console tab open in your browser in case you need to copy/paste the token again.
  ![Generated auth token](/images/1_momento_token.png)

*NOTE - it is not recommended to create tokens with no expiration date due to security concerns*

___

Next, let's [setup our AWS account and development environment](/1_scenario-and-setup/account_setup.html).