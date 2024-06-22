---

# AWS Deployment Automation with CodeDeploy and CodePipeline

This guide provides step-by-step instructions for setting up an automated deployment pipeline using AWS services including IAM Roles, S3, CodeBuild, CodeDeploy, EC2, and CodePipeline.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Create IAM Roles](#step-1-create-iam-roles)
- [Step 2: Create S3 Bucket](#step-2-create-s3-bucket)
- [Step 3: Create CodeBuild Project](#step-3-create-codebuild-project)
- [Step 4: Create CodeDeploy Application](#step-4-create-codedeploy-application)
- [Step 5: Create EC2 Instance](#step-5-create-ec2-instance)
- [Step 6: Create Deployment Group](#step-6-create-deployment-group)
- [Step 7: Create CodePipeline](#step-7-create-codepipeline)
- [Step 8: Verify the Deployment](#step-8-verify-the-deployment)

## Prerequisites
- AWS account with appropriate permissions.
- Basic knowledge of AWS services.
- GitHub repository containing the application code.

## Step 1: Create IAM Roles

1. **Create IAM Role for EC2**:
    - Open the IAM Console and create a new role.
    - Choose **EC2** as the service.
    - Attach the required policy (e.g., `AmazonEC2FullAccess`).
    - Name the role (e.g., `ec2codedeploy_role`).
    - Click **Create Role**.

2. **Create IAM Role for CodeDeploy**:
    - Create a new role and choose **CodeDeploy** as the service.
    - Name the role (e.g., `codedeploy_role`).
    - Click **Create Role**.

3. **Attach S3 Full Access Policy**:
    - Edit the `ec2codedeploy_role`.
    - Attach the `AmazonS3FullAccess` policy.
    - Save the changes.

## Step 2: Create S3 Bucket

1. **Create an S3 Bucket**:
    - Open the S3 Console.
    - Create a new bucket with a globally unique name (e.g., `my-deploy-bucket-unique`).

## Step 3: Create CodeBuild Project

1. **Create a CodeBuild Project**:
    - Open the CodeBuild Console.
    - Click **Create Build Project**.
    - Name the project.
    - Choose **GitHub** as the source provider and connect your GitHub account.
    - Authorize and select the repository.
    - Configure the environment:
        - Environment image: **Managed image**.
        - Operating system: **Amazon Linux 2**.
        - Runtime: **Standard**.
    - Click **Create Build Project**.
    - Update the IAM role used by CodeBuild to attach `AmazonS3FullAccess`.

## Step 4: Create CodeDeploy Application

1. **Create a CodeDeploy Application**:
    - Open the CodeDeploy Console.
    - Click **Create application**.
    - Name the application.
    - Choose **EC2/On-premises** as the compute platform.

## Step 5: Create EC2 Instance

1. **Launch EC2 Instance**:
    - Open the EC2 Console.
    - Click **Launch Instance**.
    - Name the instance and add a tag with `Key` as `Env` and `Value` as `Dev`.
    - Choose instance type: **t2.micro**.
    - Configure the security group to allow HTTP and SSH.
    - Enable **Auto-assign public IP**.
    - Add the IAM Role (`ec2codedeploy_role`).

2. **User Data Script**:
    ```bash
    #!/bin/bash
    sudo yum -y update
    sudo yum -y install ruby
    wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
    chmod +x ./install
    sudo ./install auto
    sudo service codedeploy-agent status
    ```

## Step 6: Create Deployment Group

1. **Create Deployment Group**:
    - Open the CodeDeploy Console.
    - Create a deployment group.
    - Specify the IAM role.
    - Choose **Amazon EC2 instance** and specify the tag used (`Env: Dev`).
    - Do not enable load balancer.
    - Click **Create**.

## Step 7: Create CodePipeline

1. **Create a Pipeline**:
    - Open the CodePipeline Console.
    - Click **Create Pipeline**.
    - Name the pipeline.
    - Choose **Custom location** and select the S3 bucket created earlier for the artifact store.
    - Connect GitHub in the source section and specify the repository and branch.
    - Choose **CodeBuild** as the build provider.
    - Choose **CodeDeploy** as the deploy provider.
    - Click **Create Pipeline**.

## Step 8: Verify the Deployment

1. **Verify Deployment**:
    - Ensure all pipeline stages succeed.
    - Access the deployed application via the EC2 instance public IP (http://<public-ip>:80).

2. **Test Changes**:
    - Edit the `index.html` file in your GitHub repository.
    - Commit the changes to trigger a new deployment.
    - Verify the changes in your deployed application.

---
