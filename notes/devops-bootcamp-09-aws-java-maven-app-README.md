[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 7 - Introduction to EC2 Virtual Cloud Server

We've created our own instance
- own security group which handles port forwarding etc
    - had to reconfigure it as the IP address changes regulary
- own ssh pem key to ssh into the machine
    - stored that under ~/.ssh
- added the instance to existing vpn and subnet
    ssh -i ~/.ssh/ docker-server.pem ec2-user@3.68.213.53
- company specific policy: add SERVICE tag to instance and volume

Installing docker on the ec2 instance
- update packages

    sudo yum update

- install docker

    sudo yum install docker

- start the docker service, this allows to pull, run, build, ...

    sudo service docker start

- check weather docker is running or not
    
    ps aux | grep docker

We want to add the user to the docker group

    sudo usermod -aG docker $USER

- check weather the group is see (relog is required)

    group

# 8 - Deploy to EC2 server from Jenkins Pipeline - CI/CD Part 1

Through having to take a break on this subject I had to cancel DigitalOcean and therefore re-setup jenkins as docker container (ec2) and a new docker repository (ecr)

- Installing jenkins again was a good practise, especially mounting docker again into the jenkins container
    took a while but in the end the pipelines were working again
- Introducing ECR (instead of Nexus or dockerhub) was more of a jenkins challenge
    made some changes in the **devops-bootcamp-08-jenkins** project which were necessary to authenticate with AWS and push images there
- Checked out the project https://gitlab.com/twn-devops-bootcamp/latest/09-aws/java-maven-app and moved it to my github
    It seems like this is actually doable automatically through https://github.com/new/import but I did it another way
        
        git clone https://gitlab.com/twn-devops-bootcamp/latest/09-aws/java-maven-app.git devops-bootcamp-09-aws-java-maven-app

    This short script tracks every branch locally so that it can be pushed through git pull --all later

        #!/bin/bash
        for branch in $(git branch --all | grep '^\s*remotes' | egrep --invert-match '(:?HEAD|master)$'); do
            git branch --track "${branch##*/}" "$branch"
        done

    set origin and push

        git remote remove origin
        git remote add origin git@github.com:TheAbys/devops-bootcamp-09-aws-java-maven-app.git
        git push --all


The pipeline now automatically runs the container on the second ec2 instance.

    docker run -p 3080:8080 -d 561656302811.dkr.ecr.eu-central-1.amazonaws.com/k0938261-training:latest

While in the video port 3080 should be exposed I found that I've still got an image with port 8080 exposed.
After some troubleshooting I've changed the port when running the application and everything works now.

If the port 3080 is used for the container and the host, than obviously the application must run on the defined port.

This solution is
- only really applicable for small applications.
- working great with different clouds (DigitalOcean, Linode, AWS, ...) or self hosted like within the company as it is just a basic ssh access to those servers

For more complexe scenarios a container orchestration tool like Kubernetes is required.

# 8 - Deploy to EC2 server from Jenkins Pipeline - CI/CD Part 2

At first we need to install docker-compose and make it executable

    sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

We can check the version through

    docker-compose --version

Cleanup ec2 instance

    docker ps
    docker stop <container-hash>
    docker rm <container-hash>

Disclaimer: It is very hard to follow this part as it feels like nothing matches or information is being skipped.
I've got everything to work, even if it's not working perfect

A few notes what I had to do:
- update my jenkins-library regarding deployment, had some error because of using "deploy" as function name instead of "call"
- manual login on the ec2 instance
- add ssh key for communicating with the other ec2 instance (one topic which was somehow not clarified or I didn't get it)

# 9 - Deploy to EC2 server from Jenkins Pipeline - CI/CD Part 3

Take over the version generating functionality from previous modules.
I've already done that on Part 2.

Pushing the version commit seems not to work with github.

# 11 - Introduction to AWS CLI - Part 1

Installing awscli

    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install

Configuring awscli

    aws configure

Usage

    aws <command> <subcommand> e.g. aws ec2 ...

Create a new ec2 instance with a new keypair and security group, subnet will be reused

    # set the company proxy
    export HTTP_PROXY=http://http-proxy.krones-deu.krones-group.com:3128
    export HTTPS_PROXY=http://http-proxy.krones-deu.krones-group.com:3128

    aws ec2 describe-security-groups
    
    aws ec2 describe-vpcs

    aws ec2 create-security-group --group-name k0938261_security_group --description "k0938261 security group created by awscli for training" --vpc-id vpc-071a096f

    {
        "GroupId": "sg-0b41576cd1633d2a0"
    }

    # get information about a security group
    aws ec2 describe-security-groups --group-ids sg-0b41576cd1633d2a0

    # relate security group
    aws ec2 authorize-security-group-ingress \
    --group-id sg-0b41576cd1633d2a0 \
    --protocol tcp \
    --port 22 \
    --cidr 130.41.104.45/32	

    # create ssh key
    aws ec2 create-key-pair \
    --key-name k0938261-key-pair \
    --query 'KeyMaterial' \
    --output text > k0938261-key-pair.pem

    # start a new ec2 instance from cli
    # all the information like image-id or instance-type can be looked up via the cli or directly in the aws console ui
    # in the previous steps we created a security group and key pair and reused the existing subnet
    aws ec2 run-instances \
    --image-id ami-02fe204d17e0189fb \
    --count 1 \
    --instance-type t2.micro \
    --key-name k0938261-key-pair \
    --security-group-ids sg-0b41576cd1633d2a0 \
    --subnet-id subnet-d650809b \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k0938261_training_awscli}, {Key=SERVICE,Value=training}]' 'ResourceType=volume,Tags=[{Key=Name,Value=k0938261_training_awscli},{Key=SERVICE,Value=training}]'

# 11 - Introduction to AWS CLI - Part 1

Filter and query specific values of a result

    aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"

    aws ec2 describe-instances --filters "Name=tag:SERVICE,Values=training" --query "Reservations[].Instances[].InstanceId"

Create a user group

    aws iam create-group --group-name k0938261-awscli-group

    aws iam create-user --user-name k0938261-awscli-user

Get information about group

    aws iam get-group --group-name k0938261-awscli-group

Add user to group

    aws iam add-user-to-group --user-name k0938261-awscli-user --group-name k0938261-awscli-group

Find the ARN (amazon resource name) for "AmazonEC2FullAccess"

    aws iam list-policies --query 'Policies[?PolicyName==`AmazonEC2FullAccess`].Arn' --output text

Attach policy to group

    aws iam attach-group-policy --group-name k0938261-awscli-group --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

List again

    aws iam list-attached-group-policies --group-name k0938261-awscli-group

Create a new login profile for the user

    aws iam create-login-profile --user-name k0938261-awscli-user --password MyPassword! --password-reset-required

    aws iam get-user --user-name k0938261-awscli-user
    aws iam get-group --group-name k0938261-awscli-group

Create a policy

    vim changePwdPolicy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:GetAccountPasswordPolicy",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:ChangePassword",
      "Resource": "arn:aws:iam::*:user/${aws:username}"
    }
  ]
}

    aws iam create-policy --policy-name k0938261-policy --policy-document file://changePwdPolicy.json

    aws iam attach-group-policy --group-name k0938261-awscli-group --policy-arn arn:aws:iam::561656302811:policy/k0938261-policy
    
Create access key

    aws iam create-access-key --user-name k0938261-awscli-user

{
    "AccessKey": {
        "UserName": "k0938261-awscli-user",
        "AccessKeyId": "AKIAYFRKTBTNQHL4Y3VU",
        "Status": "Active",
        "SecretAccessKey": "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",
        "CreateDate": "2024-02-01T15:19:48+00:00"
    }
}
