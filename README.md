# simple-sinatra-app-deploy

This is an example deployment application for preparing an environment for and deploying the [simple-sinatra-app](https://github.com/rea-cruitment/simple-sinatra-app).

This application deploys a basic Ruby Sinatra Web Application using Ansible for Configuration Management and AWS for Cloud Hosting.

# Overview of Playbook Process:

 - Add the ec2-user key to AWS
 - Create the target host instance with the relevant ports open
 - Create a deployment user on the target host
 - Login to the target host as the deployment user and:
   - Install prerequisite packages (git, ruby rvm, nginx)
   - Configure Nginx
   - Clone the repository in a timestamped path and link as the current deployment
   - Start WEBRick for the application and keep running in the background
 - Test the site is up from localhost and the external ip of the target host

# Prerequisites:

On the computer the playbook is run from, the following must be present:

  - Your local operating system must be of the Linux variety, e.g. MacOS, Ubuntu, Centos, etc.
  - An Amazon Web Services Account is required. A free tier account can be created [here.](https://aws.amazon.com/free/)
  - The application uses the latest public Amazon Linux image (ami-30041c53). This can be changed in the varaiable `image_id` in the file `./roles/create/vars/main.yml`
  - Ansible 2.3 must be installed. Further instructions on installing Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/intro_installation.html)
  - Python 2.6 or higher must be installed.

# Assumptions and Design Choice:

It is assumed that this application is for an audience of only a few people for testing the competency of the candidate in deploying a Sinatra Application.
For a larger audience, more focus on designing for scalability would have been involved.

There is an assumption that the tester has a basic proficiency of the Linux Bash terminal and how to run commands.

In terms of security, the following considerations have been made:
  - Updating all installed applications to allow for the latest security patches to be applied.
  - Only ports 80 and 22 are open via AWS security groups for the web application and SSH access for Ansible respectively.
  - A deployment user has been implemented for deployments, running Nginx and SSH connection via Ansible.
  - WEBrick is proxied to port 80 via Nginx to allow for a more secure and robust web server implementation.

Using timestamped deploys has been implemented to allow for rollback and traceability of deployments.
This deployment approach is not fully idempotent but has been done for the aforementioned benefits.

Cloning the application repository is done using https instead of SSH as it is assumed the deployment user's key is not from a user on Github.

Testing of the website is done via a simple curl-like request. On a production system, this would be done using an application such as Selenium or PhantomJS for greater coverage.

For further scalability and cost considerations, AWS Lambda could be used but would require completely re-architecting both the deployment and application solutions.

Using Ansible Galaxy for installing the RVM has been chosen to simplify the Ruby installation process. Ruby version 2.3.1 has been used to meet the minimum version requirement for Sinatra of version 2.2.0.

The stage role could be more fine-grained and split into separate roles for packages and Nginx but as this is a once-use deployment solution without any reuse, there is little need. Likewise, the create role could be split further but is unnecessary with the current requirements.

A different release branch can be specified as an Ansible command-line extra parameter `cli_release_branch`. e.g. `ansible-playbook ... -e cli_release_branch=test` This defaults to the master branch if not otherwise specified.

# Preparing to Run the Playbook:

## Generating Key Pairs

After cloning this repository, a public and private key for the `ec2-user` and `deployment` users must be generated and copied into the `./keys/ec2-user`
and `./keys/deployment` paths respectively.

Keys can be generated from the command line using `ssh-keygen`. e.g.

```
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/ec2-user/id_rsa
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/deployment/id_rsa
```

Note: Do not enter a password for these keys. Otherwise you will need to use it every time you deploy.

All keys must have permissions 0600. This can be achieved by using the `chmod` command. e.g.

```
  sudo chmod 0600 ~/simple-sinatra-app-deploy/keys/ec2-user/*
  sudo chmod 0600 ~/simple-sinatra-app-deploy/keys/deployment/*
```

## AWS Authentication

In order to run commands from Ansible on your AWS account, your **AWS Access Key** and **AWS Secret Access Key** must be known to Ansible.
This can be done as local environment variables or as part of the AWS CLI credentials.
Further details on configuring AWS for Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/guide_aws.html)

## Galaxy Requirements

Before running the playbook, the prerequisite Ansible Galaxy role for rvm must be installed.

From the base path of the repo, run the command:
```
  ansible-galaxy install -r requirements.yml
```

# Running the Playbook:

To run the playbook, from the base path of the repository, run the command:

```
  ansible-playbook sinatra-deploy-playbook.yml
```

It may take upwards of 10 minutes to run the playbook and it should not error.

The playbook is idempotent and can be run multiple times without affecting state (except for the timestamped deployment paths as mentioned in the Design Choices section).

# Expected Output:

Upon successful completion of the playbook, you should see results similar to the following:

```
TASK [test : assert that output is correct from external ip and localhost] *******************************************************************************************************************************************************************
ok: [54.153.210.34] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [test : output message of web server ip address] ****************************************************************************************************************************************************************************************
ok: [54.153.210.34] => {
    "msg": "The web server simple-sinatra-app is now running on ip address 54.153.210.34"
}

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************
127.0.0.1                  : ok=18   changed=1    unreachable=0    failed=0
54.153.210.34              : ok=38   changed=7    unreachable=0    failed=0
```

You should also be able to browse to the ip address specified and see the resultant 'Hello World!' message.

Additionally, you will have a `t2.micro` AWS instance running in your AWS console.
You should terminate this instance to prevent unnecessary charges once testing is completed.
