---
title: "TVM Explained" 
chapter: true
weight: 3
---

# Token Vending Machine Code Explained

As discussed earlier, the token vending machine is responsible for creating and dispersing short-lived API tokens for use in end user browser sessions. To do this, we created the `token-vending-machine` Lambda function.

## Client initialization

Similar to what we've done a couple times before, we need to initialize the Momento clients globally so they can be reused across Lambda execution contexts.

```javascript
const initializeMomento = async () => {
  if (cacheClient && authClient) {
    return;
  }

  const secretResponse = await secrets.send(new GetSecretValueCommand({ SecretId: process.env.SECRET_ID }));
  const secret = JSON.parse(secretResponse.SecretString);
  cacheClient = new CacheClient({
    configuration: Configurations.InRegion.Default.latest(),
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento }),
    defaultTtlSeconds: 3300
  });

  authClient = new AuthClient({
    credentialProvider: CredentialProvider.fromString({ authToken: secret.momento })
  });
};
```

This is exactly as we did with our stream handler, but instead of the `TopicClient` we are creating an `AuthClient` - which is responsible for creating the short-lived tokens.

The big distinction here is our default time to live on the cache client. We set the data expiration to 55 minutes compared to the 60 seconds from before.

## Main handler

The rest of the code is relatively short, but dense in what it's actually doing.

```javascript
exports.handler = async (event) => {
  try {
    await initializeMomento();

    const ipAddress = event.requestContext.identity.sourceIp;
    const cacheResponse = await cacheClient.get('pizza', `${ipAddress}-token`);
    if (cacheResponse instanceof CacheGet.Hit) {
      return {
        statusCode: 200,
        body: cacheResponse.valueString(),
        headers: { 'Access-Control-Allow-Origin': '*' }
      };
    } else {
      const tokenScope = {
        permissions: [
          {
            role: TopicRole.SubscribeOnly,
            cache: 'pizza',
            topic: AllTopics
          }
        ]
      };

      const token = await authClient.generateAuthToken(tokenScope, ExpiresIn.hours(1));
      if (token instanceof GenerateAuthToken.Success) {
        const vendedToken = JSON.stringify({
          token: token.authToken,
          exp: token.expiresAt.epoch()
        });

        await cacheClient.set('pizza', `${ipAddress}-token`, vendedToken);
        return {
          statusCode: 200,
          body: vendedToken,
          headers: { 'Access-Control-Allow-Origin': '*' }
        };
      } else {
        throw new Error('Unable to create auth token');
      }
    }
  } catch (err) {
    console.error(err);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Something went wrong' }),
      headers: { 'Access-Control-Allow-Origin': '*' }
    };
  }
};
```

Like with our *get-order* function, we want to cache results. We don't want someone to regenerate auth tokens every time they refresh a page in a browser, so tokens are cached per IP address. The first thing we check is if we have a token for that IP. If we do, we return it. If not, we need to create a new one.

When creating the token, we first need to configure the *scope* of permissions. We don't want somebody to intercept our auth token and use it for malicious purposes, like deleting all the entries out of our cache or publishing messages to topics directly. So we set the permissions to `SubscribeOnly` and limit the access to the *pizza* cache. We could scope permissions down even further if we had only status topic names like *order-updated*. But since we include order ids in our topic names, we must set the scope to *AllTopics*.

If somebody tries to read or write from the cache, publishes to any topic, or tries to subscribe to a topic outside of the *pizza* cache, they will receive an access denied error. Our data is safe!

We are also setting a one hour expiration with the `ExpiresIn.hours(1)` argument. We then cache the token for 55 minutes, so when we get close to expiration we go ahead and generate a new one for callers so it doesn't expire on them.

Token creation is a single call to the Momento SDK via the `authClient.generateAuthToken` command, then we turn around and put it in the cache. Simple enough!

___

Alright, now that we're done with the server side code, let's make some UI changes! :desktop_computer: