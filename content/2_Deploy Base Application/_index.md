---
title: Deploy base application
chapter: true
weight: 2
---

![Workshop Component Architecture](/images/2_deploy_architecture.png)

## Deploying the API

This workshop consists of two web applications and a back-end API. The web applications are run locally while the API is run out of AWS. To get started, we must deploy the API using the Serverless Application Model (SAM) CLI.

1. In a terminal in the root directory of the code repository, navigate to the `api` directory by typing in the following command:
  ```bash
  cd api
  ```
2. Run the following commands:
  ```bash
  sam build --parallel
  sam deploy --guided
  ```
  **NOTE** - The above commands will deploy to the default profile configured via the AWS CLI. If you wish to deploy with a specific profile, you can use the following commands.
  ```bash
  sam build --parallel
  sam deploy --guided --profile <profilename>
  ```
3. Complete the wizard in the terminal, filling out values appropriately. Recommended values are below. If you should need to redeploy for any reason, you need not
provide the "--guided" parameter and will not need to answer the questions again.

  |Argument|Recommended Value|
  |--------|-----------------|
  |Stack Name|momento-pizza-app|
  |AWS Region| *use default*|
  |MomentoApiToken| *use token obtained from setup step*|
  |Confirm changes before deploy |N|
  |Allow SAM CLI IAM role creation|Y|
  |Disable rollback|N|
  |*All "no auth defined" prompts*|Y|
  |Save arguments to configuration file|Y|
  |SAM configuration file|samconfig.toml|
  |SAM configuration environment|default|
  
4. The stack will deploy into your AWS account. On completion, you will see two outputs provided in the terminal:
  ![Outputs from deployment](/images/2_deploy_outputs.png)
5. Copy the value from the **AdminApiEndpoint** and open the file `./admin-ui/next.config.js` (from the root directory).
6. Update the `NEXT_PUBLIC_ADMIN_API` property with the copied value.
  ```javascript  
  const nextConfig = {
    env: {
      NEXT_PUBLIC_ADMIN_API: '<copied value from output>'
    }
  }

  module.exports = nextConfig
  ```
7. Copy the deployment output value **OrderApiEndpoint** and open the file `./order-ui/next.config.js` (from the root directory).
8. Update the `NEXT_PUBLIC_ORDER_API` property with the copied value.

That's it! You're setup and we are ready to start!