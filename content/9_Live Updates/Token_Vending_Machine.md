---
title: "Token Vending Machine" 
chapter: true
weight: 2
---

# Code changes for the TVM

A token vending machine (TVM) is a piece of server-side code responsible for creating short-lived, limited-scope auth tokens. These tokens are intended to be used in the front-end of an application. Since the tokens will be sent to an end user's browsing session, there is potential for the token to be intercepted and the session could be compromised. We set the tokens to expire automatically after a short time, limiting the exposure and improving our security.

## Creating the Lambda function

Once again, we need to create a new Lambda function. This process will be identical to how we created the stream handler function.

1. Create a new folder in the `/api/functions` directory called `token-vending-machine
```bash
  mkdir functions/token-vending-machine
  ```
2. Create a new file called `index.js` in the *token-vending-machine* folder
```bash
  touch functions/token-vending-machine/index.js
  ```
3. In your terminal, navigate to the new folder with the following command (assuming you are in the `/api` directory already)
  ```bash
  cd functions/token-vending-machine
  ```
4. To initialize a new `package.json` file, run the following command and walk through the prompts using the default values
  ```bash
  npm init
  ```
5. Now we need to install the required packages, so run this command in the terminal:
  ```bash
  npm install @gomomento/sdk @aws-sdk/client-secrets-manager --save-dev
  ```
6. You should have a folder structure that looks like this:
  ![Folder structure for new Lambda function](/images/9_live_folders.png)
7. Copy and paste the following code into the `index.js` file:
  ```javascript
  const { AuthClient, CacheClient, Configurations, CacheGet, CredentialProvider, ExpiresIn, GenerateAuthToken, TopicRole, AllTopics } = require('@gomomento/sdk');
  const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

  const secrets = new SecretsManagerClient();
  let authClient;
  let cacheClient;

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

Like last time, we'll walk through this on the next page. But for now, let's finish up making the code changes.

## Template updates

We need make an update to the `template.yaml` file to include our new function. This time though, our Lambda function is backing two API endpoints, so our structure looks a little different than it did with the stream handler.

```yaml
  TokenVendingMachineFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: functions/token-vending-machine
      Policies:
        - AWSLambdaBasicExecutionRole
        - Version: 2012-10-17
          Statement:            
            - Effect: Allow
              Action:
                - secretsmanager:GetSecretValue
              Resource: !Ref MomentoSecret
      Events:
        AdminApi:
          Type: Api
          Properties:
            RestApiId: !Ref AdminApi
            Path: /tokens
            Method: GET
        OrderApi:
          Type: Api
          Properties:
            RestApiId: !Ref OrderApi
            Path: /tokens
            Method: GET
```

The code above should be pasted into the **Resources** section of the `template.yaml` file. 

You see here there are two events listed in this resource definition. This means that if someone hits the `GET /tokens` endpoint in either the Admin Api or the Order Api, our function will be triggered.

## Api definition updates

Our Apis are defined in [Open API Specification](https://www.openapis.org/) files. The Admin Api is located at `/api/admin-openapi.yaml` and the Order AApi is at `/api/order-openapi.yaml`. We need to make the following change to both specifications.

```yaml
  /tokens:
    get:
      summary: Gets a temporary, limited-scope token for use in a browser
      tags:
        - Auth
      responses:
        200:
          description: A token was generated and returned to the caller
          content:
            application/json:
              schema:
                type: object
                properties:
                  token:
                    type: string
                  exp: 
                    type: Number
        500:
          $ref: '#/components/responses/UnknownError'
      x-amazon-apigateway-request-validator: Validate All
      x-amazon-apigateway-integration:
        uri:
          Fn::Sub: arn:${AWS::Partition}:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${TokenVendingMachineFunction.Arn}/invocations
        httpMethod: POST
        type: aws_proxy
```

This entire code block goes inside the `paths` block in both files. 

This definition is enabling SAM to create the necessary permissions and connections for API Gateway to our Lambda function. A cool note here - if we were defining a POST endpoint with a request body, SAM would automatically generate validation models and check schemas for us at API Gateway, freeing our function code up from doing manual validations!

## Deploy

With these changes in place, it's time to deploy our changes and verify we can get a short-lived token. Run the following commands in a terminal at the `/api` directory:

```bash
sam build
sam deploy
```

To test, we need to take the base urls from the output of our deployment and run them through a curl command via the terminal (or something like [Postman](https://postman.com), if that's your preference). Remember the URLs from the deployment output? They looked a bit like this...

![Outputs from deployment](/images/2_deploy_outputs.png)

```bash
curl '<OrderApiEndpoint>/tokens'
curl '<AdminApiEndpoint>/tokens'
```

If all goes to plan, you should recieve a response like this:
![Curl command and successful response](/images/9_live_curl.png)

___

Now we're good to go! Let's take a step back for a minute and go through the function code on the next page so we know what we just built :backhand_index_pointing_right: