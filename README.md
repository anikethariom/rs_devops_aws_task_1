Step 1: Install AWS CLI and Terraform 

Install AWS CLI:

AWS Command Line Interface (CLI) helps you manage your AWS services using terminal commands.

Follow AWS CLI installation instructions to install version 2.

For Linux/Mac:

Use the package manager or download the .zip file and run the following commands:

curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /

Check Installation: Run this to verify the installation:

aws --version

Install Terraform:

Terraform helps in automating infrastructure using code.

To install version 1.6+, follow Terraform installation instructions.

For Linux/Mac:

Run: 

sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform

Verify Installation:

terraform --version

Step 2: Create IAM User and Configure MFA

Go to the AWS Management Console, then navigate to IAM (Identity and Access Management).

Create a New User with the following policies:

AmazonEC2FullAccess

AmazonRoute53FullAccess

AmazonS3FullAccess

IAMFullAccess

AmazonVPCFullAccess

AmazonSQSFullAccess

AmazonEventBridgeFullAccess

Enable Multi-Factor Authentication (MFA) for added security.

For the user and root account, go to IAM > Security credentials > Activate MFA and follow the steps to link an MFA device.

Generate Access Key:

While creating the user, generate an Access Key ID and Secret Access Key. These will be needed to configure AWS CLI.

Step 3: Configure AWS CLI

Open a terminal and run:

aws configure

Enter the Access Key ID, Secret Access Key, your preferred region (e.g., us-east-1), and output format (JSON).

Verify setup:

aws ec2 describe-instance-types --instance-types t4g.nano

This command fetches details about a specific instance type (here, t4g.nano) to ensure the CLI is configured properly.

Step 4: Create GitHub Repository for Terraform Code

Go to your GitHub account, create a new repository named rsschool-devops-course-tasks.

Clone the repository to your local machine:

https://github.com/anikethariom/rs_devops_aws_task_1.git

Step 5: Create an S3 Bucket for Terraform States

S3 stores Terraform states which keep track of infrastructure deployment.

Create an S3 bucket with versioning enabled:

aws s3api create-bucket --bucket my-terraform-states --region us-east-1
aws s3api put-bucket-versioning --bucket my-terraform-states --versioning-configuration Status=Enabled

Configure Terraform to use this bucket for state management:hcl code

terraform {
  backend "s3" {
    bucket = "my-terraform-states"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}

Step 6: Create an IAM Role for GitHub Actions

In AWS IAM, create a new role GithubActionsRole and attach the same policies as in Step 2.

Trust policies allow specific identities (like GitHub) to assume the role. Add GitHub’s OpenID Connect (OIDC) provider:

Go to IAM > Identity Providers and add the GitHub OIDC provider. Use:

URL: https://token.actions.githubusercontent.com

Audience: sts.amazonaws.com

Attach a trust policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:sub": "repo:USERNAME/REPO_NAME:ref:refs/heads/main"
        }
      }
    }
  ]
}

Step 7: Create a GitHub Actions Workflow for Terraform Deployment

In your GitHub repository, create a .github/workflows/terraform.yml file:yaml code

name: Terraform Deployment

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  terraform-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Terraform Fmt Check
        run: terraform fmt -check

  terraform-plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Terraform Init
        run: terraform init
      - name: Terraform Plan
        run: terraform plan

  terraform-apply:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      - name: Terraform Init
        run: terraform init
      - name: Terraform Apply
        run: terraform apply -auto-approve

Add GitHub secrets for your AWS access:

Go to your GitHub repository settings > Secrets, and add AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY.
