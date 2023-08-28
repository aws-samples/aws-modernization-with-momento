---
title: AWS account setup and prerequisites
chapter: true
weight: 3
---

# AWS Account Setup 

You can deploy and build this workshop application _EITHER_ in an AWS-provided temporary account provided by your AWS team at an AWS-managed event, _OR_ use your own development account if you're following at an event hosted outside of an AWS-managed event or taking in the workshop at your own pace at home, school or work. Choose one of the following two sections below as applicable to your situation, and then continue on from "Machine Setup".

## At an AWS-hosted Live event (workshop) ?

1. Use the hashcode provided by AWS team and enter it into to your AWS account. [Event Engine - Team Dashboard](https://dashboard.eventengine.run/dashboard)
2. Pick Email One Time Password OTP, accept the terms, and login.
3. Check your email for a one time password to login. Afterward, you will see a dashboard.
4. Click on Set your Team Name in the dashboard by entering your name. This will help the workshop support team to help you troubleshoot when you need it.
5. Click on the AWS Console button. This will bring up a popup window AWS Console Login.
6. Click on Open AWS Console. This will log you into the [AWS account](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1).
7. You will use this account for the entire workshop duration.
8. Go to "Machine Setup" below.


## Self-paced or at a non-AWS event

1. [Login](https://us-east-1.console.aws.amazon.com/console/home?region=us-east-1) to your AWS account and make sure you're working in the same AWS Region as you used to create your Momento cache and auth token.
2. Go to "Machine Setup" below.

## Cloud9 Setup

<<<<<<< HEAD
Cloud9 provides a simple way to walk through this workshop with an environment that already has all the required prerequisites. From the Cloud9 console,
you can create a new environment (all the default options should be fine). 

**Log-In to your AWS account and search for "Cloud9" in Search bar.**

=======
Cloud9 provides a simple way to walk through this workshop with an environment that already has all the required prerequisites. From the Cloud9 console (you can quickly find this by typing Cloud9 into the search bar of the AWS Console landing page), you can create a new environment (all the default options should be fine).
>>>>>>> e01879345ccf562516cc61957e90e9269b340f1c

![Create Cloud9 Environment](/images/1_cloud9-create.png)

Once your new Cloud9 environment is created, you'll see it listed in the Cloud9 console. Select it, and click on "Open in Cloud9". It may take a short time to start up, but once it is running, refer to the screenshot below for next steps.

![Configure Cloud9 Auth](/images/1_cloud9_auth_config.png)

When the environment is ready, open Preferences and scroll to "AWS Settings". Disable the AWS-managed temporary credentials. Now you can close out the Preferences panel and continue.

You can also use your own local environment on your laptop or desktop if you prefer (and you don't mind installing any missing prerequisites).

### Update the permissions for Cloud9 EC2 instance

<<<<<<< HEAD
* Go to EC2 Dashboard and look for EC2 with name aws-cloud9 as prefix. Click on the Instance and go to instance summary
* In the instance summary section, find IAM role "AWSCloud9SSMAccessRole" and click on it.
* This role will be upgraded with additional permissions to run the rest of the lab .
* Add "AdministrationAccess" to this role.
  ![Configure Cloud9 Auth](/images/cloud9-permissions.png)
### Prerequisites
=======
### Prerequisites (for your local environment)
>>>>>>> e01879345ccf562516cc61957e90e9269b340f1c

Please install the following software on your machine if it is not already there (these are already included in Cloud9 environments):

* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) - *used to configure AWS credentials on your machine*
* [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) - *used to build and deploy the pizza API*
* [NVM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) - *used to install Node.js and npm*
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) - *used to clone workshop source code*

### AWS CLI authentication configuration (both Cloud9 and local environment)

You will need to setup the AWS CLI to point to your AWS account. If you have not done this before, please [follow the guide from AWS](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html). You'll need to configure credentials in Cloud9, and if your local laptop/desktop is not already configured you'll also need to follow this procedure.  IMPORTANT: be sure to configure your default region to be the same region where you created your new Momento "pizza" cache. To simplify the access requirements, it's recommended that you login using credentials for a user/role which has AdministratorAccess or PowerUserAccess policy applied. Identities in production accounts are likely to be more restrictive - it's simplest to use an AWS-provided temporary account or use a personal dev account and login as Admin. You can use the shell window at the bottom of the Cloud9 IDE screen for this, or use your own shell on your local environment.

### Source Code

The source code for the workshop is in GitHub. Please clone the [momento-pizza-app repository](https://github.com/momentohq/momento-pizza-app) to your Cloud9 or
local environment. You can use the following command in a terminal to clone to your git directory and make the cloned repo your current directory.

```bash
git clone https://github.com/momentohq/momento-pizza-app.git
cd momento-pizza-app
```

___

We did it! We're now fully setup with Momento and AWS - time to work with some code :rocket: