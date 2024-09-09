[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 15 - Configuration Management with Ansible

## 2 - Install Ansible

Ansible must only be installed on a machine and not on the managed clients.
It can be used on a local laptop or setup a server which specifically is used as an ansible control node.

Installing ansible through python, obviously needs python installed first

    python3 -m pip install --user ansible

## 3 - Setup Managed Server to Configure with Ansible

Python must be installed on the ssh target, otherwise only "raw" and "script" module can be used.

## 4 - Ansible Inventory and Ansible ad-hoc commands

Target hosts are configured in the hosts file (name can be configured).
For each host additional attributes can be configured e.g.

    <ip-address> ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root

If values are repeated over multiple hosts they can even be set once and reused.

    [droplet]
    <ip-address>
    <ip-address>

    [droplet:vars]
    ansible_ssh_private_key_file=~/.ssh/id_rsa
    ansible_user=root

With the following command all hosts are pinged. The module "ping" is executed for all hosts in inventory "hosts":

    ansible all -i hosts -m ping

Same command, but only for a group of hosts, not all:

    ansible <groupname> -i hosts -m ping

## 5 - Configure AWS EC2 server with Ansible

Hosts in inventory file must not be IP addresses, it's also possible to add the domain/hostname of a server
AWS EC2 requires the ssh private key to be set to 400 (only owner can read)

    chmod 400 <private-key-file>

The ansible python interpreter can be set in the hosts file by

    [aws_ec2:vars]
    ansible_python_interpreter=/usr/bin/python3.9

By default ansible uses /usr/bin/python on the remote system. If that path is not correct for a system it can be overwritten using this.
There are other solutions, like creating a symlink on the server before.

## 6 - Managing Host Key Checking and SSH Keys

List all known hosts for which the key was already accepted.

    cat ~/.ssh/known_hosts

For "long-time" servers, servers that are kept for a long time

    ssh-keyscan -H <ip-address> >> ~/.ssh/known_hosts

Take public key of default location ~./ssh/id_rsa.pub and copy to remote host

    ssh-copy-id root@<ip-address>

disable host key checking on various locations, for example with a ansible.cfg file in the project directory

    [defaults]
    host_key_checking = False

Configurations can be set in following files and processing order

    1. ANSIBLE_CONFIG environment variable
    2. ansible.cfg in current directory
    3. ~/.ansible.cfg in home directory
    4. /etc/ansible/ansible.cfg

## 7 - Introduction to Playbooks

Execute a playbook:

    ansible-playbook -i hosts my-playbook.yaml

Gather facts tasks will always be executed first to get info about the remote systems.
Can be disabled.

Check if nginx is really running

    ps aux | grep nginx

If possible Ansible checks wether actual state and desired state differ and only execute the change if they do.
The playbook can be executed multiple times without changes.
Not all tasks are idempotent.

## 8 - Modules & Collections in Ansible

https://docs.ansible.com/ansible/latest/collections/all_plugins.html
Find modules required for specific tasks like handling mongo db.

Collection can contain playbooks, modules, plugins, ... as a single bundle.
All modules are part of a collection.

Main hub for collections: https://galaxy.ansible.com/

List installed collections

    ansible-galaxy collection list

Example to upgrade a collection already installed

    ansible-galaxy collection install amazon.aws --upgrade

Creation of own collection with predefined project structure
https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_creating.html

## 9 - Project: Deploy Nodejs application - Part 1

Changes and comments made in deploy-node.yaml file.

## 10 - Project: Deploy Nodejs application - Part 2

Changes and comments made in deploy-node.yaml file.

shell vs cmd modules
shell is more powerful but shellinjection is possible
so if sufficient, use cmd, otherwise use shell for a more complexe situation

**valid for both**: They are not idempotent!

for any task/module the attribute register can be used to store data into a variable

this variables can then be used for debug module for example
a variable can be an object and each variable within can be access with dot notation like

    - debug: msg={{app_status.stdout_lines}}

## 11 - Project: Deploy Nodejs application - Part 3

Applications shouldn't be started as non-root. Therefore introduce a new user and execute tasks as this new user.
For this become and become_user attributes are used.

## 12 - Ansible Variables - make your Playbook customizable

Not all modules return values, but information is available on the module documentation.
One variable type is module execution results.

Example:

    src: /home/mueller/IdeaProjects/nodejs-app-1.0.0.tgz
    src: "{{node-file-location}}"

Hint: variables must be enclosed in quotes, otherwise it's viewed as a ansible dictionary

Viable alternative without quotes

    src: /home/mueller/IdeaProjects/nodejs-app-{{version}}.tgz

Variables can be passed through "vars" Attribute of a play or by commandline argument.

    vars:
        version: 1.0.0

    ansible-playbook -i hosts deploy-node.yaml -e "version=1.0.0 location=..."

Some variable names are not allowed like "linux-name" with a dash.
Instead must be like "linux_name".

Variables can also be set through a file, which makes it easier to work with
see project-vars file in this repository for an example
In that case the file must be passed to the plays by

    vars_files:
        project-vars

## 13 - Project: Deploy Nexus - Part 1

Changes and comments in deploy-nexus-yaml

## 14 - Project: Deploy Nexus - Part 2

Changes and comments in deploy-nexus-yaml

There are multiple ways to wait for a result https://docs.ansible.com/ansible/latest/collections/ansible/builtin/wait_for_module.html
Waiting for a port to be opened, or waiting for a folder to be present etc.

## 15 Ansible Configuration - Default Inventory File

In the ansible.cfg "inventory" can be configured
This way it must not be added every command

    ansible-playbook my-playbook.yaml

## 16 - Project: Run Docker applications - Part 1

Using terraform again to provision ec2 instances

Changes and comments in deploy-docker*.yaml files

## 17 - Project: Run Docker applications - Part 2

Changes and comments in deploy-docker*.yaml files

Pulling images from public docker repositories

    - name: Pull redis
      docker_image:
        name: redis
        source: pull

## 18 - Project: Terraform & Ansible

Within terraform script.
The hosts within the playbook must be set to all or aws_ec2

    resource "null_resource" "configure_server" {

        # this resource triggers when the ip address changes, meaning in that case it probably needs reconfiguration
        trigger = {
            trigger = aws_instance.myapp-server.public_ip
        }

        provisioner "local-exec" {
            working_dir = "/home/mueller"
            # the followup comma is necessary to work
            command = "ansible-playbook --inventory ${aws_instance.myapp-server.public_ip}, --private-key ${var.ssh_key_private} --user ec2-user deploy-docker-new-user.yaml"
        }
    }

## 19 - Dynamic Inventory for EC2 Servers

auto scale up or down generates or removes a lot of hosts that need configuration

hosts can be loaded by python script or via a ansible plugin
best practise is to use the plugin
https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html

See aws_inventory_ec2.yaml plugin configuration

Using command to list inventory

    ansible-inventory -i aws_inventory_ec2.yaml --list

per default it retrieves the private dns names which are only reachable from within the vpc, meaning localhost cannot connect to the instance
Within terraform VPC creation script the argument enable_dns_hostnames must be set to true, so the public dns names are generated

the inventory can be filtered, for example only selecting production servers
https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options


## 20 - Project: Deploying Application in K8s

Check if necessary python modules are installed, throws ModuleNotFoundError if not installed

    python3 -c "import YAML"
    python3 -c "import kubernetes"
    python3 -c "import jsonpatch"

Install in users home directory, not in system

    pip3 install pyyaml --user
    pip3 install kubernetes --user
    pip3 install jsonpatch --user

Set the kubeconfig to access cluster

    export KUBECONFIG=/home/mueller/kubeconfig_myapp-eks-cluster

Set environment variable for Ansible to connect to cluster

    export K8S_AUTH_KUBECONFIG=/home/mueller/kubeconfig_myapp-eks-cluster


## 21 - Project: Run Ansible from Jenkins Pipeline - Part 1

**Common practise**
don't install Ansible within Jenkins but instead create a server only responsible for executing Ansible.

ssh into the created ansible server

    sudo apt update

On ubuntu typing in ansible gives a hint for installation

    ansible

Installing ansible

    sudo apt install ansible-core

If ansible should configure ec2 instances aws must also be configured on the ansible server

    apt install python3-boto3
    # boto-core is installed as depencendy here

Inventory is again loaded by the aws_ec2 plugin.

    cd ~
    mkdir .aws
    cd .aws
    vim credentials

Paste local .aws credentials to the file and save

For newly created ec2 instances a new "ansible-server-key" can be created to use for connection from the ansible server to the ec2 instances

## 22 - Project: Run Ansible from Jenkins Pipeline - Part 2

See file Jenkinsfile for changes and comments.

## 23 - Project: Run Ansible from Jenkins Pipeline - Part 3

See file Jenkinsfile for changes and comments.

## 24 - Ansible Roles - Make your Ansible content more reusable and modular

See folder roles

The folder has to have a specific structure
The subfolders are the rolenames which can be used within other ansible scripts.