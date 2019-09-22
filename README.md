## Deploying a containerized application on AWS Elastic Container Service using the Elastic Container Registry

### Prerequisites

First, you'll need an AWS account. You'll also need the AWS CLI installed on your computer. Once you've created your account, [follow the instructions here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) to install the AWS CLI.

You'll also need to log into your AWS account through the AWS Console (in the web browser) to get new keys to use with the AWS CLI. Before creating credentials, take note of the region you're logged in for (for example `us-west-2` or `us-east-1`), which you can see in the top right menu section of the AWS Console display To do that, once logged in, go to the IAM service (in the Services menu at the top), click on your user, then click the Credentials tab. Create new credentials there, and be sure do download the CSV file and save both keys in a password keeper. **You will not be able to see the secret key through the console after you close the window showing it!**

Once you've installed AWS CLI and created your credentials, in your command line, enter `aws configure`, and paste in the new credentials when asked. If it asks you for a default region, use the one you took note of in the prior step above.

### Setting up ECR

...
