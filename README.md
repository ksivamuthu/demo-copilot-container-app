# AWS Copilot GitHub Actions

Status: In Progress

In this blog, I’m going to walk through how to set up AWS Copilot in GitHub actions. 

## AWS Copilot

AWS Copilot is an open-source command-line interface that makes it easy for developers to **build**, **release**, and **operate** production-ready containerized applications on AWS App Runner, Amazon ECS, and AWS Fargate. 

### Initialize the Copilot Application

Using copilot, you can initialize the entire infrastructure to run your containerized apps. Let’s take a look at an example. In this repository, we’ve demo **nginx** container that serves the index.html file. You can check out this repo here.

- Initialize the application using the `copilot` command-line tool.
    
    ```bash
     copilot init --app demo \
    		--name app \
    	  --type 'Load Balanced Web Service'\
    	  --dockerfile './Dockerfile
    ```
    
- Copilot will create the necessary infrastructure needed for your app in the AWS account. It will create an app service in the demo app, exposed to the internet using a load balancer. The container for running services is mentioned using dockerfile. The CLI will build the container from the docker file, set up ECR, push the image to ECR and configure the ECS service to pull the image from ECR.
    
    ```bash
    Ok great, we'll set up a Load Balanced Web Service named app in application demo listening on port 80.
    
    ✔ Created the infrastructure to manage services and jobs under application demo.
    
    ✔ The directory copilot will hold service manifests for application demo.
    
    ✔ Wrote the manifest for service app at copilot/app/manifest.yml
    Your manifest contains configurations like your container size and port (:80).
    
    ✔ Created ECR repositories for service app.
    
    All right, you're all set for local development.
    Deploy: Yes
    
    ✔ Linked account 495775103319 and region us-east-1 to application demo.
    
    ✔ Proposing infrastructure changes for the demo-test environment.
    - Creating the infrastructure for the demo-test environment.             [create complete]  [82.9s]
      - An IAM Role for AWS CloudFormation to manage resources               [create complete]  [15.9s]
      - An ECS cluster to group your services                                [create complete]  [9.4s]
      - An IAM Role to describe resources in your environment                [create complete]  [16.7s]
      - A security group to allow your containers to talk to each other      [create complete]  [5.7s]
      - An Internet Gateway to connect to the public internet                [create complete]  [21.7s]
      - Private subnet 1 for resources with no internet access               [create complete]  [5.7s]
      - Private subnet 2 for resources with no internet access               [create complete]  [5.7s]
      - Public subnet 1 for resources that can access the internet           [create complete]  [12.6s]
      - Public subnet 2 for resources that can access the internet           [create complete]  [9.3s]
      - A Virtual Private Cloud to control networking of your AWS resources  [create complete]  [18.2s]
    ✔ Created environment test in region us-east-1 under application demo.
    ```
    

### Copilot Pipeline

Copilot has a pipeline command to set up the Automated release pipeline using AWS CodePipeline for your application. You can learn more about it [here](https://aws.github.io/copilot-cli/docs/concepts/pipelines/).

## Copilot in GitHub Actions

This article is going to focus on how to deploy the application using Copilot using GitHub Actions.  Let’s walk through the setup step by step and then I wrote a GitHub action to set up for you with easy configuration.

### **Installing Copilot CLI**

The AWS copilot CLI has to be downloaded and installed to access. The AWS Copilot binaries are released on GitHub. We can download the platform-specific binaries for a specific version or the latest version from the GitHub releases pages here.

[Releases · aws/copilot-cli](https://github.com/aws/copilot-cli/releases)

### Setting up OIDC provider

OpenID Connect (OIDC) allows your GitHub Actions workflows to access resources in Amazon Web Services (AWS), without needing to store the AWS credentials as long-lived GitHub secrets.

We can configure AWS to trust GitHub's OIDC as a federated identity and includes a workflow for the `[aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)` that use tokens to authenticate to AWS and access resources.

### Copilot CLI Usage

You can use the copilot after installing the tool in your PATH. 

```bash
copilot --version 
```

You can use the deploy command using copilot cli

```bash
copilot deploy --name demo --env test
```

I’ve written a GitHub Actions wrapping these functionalities

- Installing Copilot CLI in Tool path
- Deploy the Copilot Application

[AWS Copilot - GitHub Marketplace](https://github.com/marketplace/actions/aws-copilot)

## AWS Copilot GitHub Actions Usage

To install the copilot cli in the path. You can add the `ksivamuth/aws-copilot-github-action@v0.0.1` steps with commands install to add the copilot cli in tool path. You can access the copilot cli - for e.g we are checking the version here.

```yaml
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::111111111111:role/my-github-actions-role-test
          aws-region: us-east-1
      - uses: ksivamuthu/aws-copilot-github-action@v0.0.1
        with:
          command: install
      - run: |
          copilot --version
```

To deploy the application using copilot. Pass the application name and environment.

```yaml

  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::111111111111:role/my-github-actions-role-test
          aws-region: us-east-1
      - uses: ksivamuthu/aws-copilot-github-action@v0.0.1
        with:
          command: deploy
          app: your-awesome-app
          env: prod
          force: false # optional

```

The above example is in the demo repository [here](https://github.com/ksivamuthu/demo-copilot-container-app).

When the changes are pushed into GitHub, the action build the docker image and push the image repository into ECR. Then the pushed docker image is deployed into configured environment for this ECS or AppRunner application.

![Untitled.jpg](AWS%20Copilot%20GitHub%20Actions%20d365396ce89e4d4e9de6011cc6136871/Untitled.jpg)

Once the workflow is succeed, you can see the latest version of the application deployed. 

## Conclusion

🚀 AWS Copilot supercharges your application - one cli tool to set up infrastructure, build your application with many services, setting up the pipeline to automate release, monitor the status of stack and application, and with add-ons. In this blog, we took a look how to integrate the Copilot app using GitHub Actions.

I'm Siva - working as Sr. Software Architect at Computer Enterprises Inc from Orlando. I'm an AWS Community builder, Auth0 Ambassador and I am going to write a lot about Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me **[@ksivamuthu](https://hashnode.com/@ksivamuthu)** Twitter or check out my blogs at **[blog.sivamuthukumar.com](https://blog.sivamuthukumar.com/)**!