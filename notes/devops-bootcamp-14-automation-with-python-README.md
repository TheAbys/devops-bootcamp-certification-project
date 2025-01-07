[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 14 - Automation with Python

## 1 - Introduction to Boto Library (AWS SDK for Python)

Automate repetitive tasks ike backups or cleanups, health checks or configuration of existing servers.
Boto is AWS SDK for Python, there are own libraries for Google Cloud or Azure Cloud.

## 2 - Install Boto3 and connect to AWS

Installing Boto3

```
pip install boto3
```

Configure AWS credentials

```
aws configure
```

Boto will take the ~/.aws/credentials for authenticating with AWS.

## 3 - Getting familiar with Boto

https://boto3.amazonaws.com/v1/documentation/api/latest/index.html
https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html

Simple example to create a VPC through Boto

see create-vpc.py

## 4 - Terraform vs Python - understand when to use which tool

Terraform keeps a state of the infrastructure and it's idempotent (no surprises, can be executed 100 times and output is always the same)
Python Boto instead (without any conditional checks) can only be executed once, otherwise multiple resource are created

Terraform could easily delete/destroy the create infrastructure, in Python i would explizitly delete one by one

Terraform is more highlevel and therefore easier to use in that case.
But with Boto way more complexe logic can be achieved.

## 5 - Health Check EC2 Status Checks

It's a basic requirement to read the documentation of the API and understand python.
An important part also is the return structure of an API call, so that the JSON can be navigated properly.

see ec2-status-checks.py

## 6 - Write a Scheduled Task in Python

Execute part of logic on a timer, every day or minute or whatever.
https://schedule.readthedocs.io/en/stable/

see ec2-status-checks.py

## 7 - Configure Server: Add Environment Tags to EC2 Instances

Automatically adding tags to ec2 instances through Boto3
see add-env-tags.py

## 8 - EKS cluster information

Checking EKS with Boto3
see eks-status-checks.py

## 9 - Backup EC2 Volumes: Automate creating Snapshots

Backing up EC2 volumes with Boto3
see volume-backups.py

## 10 - Automate cleanup of old Snapshots

Cleanup of old snapshots
see cleanup-snapshots.py

## 11 - Automate restoring EC2 Volume from Backup

Restoring of old snapshots and reattaching to EC2 instance
see restore-volume.py

## 12 - Handling Errors

Example:
- Creating of a snapshot fails, the snapshot should be deleted and not kept
- Update partly failed, rollback necessary

Terraform would handle that automatically, but in Python this needs to be done manually.
Use try/except statement for this.
https://docs.python.org/3/tutorial/errors.html

## 13 - Website Monitoring 1: Scheduled Task to Monitor Application Health

Create a linode server, add public ssh key

ssh into the server, install docker and start nginx container

see monitor-website.py

## 14 - Website Monitoring 2: Automated Email Notification

see monitor-website.py

## 15 - Website Monitoring 3: Restart Application and Reboot Server

see monitor-website.py