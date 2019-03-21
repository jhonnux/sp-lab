# Introduction

This repository contains the Cloudformation template YAML files in order to build publicly available HA Load-Balanced. The configuration are
- Deploy a 3-tier with 2 AZ on each tier Virtual Private Network
- Deploy EC2 Security Group Inbound and Outbound Rules
- Deploy RDS Cluster using RDS Aurora
- Deploy a full Web Application within a Public Elastic Load Balance and autoscaling

NOTE: You need to have a valid AWS and willing to spend some money as some AWS resources are outside the free tier (aka RDS Aurora Cluster ans KMS)

In order to run this lab you need to create CFN stack in the following order:
1. cloudformation/vpc.yaml
2. cloudformation/vpc-securitygroups.yaml
3. cloudformation/rds-cluster.yaml
4. cloudformation/wordpress.yaml

# CloudFormation Template
Now let's explain each CFN Template file

## cloudformation/vpc.yaml

This template is reponsible of deploy a brand new VPC with the following features
* 3 Tier Network distributed as Public Tier (Public Network), Application Tier (Private Network) and Database Tier (Private Network)
* Each Tier is distributed in two availability zones for failover
* Other VPC AWS Resources such as Subnet. RouteTable, Elastic IP, NAT Gateway are also included
* Created RDS and ElastiCache SubnetGroup in a event of use RDS/ElastiCache cluster

The relationship between each template is important as we use export/import values on each template to pass parameters between them

## cloudformation/vpc-securitygroups.yaml

This template is responsible of the creation of some basic EC2 Security Groups required for the interoperation of the VPC Network created above. It takes care of the Inbound and Outbound basic rules.

By default it deploys the following SG for the following services
* Internet Access: Used to attach outbound rules for basic internet access
* Wordpress Instance: Attached to a EC2 Instance running Wordpress
* Generic Load Balancer: To be attached to a Classic and/or Application Load Balancer
* Database Instances: To be attached to a simple EC2 Instance running any SQL Services. Not in the scope for this lab
* JumpHost Access: To be attached to a JumpHost EC2 Instance located on the public subnet for remote administration. Not in the scope for this lab
* ECS Instances: To be used for a Elastic Container Cluster (either Docker or Kubernetes). Not in the scope for this lab

Some SGs can be added/edited/removed depending of what services are required

## cloudformation/rds-cluster.yaml

This CFN template deploys an Aurora RDS Cluster with some default values, however the template is capable to deploy up to two database instances, dbcluster parameter group, parameter group, subnetgroup, security group, backup retention, maintenance windows and other attributes.

In addition allows to have Encryption at rest using KMS RDS key (valid only for the current AWS account)

## cloudformation/wordpress.yaml

This template deploy EC2 Instances using the basic Amazon AMI, put their instance within an Autoscaling Group.  In addition configure Application Load Balancer, Target Group and Listeners (HTTP and HTTPS)

# Basic Goals answers

1. Using any tooling of your choice, automate the deployment of secure, publicly available HA Load-Balanced Web Servers and or Application.

ANSWER: I decide to use CloudFormation (explained above) in order to deploy relevant AWS resources required

2. Ensure that the web service is run in a manner where is there is no outage to the service in the event of a failure. 

ANSWER: The template created has the ability to have failover on the application side by using AutoScaling and Launch Configuration rules, each Wordpress instance are bootstrapped.
On the database side, the RDS Aurora cluster can deploy up to 2 instance (one on each availability zone) in order to have some fault tolerance

3. Redirect any HTTP requests to HTTPS. Self-signed certificates are AOK.

ANSWER: At this stage I leverage HTTPS on the ALB only, terminate SSL and pass to the Wordpress as HTTP

Bear in mind that a valid ACM is required to make HTTPS works. I have a personal domain (managed outside Route53) as mydomain.com.au
I created a CNAME entry from mylab.mydomain.com.au to the ALB DNS

4. Answer the question: "How would you further automate the management of the infrastructure if given unlimited time and resources?"

ANSWER: I'd deploy this solution within a CI/CD pipeline (use Jenkins, Bamboo) to further automatize.  Of course the source code must be on a GIT repository. 

In addition If the platform has configuration drift I'd use a Configuration Management tool such as Puppet and/or Ansible to keep a state of the EC2 Instances.

I'd add some units test within the code to make sure is more reliable

# Additional Challenges answers

1. Drive the deployment with automated orchestion.

ANSWER: Not considered at this but I'd consider use Ansible to drive the creation of the CFN templates

2. Provide basic automated tests to cover standup, maintanence and scale: included scripts, templates, manifests, recipes, code etc.

ANSWER: Not considered for this lab due of time

3. Redirect any 404 errors to a custom static page.

ANSWER: This is included inside wordpress.yaml file.  Check under LaunchConfiguration - Metadata - ConfigSets - configure_wordpress
I use Cfn-init to create a static 404.html page and the use "sed" command against httpd.conf file to enable it

4. Add a Database to your automation and have your application serve the data stored in addition to the instance ID.

ANSWER: This is included on rds-cluster.yaml and wordpress.yaml file

# Output

1. A public URL for hitting the web server deployment.

ANSWER: 
HTTP
  Basic Apache static page
    https//dev-wordpress-lb-951451375.ap-southeast-2.elb.amazonaws.com
Wordpress
    http://dev-wordpress-lb-951451375.ap-southeast-2.elb.amazonaws.com/wordpress

HTTPS
  Basic Apache static page
    https//http://mylab.mydomain.com.au
  Wordpress
    https://http://mylab.mydomain.com.au/wordpress

2. a set of read-only access credentials (Access Key and Secret Key) allowing us to see any recourses that you used to complete the task.

ANSWER: 
check email send to Alec on Saturday 23th Marchfor more information


3. SSH (or any authentication mechanism that would allow us Command and Control to see what is happening) key pairs for logging onto the web instances used in the deployment or console access if you go for containers/server less.

For all EC2 Instance access use the following private key

file:  dev.pem

    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAgZnEck6nGIqBvZAt7rYieUdQmMELdZ1uWkRt98mba1RzBxrXHq/1N2hOZreM
    iogowuZD2Inbyic2CM9mSJ1pr5QwX8Kj9slxsMEII+0li7G4oXLZTdJ527GAPhko7dXn0Fo+PgMX
    Bgndjvcpglke2rFABBxIGmobNycYKEFFHsPBA5M9xcoGLG/LojI5FvvyiUEk+U+HRv429ErpUFji
    5esxlMdUn62YurzBd7AHZv4TfjqVXicEhwfPvL2A1Sxg0maQRjxPSR92EYWkTmfjbCEWzy894EGm
    08uHL6AmYo9z3nEoUPUnTdI8lv/MFKcFZasq642JE+F/ieDPH8h1FwIDAQABAoIBABatHAVQI8aU
    fYz4jEDnV3LW+pAvvtyOdj+PF5qyiOInvZSxqpAxA3v1YTpxxUJ7/n3Tom1h+bYOVFMITwJHoLVa
    /XfT67KDnsPpko4OsXIW35JIMSN/v1ikXyb+af3rXotDLv7UtZOV6FFah8XJ6C8lsmjFtgwBIs/s
    pDpu6TQvlb2po3xXw92MmGdrcbCqwWRWFHlLgsq8surubVTlCRsT+XprxTjyYiTwuRJPMCEBuAch
    frmgcbNUN16HaTfMwoSWKdHjP6iBpIOvkZJOJeK+WWkEZ7mHnyMzKYUzNlwYnpr+bXMgLqnRnr/n
    8Mwl0VjEJ/uY6jXPikN0lWAsAIECgYEA0+/bMEu63LNTmhTg2Gr3mLBftHiUcI6TRSd8RAvnoNAg
    zKF0fxYU7uM3fga68IjhXVe4pFmz++8xSVioMHS8y0dqD0YmyY3xd1f+cKszlVpY2f5/H3vmmHlY
    JhmDokArKKuLmAfUjo+lfz/vgY+s4lD+z659t0XxAoop8YR1WVcCgYEAnIuj2GN19rL9B2yuphPi
    jLlCCy8WwMOMA/VdKX2hdSl6uf/oxr4L4WELjDKNq6L6D/1DAr2byqWAsTz5kdnclvbHOkBg21pO
    GQnZ9FlhzIUyY6FL10QnyHry+PRt1Vh2laOGeMmX3HRRS498U/u1rYEu/DTwE+op6AZTbyX/qkEC
    gYEA0d7ZjS/Q8TBbf19hcK4sVXWCsHIffH6Tc42wTqoDS4oOkNTpSdgSDqXOk+wSPpMtqINvgsCZ
    rMemZN14X9OaCSrE6i8rxbfb/7SRb/z47dMz3VtZg1Hsfdzb346wfYFRu8p0R66pXCr/Vc14XTJr
    nwlZ60r9jvmhukQbWOE0W60CgYEAiP1RpdxzsCy1a7fZpY+lIsxDVRIh8RGuBxDCM7qyfZqMwROG
    mdRZBSMtPcRHYTk/ZpqT92QBXvYxhef75XwmoxzU/s6zc4C08whB6KgHAzhT/gd6HKiRv8iHswAC
    1T2SmCP/WtfewpYvRdYMUFnmhCyV9zJEHMk/XsGaZZt0/YECgYBSt0LjyGuKmDlB6t+Ar9G2eJXk
    LD2L891TjfHXz6j3F8jCyWWyFfy68jVw+v79fvPnEdRridt3CCuyNV7pgS+b45rZ3s3VZhXZo4JJ
    143pv4nsMyhJb3beVT5zJBtnqbezUTSDHk+LXV8/naI/85ZchmPC5+sn8AEVJM3fW8VLSg==
    -----END RSA PRIVATE KEY-----


4. any scripts, config files, manifests, recipes, or source code you used to achieve the goal above.

ANSWER: Read this document

5. any detailed notes, written explanations, diagrams, screen shots to help demonstrate your work.

ANSWER: Read this document. 

6. written answers to the question: "How would you further automate the management of the infrastructure?"

ANSWER: Well depends of the requirement as there some avenues to automatize further.

I'd add Jenkins to do a CI/CD pipeline, trigger new build from GIT when the branch is updated or allow to other users to run job manually. Configure Job using AWS CLI and trigger the CloudFormation stack create/update.  I'd setup parameters from Jenkins or even better from a configuration file.  Jenkins job can create all 4 CFN stack in the order required.

Within Jenkins Job I could use Unit of Testing to verify CFN template has the right syntax

If Jenkins is not an option I could use CodePipeline and/or another CI/CD Tool

For orchestration and configuration management I'd prefer Ansible

In additon I'd consider to add monitoring on the infrastructure, it could be CloudWatch, ElasticSearch+Kibana+LogStash and other monitoring tools.

For security I improve encryption at rest on EC2 instances using custom AMI with EBS Encrypted.

I created a jumpbox EC2 Instance manually.  I could add another CFN to automatize this box deployment, since it was out of scope I didn't do it.
