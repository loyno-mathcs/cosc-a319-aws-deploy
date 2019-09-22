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

### Building and running locally to test the image

In most cases for this class, building the image locally will be as simple as running:

```{bash}
docker-compose build
```

And running the image is simple also:

```{bash}
docker-compose up
```

Note that prior built layers of the Docker image can be used from layer cache, and will not need to be rebuilt. Because of this, subsequent image builds will usually be faster than the initial build.

You should always build and run your image to test that the image built correctly and the application runs successfully before pushing the image to AWS ECR.

### Building the image for deploy

To build the image for deploy, follow the previously mentioned commands available from the ECR repository's image listing.

You may need to replace `:latest` in some of the commands with a specific version number tag (i.e., `1.3.2`) if you choose. Otherwise, the `:latest` label will be re-assigned to the newly built image.

## Setting up ECS and Deploying Your App

### Creating an ECS Cluster

In the AWS Console, from the ECR Repository you created, open the **Clusters** link from the left sidebar (under **Amazon ECS**) in a new tab/window.

On that page, click the **Create Cluster** button, then select **Networking Only** from the Cluster Templates, and click the **Next Step** button.

Enter a name for your cluster (for example, `cosc-a319-internet-techologies`) and check the **Create VPC** checkbox. If you would like to see monitoring for your application in the AWS Console, you can check the **Enable Container Insights** checkbox. _Note that this may result in charges to your AWS account. Therefore, the Container Insights option is NOT required._ Once the options are set, click the **Create** button.

When you click Create, AWS will take a few minutes to create the ECS Cluster and its dependencies (a suite of related AWS cloud resources referred to as a CloudFormation Stack).

Once created, click the newly enabled **View Cluster** button.

### Creating a new Task Definition for your Application

From the Cluster screen, click **Task Definitions** in the left sidebar. On the next screen, click the **Create new Task Definition** button.

Select the **Fargate** launch type, and click the **Next Step** button.

Enter a **Task Definition Name** (for example, `cosc-a319-project-web-app`) and select `ecsTaskExecutionRole` from the **Task Role** dropdown.

In the **Task Size** section, select the minimum amounts for both **Task Memory (GB)** (`0.5GB`) and **Task CPU (vCPU)** (`0.25 vCPU`).

Click the **Add container** button. In the right side flyout, enter a **Container Name** (for example, `cosc-a319-project-web-app`).

In a new tab, open the ECR Repository you created previously. In the images table, click the copy icon in front of the Image URI for your container image (or just copy the text into your clipboard). Before leaving, aslo make note of the Image Tag.

Back in your Task Definition creation tab, paste that Image URI into the **Image** input, and check that it ends with the Image Tag. If not, you can add the Image Tag at the end, using the pattern `<Image URI>:<Image Tag>`.

Under **Port mappings**, enter `80` in the **Container port** blank, and ensure that `tcp` is selected for **Protocol**.

If not expanded, expand the **Advanced container configuration** section. Under the **Environment** section, ensure that the **Essential** checkbox is checked, and add the following **Environment Variables** (using the `value` setting in the middle dropdown of each row):

* **Key:** `PORT` **Value:** `80`
* **Key:** `NODE_ENV` **Value:** `production`

Finally, click the **Add** button at the bottom of the flyout. Then, click **Create** at the bottom of the Task Definition creation screen.

AWS will take a few seconds to generate the Task Definition and any related resources needed. Once complete, click the **View task definition** button at the bottom.

### Creating a new Service in your ECS Cluster

From the task definition screen, click **Clusters** in the left sidebar, then click the cluster name to view its information. Once on the cluster screen, in the **Services** tab, click the **Create** button just above the table of services.

In the next screen, select the Launch Type **Fargate**. If not already selected, select the task definition you just created from the **Task Definition** dropdown.

Enter a **Service Name** (for example, `cosc-a319-project-web-app`), then enter the number of copies of your app you would like to run (for example, `1` or `2`) in the **Number of tasks** input.

_Note that running more tasks will use AWS credits more quickly and will result in being billed for service use more quickly. You do NOT need to run 2 copies -- that only helps if there is some concern that your service will go down, in which case having a second copy can provide some extra reliability._

Once you've entered these settings, click the **Next Step** button.

On the next screen, select the available **Subnets** (usually, there are two) in the dropdown. Ensure that **Auto-assign public IP** dropdown is set to `ENABLED`.

Ensure that **Load balancer type** is set to `None`. _In practice, for production deployment of a moderate-to-high traffic service, you would normally want some sort of load balancing. However, for our purposes, this just complicates service inspection, so we'll leave this off for the project._

Leave all other settings as they are, and click the **Next Step** button.

On the next screen, leave **Auto-scaling** set to `Do not adjust the service’s desired count` and click the **Next Step** button.

Finally, on the next screen, review the configurations and click the **Create** button.

AWS will take a few seconds to create the service and its required dependencies. Once complete, click the **View Service** button.

## Viewing the Deployed App in the Browser

In the **Service** information screen for your new service, on the **Tasks** tab, click the **Task** ID string (called a UUID) link in the first column of the Tasks table.

On this **Task** information screen, under **Network**, copy the IP address listed next to **Public IP** label. Paste this IP address into the browser, and you should see your web app, now live on the Internet!
