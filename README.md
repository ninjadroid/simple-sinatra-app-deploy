# simple-sinatra-app-deploy

This is an example application for preparing an environment for and deploying the simple-sinatra-app.

The source code that is deployed is available at https://github.com/rea-cruitment/simple-sinatra-app

This application deploys a basic Ruby Sinatra Web Application using Ansible for Configuration Management and AWS for Cloud Hosting.

# Overview of Playbook Process:

 - Add the ec2-user key to AWS
 - Create the target host instance with the relevant ports open
 - Create a deployment user on the target host
 - Login to the target host as the deployment user and:
   - Install prerequisite packages (git, rvm, nginx)
   - Configure Nginx
   - Clone the repository in a timestamped path and link as the current deployment
   - Start WEBRick for the application and keep running in the background
 - Test the site is up from localhost and the external ip of the target host

# Prerequisites:

On the computer the playbook is run from, the following must be present:

  - Your local operating system must be of the Linux variety.
  - An Amazon Web Services Account. A free tier account can be created [here.](https://aws.amazon.com/free/)
  - Ansible 2.3 must be installed. Further instructions on installing Ansible can be found on the [Ansible website](http://docs.ansible.com/ansible/latest/intro_installation.html)
  - Python 2.6 or higher must be installed.

# Preparing to Run the Playbook:

## Generating Key Pairs

After cloning this repository, a public and private key for the `ec2-user` and `deployment` users must be generated and copied into the `./keys/ec2-user`
and `./keys/deployment` paths respectively.

Keys can be generated from the command line using `ssh-keygen`. e.g.

```
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/ec2-user/id_rsa
  ssh-keygen -t rsa -b 4096 -f ~/simple-sinatra-app-deploy/keys/deployment/id_rsa
```

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
