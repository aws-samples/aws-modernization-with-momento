---
title: "Code changes"
chapter: true
weight: 7.1
---

# Code Changes step-by-step

## 1. Add a DynamoDB Stream Resource

Navigate to the `/api` folder in the *momento-pizza-app* repo and open the `template.yaml` file. 

Uncomment the following lines in the *Properties* section of the `OrderTable` resource:

```yaml
StreamSpecification:
  StreamViewType: NEW_AND_OLD_IMAGES
```

This will enable streams on our Order table... nice!

## 2. Create the stream handler Lambda function

With the stream in place, we need a Lambda function that processes the data. Luckily, it's already there. Take a look at `index.js` in the `stream-handler` directory under `api/functions`. We will talk more about this code [on the next page](/7_add_dynamodb_streams_processor/understanding_the_code). But before that, we have some more work to do in our template file.

In our `template.yaml` file, we need to declare the function we just built. Uncomment the following lines from the *Resources* section:

```yaml
StreamHandlerFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: functions/stream-handler/
    Policies:
      - AWSLambdaBasicExecutionRole
      - Version: 2012-10-17
        Statement:
          - Effect: Allow
            Action:
              - dynamodb:DescribeStream
              - dynamodb:GetRecords
              - dynamodb:GetShardIterator
              - dynamodb:ListStreams
            Resource: !Sub ${OrderTable.Arn}/stream/*
          - Effect: Allow
            Action:
              - secretsmanager:GetSecretValue
            Resource: !Ref MomentoSecret
```

With that in place, we've successfully added the stream handler function! Now onto our last component.

### 3. Event Source Mapping

Now that we've created the stream from DynamoDB and the Lambda function to process the data, we need to connect the two together. To do this, we add an `EventSourceMapping` in our `template.yaml` file. Uncomment the code below in the *Resources* section of the template.

```yaml
OrderTableStream:
  Type: AWS::Lambda::EventSourceMapping
  Properties:
    EventSourceArn: !GetAtt OrderTable.StreamArn
    FunctionName: !GetAtt StreamHandlerFunction.Arn
    StartingPosition: TRIM_HORIZON
    BatchSize: 1
```

## Time to Deploy!

Now that we've added all the components necessary to process data changes as they happen in DynamoDB, it's time to deploy! Run the following commands in a terminal in the root `api` directory (don't forget to provide your profile as a parameter if you're not using the default for authentication).

```bash
sam build --parallel
sam deploy
```

This will build our solution and deploy it into the cloud with the saved parameters from our initial deployment. 

___

Now that we have that done and in the cloud, let's talk a bit about what we just built.