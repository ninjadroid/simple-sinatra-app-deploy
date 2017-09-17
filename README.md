# simple-sinatra-app-deploy

This is an example application for preparing an environment for and deploying the simple-sinatra-app.

The source code that is deployed is available at https://github.com/rea-cruitment/simple-sinatra-app

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

  - Your local operating system must be of the Linux variety.
  - An Amazon Web Services Account. A free tier account can be created [here.](https://aws.amazon.com/free/)
  - The application uses the latest Amazon Linux image (ami-30041c53). This can be changed in the varaiable `image_id` in the file `./roles/create/vars/main.yml`
  - Ansible 2.3 must be installed. Further instructions on installing Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/intro_installation.html)
  - Python 2.6 or higher must be installed.

# Assumptions and Design Choice:

It is assumed that this application is for an audience of only a few people for testing the competency of the candidate in deploying a Sinatra Application.
For a larger audience, more focus on designing for scalability would have been involved.

There is an assumption that the tester has a basic proficiency of the Linux Bash terminal and how to run commands.

In terms of security, the following considerations have been made:
  - Updating all installed applications to allow for the latest security patches to be applied.
  - Only ports 80 and 22 are open via AWS security groups for the web application and SSH access for Ansible respectively.
  - A deployment user has been implemented for deployments, running Nginx and connection via Ansible.
  - WEBRick is proxied to port 80 via Nginx to allow for a more secure and robust web server implementation.

Using timestamped deploys has been implemented to allow for rollback and traceability of deployments.
This deployment approach is not fully idempotent but has been done for the aforementioned benefits.

Cloning the application repository is done using https instead of ssh as it is assumed the deployment user's key is not from a user on Github.

Getting the ip address of the target host and using it through the playbook has been done as a matter of simplicity.
This could be modified to get the ip address based on the instances 'Project' tag for greater flexibility in calling roles via the playbook.

Testing of the website is done via a simple curl-type request. On a production system, this would be done using an application such as Selenium or PhantomJS for greater coverage.

For further scalability and cost considerations, AWS Lambda could be used but would require completely re-architecting both the deployment and application solutions.

Using Ansible Galaxy for installing the RVM has been used to simplify the Ruby installation process.

The stage role could be more fine-grained and split into separate roles for packages and nginx but as this is a once-use deployment solution without any reuse, there is little need. Likewise, the create role could be split further but is unnecessary with the current requirements.

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

All keys must have permissions 0600. This can be achieved by using the chmod command. e.g.

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

It may take upwards of 10 minutes to run and should not error.

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
    "msg": "The web server simple-sinatra-app is now running on ip address 54.153.210.34 "
}

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************
127.0.0.1                  : ok=18   changed=1    unreachable=0    failed=0
54.153.210.34              : ok=37   changed=7    unreachable=0    failed=0
```

You should also be able to browse to the ip address specified and see the resultant 'Hello World!' message.
Additionally, you will have a `t2.micro` AWS instance running in your AWS console.
You should terminate this instance to prevent unnecessary charges once testing is completed.

# Additional Notes:

## Possible Improvements:

 - Placing the instance behind a load balancer for scalability.
 - Removing older timestamped deploys.
 - Getting ip address by tag instead of ip address - this would allow for more flexible playbooks.
 - Split playbooks into separate create, deploy and test playbooks.
 - Adding a role to destroy the instance.
