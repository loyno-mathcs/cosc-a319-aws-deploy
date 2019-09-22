# Deploying a containerized application on AWS Elastic Container Service using the Elastic Container Registry

## Prerequisites

### Setting up AWS and AWS CLI

First, you'll need an AWS account. You'll also need the **AWS CLI** installed on your computer. Once you've created your account, [follow the instructions here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) to install the AWS CLI.

You'll also need to log into your AWS account through the **AWS Console** (in the web browser) to get new keys to use with the AWS CLI. Before creating credentials, take note of the region you're logged in for (for example `us-west-2` or `us-east-1`), which you can see in the top right menu section of the AWS Console display.

To create your credentials, once logged in, go to the **IAM** service (in the **Services** menu at the top, under **Security, Identity & Compliance**), click on your user, then click the **Security credentials** tab. Create new credentials there, and be sure do download the CSV file and save both keys in a password keeper. **You will not be able to see the secret key through the console after you close the window showing it!**

Once you've installed AWS CLI and created your credentials, in your command line, enter `aws configure`, and paste in the new credentials when asked. If it asks you for a default region, use the one you took note of in the prior step above.

### Setting up Docker

You will also need Docker Community Edition installed. To download it, you'll need to create a login to [Docker Hub](https://hub.docker.com/).

Once you've logged in, you can click the **Download Docker Desktop** button in the right sidebar (or go directly to the download page for [Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows) or for [macOS](https://hub.docker.com/editions/community/docker-ce-desktop-mac)). Install the Docker Desktop application, and ensure it's running by entering `docker ps` at the command line.

## Setting up ECR

In the AWS Console, go to the **ECR** service (in the **Compute** section of the **Services** menu). Once there, click the **Create repository** button.

Enter the name of the image in the blank, leave **Mutable** selected, and click the **Create repository** button.

Once the repository is created, you should see a record for it in the table on the ECR Repositories listing. Click the repository name link to be taken to the repository image listing. There, you'll see an empty table.

On this screen, at the top-right of the table, click the **View push commands** button. Remember this location &mdash; it is where you will find commands that you can copy and paste into the terminal that will allow you to login to AWS and be able to push newly built docker images into the image repository.

## Building the image

In most cases for this class, building the image locally will be as simple as running:

```{bash}
docker-compose build
```

And running the image is simple also:

```{bash}
docker-compose up
```
