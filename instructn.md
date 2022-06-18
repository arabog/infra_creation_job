Exercise: Infrastructure Creation
Write a job that creates your infrastructure - an EC2 instance and the associated Security group.

Prerequisites
Key pair - You should have an AWS EC2 key pair already created in your AWS Console, and downloaded to your local mahcine. We are assuming the key pair name is udacity.pem.
A public Github repository.
An account in the CircleCI.

Step 1. Set up a CircleCI project
Go to https://app.circleci.com/ and set up a new project associated with your Github repository.

If your repo does not contain a ./circleci/config.yml file, it will prompt you to create one. Choose a "hello-world" starter config file. The CircleCI will attempt the first build process. Don't worry if it fails at this moment.

Step 2. Set up Environment variables
To use the AWS CLI in your jobs you'll need to the following environment variables to the in Circle CI > Project Settings > environment variables. The value of these variables can be fetched from the AWS IAM user.

If not already, create an AWS IAM user with programmatic access, and it will generate these credentials.

AWS_ACCESS_KEY_ID
AWS_DEFAULT_REGION
AWS_SECRET_ACCESS_KEY

Note - While saving the environment variables in the Circle CI project settings, use capital case as discussed in this thread and also mentioned here (see the DEFAULT column for the correct names).
https://circleci.com/developer/orbs/orb/circleci/aws-cli


Another useful reference: Setting an environment variable in a project. Do read about the various types of environment variables and their relative priorities.
https://circleci.com/docs/2.0/env-vars#setting-an-environment-variable-in-a-project


Step 3. Create the CloudFormation template
Create a simple template - template.yml that will create an EC2 instance and the associated security group. This should be pushed into your git repo.
# Exercise - Rollback
AWSTemplateFormatVersion: 2010-09-09
Description: ND9991 C3 L4 
Resources:
  EC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      # Change this, as applicable to you      
      KeyName: udacity
      # Change this, as applicable to you
      # You may need to find out what instance types are available in your region - use https://cloud-images.ubuntu.com/locator/ec2/
      ImageId: 'ami-09e67e426f25ce0d7' 
      InstanceType: t3.micro
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0 

https://github.com/udacity/nd9991-c3-hello-world-exercise-solution/blob/main/template.yml


Step 4. Create the CircleCI Config file
The .circleci/config.yml file will have the following sections:
version: 2.1
# Use a package of configuration called an orb.
orbs:
  # Choose either one of the orbs below
  # welcome: circleci/welcome-orb@0.4.1
  # aws-cli: circleci/aws-cli@2.0.3
# Define the jobs we want to run for this project
jobs:
  myjob1:  # Choose any name, such as `build`
      # The primary container, where your job's commands will run
      docker:
        - image: 
      steps:
        - checkout # check out the code in the project directory
        - run: echo "hello world" # run the `echo` command
# Sequential workflow
workflows:
  # Name the workflow
  myWorkflow:
    jobs:
      - myjob1
      - myjob2

Create a job in your Circle CI config file named create_infrastructure.
  create_infrastructure: 
      docker:
        - image: amazon/aws-cli
      steps:
        - checkout
        - run:
            name: Create Cloudformation Stack
            command: |
              aws cloudformation deploy \
                --template-file template.yml \
                --stack-name myStack-${CIRCLE_WORKFLOW_ID:0:5} \
                --region us-east-1