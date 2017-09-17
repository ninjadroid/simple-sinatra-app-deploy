# simple-sinatra-app-deploy
App for deploying a simple Sinatra app

# Prerequisites

On the computer the playbook is run from, the following must be present:

  - Operating system must be of a Linux variety.
  - Ansible 2.3 must be installed. Further instructions on installing Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/intro_installation.html)
  - Python 2.6 or higher must be installed.

# Preparing to run the playbook

## Generating key pairs

After cloning this repository, a public and private key for the ec2-user and deployment users must be generated and copied into the ./keys/ec2-user
and ./keys/deployment paths respectively.

Keys can be generated from the command line using ssh-keygen:

e.g.
```
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/ec2-user/id_rsa
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/deployment/id_rsa
```

All keys must have permissions 0600. This can be achieved by using the chmod command:

e.g.
```
  sudo chmod 0600 ~/simple-sinatra-app-deploy/keys/ec2-user/*
  sudo chmod 0600 ~/simple-sinatra-app-deploy/keys/ec2-user/*
```

## AWS Authentication

In order to run commands from Ansible on your AWS account, your AWS Access Key and AWS Secret Access Key must be known to Ansible.
This can be done as environment variables or as part of the AWS CLI credentials.
Further details on configuring AWS for Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/guide_aws.html)

## Galaxy requirements

Before running the playbook, the prerequisite Ansible Galaxy role for rvm must be installed.

From the base path of the repo, run the command:
```
  ansible-galaxy install -r requirements.yml
```

# Running the playbook:

To run the playbook, from the base path of the repository, run the command:

```
ansible-playbook sinatra-deploy-playbook.yml
```

It may take upwards of 10 minutes to run and should not error.

# Overview of playbook process

 - Add the ec2-user key to AWS
 - Create the target host instance with the relevant ports open
 - Create a deployment user on the target host
 - Login to the target host as the deployment user and:
   - Install prerequisite packages (git, rvm, nginx)
   - Configure Nginx
   - Clone the repository in a timestamped path and link as the current deployment
   - Start the bundler for the repository and keep running in the background
 - Test the site is up from localhost and the external ip of the target host

# Additional notes:

Possible improvements could include:

Placing the instance behind a load balancer for scalability.
Removing older timestamped deploys.
Getting ip address by tag instead of ip address - this would allow for more flexible playbooks.
Split playbooks into separate create, deploy and test playbooks.
Adding a role to destroy the instance.
