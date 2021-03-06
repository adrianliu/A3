# Assignment 3- DevOps

For your third assignment, you will be deploying your A2 application and setting up an environment that allows for you to continously update said application.
The section [here](#example-walkthrough) showcases an extensive walkthrough with the dummy app contained in this repository: leveraging Flask, GUnicorn, Nginx, Vagrant, Ansible, and Terraform.

## Learning Objectives
This assignment is trying to allow for you to explore the word of DevOPs by getting you comfortable working with the following tools / technologies / concepts:
* `gunicorn` as a way to serve your application through `WSGI`
* `nginx` as an HTTP server to handle and route requests
* `vagrant` as a virtual machine to simule our AWs AMI
* `ansible` as a tool for automated deployment
* `terraform` as a service to launch AWS instances and other services

We have provided for you an step-by-step example walkthrough on the dummy `Flask` app contained in this repo. This example walkthrough will explain each of the technologies and how each of them should be structured to build a fully automated deployment application.

The example walkthrough is optional, but highly recommended, however you are required to go through the assignment walkthrough which will simulate and deploy your A2. More details below.

## Table of Contents

* [Academic Integrity](#academic-integrity)
* [Example Walkthrough](#example-walkthrough)
* [Assignment Walkthrough](#assignment-walkthrough)
* [Expected Functionality](#expected-functionality)
* [Extending the Assignment](#extending-the-assignment)
* [Project Submission](#project-submission)

## Academic Integrity and Collaboration

### Academic Integrity

Note that these projects should be completed **individually**.  As a result, all University-standard AI guidelines should be followed.

### Code Attribution and Collaboration

## Example Walkthrough
For this example we will be deploying our the example app in this repo leveraging Vagrant, Ansible, and AWS. The walkthrough is as follows:
### Vagrant Setup
For this assignment: instead of using [virtualenv](https://virtualenv.pypa.io/en/stable/) we will be using [Vagrant](https://www.vagrantup.com/). Why?

There’s two main challenges that you’ll encounter in DevOps:

* We’ll need to replicate what it’s like to set up a new server. If you’re using your local machine, it’s tough to get it to act like a fresh machine.
* It’s possible that your local machine is a different type of box than your server. Perhaps you’re working on a Mac but deploying to a Linux box, for example. Maybe you’re deploying to a different version of the operating system or a different operating system all together.

We’ll use Vagrant for our development environment because it allows us to address these challenges. As such, we will set up a virtual machine and run it there but still write code locally.

#### Note
If you want to mimic our development environment, you can run `virtualenv` to have all the same packages without installing `Vagrant`, `Ansible`, and `AWS CLI` on your home-machine as depicted in this guide. This can be done with the following commands:
```bash
$ pwd
<CURR_DIRECTORY>/A3
$ virtualenv venv
$ source venv/bin/activate
$ pip install -r venv-requirements.txt
```

To get started we must install Vagrant [here](https://www.vagrantup.com/docs/installation/)

With vagrant installed: check success of installation with `vagrant help`
Next you will make a new directory called `vagrant` and set up a basic Vagrant configuration file that we can change to suit our needs.
This can be done by running `vagrant init.`

```bash
$ git clone https://github.com/Cornell-PoBE/A3
$ cd A3
$ pwd
<CURR_DIRECTORY>/A3
$ mkdir vagrant
$ cd vagrant
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Now we will customize the `Vagrantfile` for our use case. Our changes for our starter app will be the following:

* Change the config.vm.box value to ubuntu/trusty64: this sets our Virtual Machine to use one of the latest Ubuntu distributions.
* Uncomment the line that has private_network in it and note the IP address. (By default, it’s 192.168.33.10).

Your Vagrantfile should now look like this:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
```

Next, in your terminal, you should run `vagrant up`.
This will download the Ubuntu image that we specified as the config.vm.box value and create your server as a virtual machine.
This might take a little while, depending on your connection speed.
Once it completes, run vagrant ssh. It will put us into the command line of our virtual machine (VM).

```bash
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/trusty64'...
...
$ vagrant ssh
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-95-generic x86_64)
...
vagrant@vagrant-ubuntu-trusty-64:~$
```

Now in your VM you can run the following:

```bash
vagrant@vagrant-ubuntu-trusty-64:~$  sudo apt-get update
vagrant@vagrant-ubuntu-trusty-64:~$  sudo apt-get install git python-pip
...
Do you want to continue? [Y/n] Y
...
vagrant@vagrant-ubuntu-trusty-64:~$  git clone https://github.com/Cornell-PoBE/A3
vagrant@vagrant-ubuntu-trusty-64:~$  cd A3
vagrant@vagrant-ubuntu-trusty-64:~/A3$ sudo pip install -r requirements.txt
...
Successfully installed Flask gunicorn Werkzeug Jinja2 itsdangerous MarkupSafe
Cleaning up...
vagrant@vagrant-ubuntu-trusty-64:~/A3$ python app.py
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```
Now you can navigate to: `http://192.168.33.10:5000/` and see the app.

This is a simple way of bringing our app online. But let's say we want to automate this, and in the future automate more complicated instructions and executions.

### Ansible Setup
We will be using [Ansible](https://www.ansible.com/) and its playbooks to automate the instructions above, as an example.
Essentially, it will log in to servers that you specify using `ssh` and run commands on them.

To begin, you will need to exit the VM we created and install Ansible on the host machine.
```bash
vagrant@vagrant-ubuntu-trusty-64:~/A3$ exit
logout
Connection to 127.0.0.1 closed.
$ sudo pip install ansible
...
Successfully installed PyYAML-3.12 ansible-2.3.0.0 asn1crypto-0.22.0 cryptography-1.8.1 idna-2.5 ipaddress-1.0.18 packaging-16.8 paramiko-2.1.2 pycrypto-2.6.1
```
Now in your `vagrant` folder we will create a file called `site.yml`.

This will be our Ansible [playbook](http://docs.ansible.com/ansible/playbooks.html), and it will contain our automation steps. `site` implies that this is the only file needed to get a successful version of our site up and running. The `.yml` extension tells us that it’s a YAML-formatted file (Ansible’s preference).

Your site.xml would look something like this, for our above example:

```yml
# This is an example of a simple Ansible cookbook
---
- name: Starting a Simple Flask App
  hosts: all
  remote_user: root
  become: true
  become_method: sudo
  vars:
      repository_url: https://github.com/Cornell-PoBE/A3
      repository_path: /home/vagrant/A3

  tasks:
    - name: Install necessary packages
      apt: update_cache=yes name={{ item }} state=present
      with_items:
        - git
        - python-dev
        - python-pip
    - name: Check if A3 directory exists
      stat: path='{{ repository_path }}'
      register: a3_cloned
    - name: Pull application repo
      command: chdir='{{ repository_path }}' git pull origin master
      when: a3_cloned.stat.exists
    - name: Clone application repo
      git: repo='{{ repository_url }}' dest='{{ repository_path }}'
      when: a3_cloned.stat.exists == false
    - name: Install pip requirements
      pip: requirements='{{ repository_path }}/requirements.txt'
```

Now that we have the playbook defined, we’ll need to tell our VM to use it when setting itself up. In your Vagrantfile, uncomment the bottom-most section on provisioning and change it to look like this:

```ruby
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'site.yml'
    ansible.verbose = 'v'
  end
```

We’re telling Vagrant to use the `site.yml` file we created and to use verbose output. Then, let’s reprovision your host. This will run all the commands we defined in the playbook on it. `vagrant provision` should take care of it, but we will destroy the VM and restart it to prove that it works.

```bash
$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/trusty64'...
...
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
...
$ vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ cd A3
vagrant@vagrant-ubuntu-trusty-64:~/A3$ python app.py
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

We have now built a fully automated script. For other examples your scripts will be more complex with templates, hosts, roles, etc.

Next, we’ll need a webserver to serve requests. Serving requests through the application’s debug server poses serious security risks and it’s not intended for anything like a production load.

To serve our requests, we’re going to use `gunicorn`. Other popular options are `uWSGI` or gevent but, for the sake of constraining choice, we’ll go with `gunicorn`.

We’re going to assume that the rest of what you’re doing here will be run within the VM unless otherwise specified.

#### Gunicorn Setup

We have already included gunicorn in our `requirements.txt` and since it is able to detect our Flask app, it is already configured! Easy!
You can run `gunicorn` with just `$ gunicorn --bind 0.0.0.0:8000 app:app`. This command will use gunicorn to serve your application through WSGI, which is the traditional way that Python webapps are served. It replaces our usual `python app.py` step. This is the simplest way to serve our application for now.

#### nginx setup

Now that we have `gunicorn` configured, we need an HTTP server to handle the requests themselves and make sure we route our users to the right application. We’ll use [nginx](https://www.nginx.com/) to do this.

As such, we will need to setup an `nginx` configuration file.
The goal of this configuration file is to make sure that we can access our server at the same host (`192.168.33.10`) but without needing to specify a port.
The nginx configuration will be created in the vagrant directory as well:

```bash
$ pwd
<CURR_DIRECTORY>/A3/vagrant
$ touch a3.nginx.j2
```
We will then edit `a3.nginx.j2` to contain the following:
```j2
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/tmp/a3.sock;
    }
}
```
This file will tell nginx to look for our server,a unix socket, which, if our gunicorn server is running properly, should be accessible.

#### Upstart Scripts

With `gunicorn` rudimentarily configured, we’ll want to set up a script so that we can run our server automatically when our server restarts or just kick the process if it’s stuck. How we’ll do that is with an [upstart](http://upstart.ubuntu.com/) script. This script will handle starting and stopping tasks.

With an upstart script we should be able to run `sudo service nginx restart` to start the task, go in your browser, and then view the service at `http://192.168.33.10`, as before. The big difference is now we have something that will run it for us, so we don’t need to SSH to run our server.

As such we will create an a3-upstart script:

```bash
$ pwd
<CURR_DIRECTORY>/A3/vagrant
$ touch upstart.conf.j2
```

We will modify that file to contain this:

```j2
description "a3"

start on (filesystem)
stop on runlevel [016]

respawn
setuid nobody
setgid nogroup
chdir {{ repository_path }}

exec gunicorn app:app --bind unix:/tmp/a3.sock --workers 3
```

However we must ensure that Ansible copies those two file into the `vagrant` directory, so each of the VMs will have access to the files.

As such, you will modify your site.yml to be this:

```yml
# This is a simple example Ansible playbook
---
- name: Starting a Simple Flask App
  hosts: all
  remote_user: root
  become: true
  become_method: sudo
  vars:
      repository_url: https://github.com/Cornell-PoBE/A3
      repository_path: /home/vagrant/A3
  tasks:
    - name: Install necessary packages
      apt: update_cache=yes name={{ item }} state=present
      with_items:
        - git
        - python-dev
        - python-pip
        - nginx
    - name: Check if A3 directory exists
      stat: path='{{ repository_path }}'
      register: a3_cloned
    - name: Pull application repo
      command: chdir='{{ repository_path }}' git pull origin master
      when: a3_cloned.stat.exists
    - name: Clone application repo
      git: repo='{{ repository_url }}' dest='{{ repository_path }}'
      when: a3_cloned.stat.exists == false
    - name: Install pip requirements
      pip: requirements='{{ repository_path }}/requirements.txt'
    - name: Copy Upstart configuration
      template: src=upstart.conf.j2 dest=/etc/init/upstart.conf
    - name: Make sure our server is running
      service: name=upstart state=started
    - name: Copy Nginx site values
      template: src=a3.nginx.j2 dest=/etc/nginx/sites-enabled/a3
      notify:
        - restart nginx
    - name: Remove any default sites
      file: path=/etc/nginx/sites-enabled/default state=absent
      notify:
        - restart nginx
    - name: Make sure nginx is running
      service: name=nginx state=started
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

We’re using Ansible’s template and service modules to accomplish our task. What we’re saying is that we want to copy the template file that we defined into the directory that we used before. It will use the variables we have defined in our file, inject them into the template, and write them to the destination path we have defined.

Then, we want to make sure our service has started. If it hasn’t been started yet, start it.

The removal of `etc/nginx/sites-enabled/default` is necessary since there is a [problem](http://stackoverflow.com/questions/14972792/nginx-nginx-emerg-bind-to-80-failed-98-address-already-in-use) with the `default` site that’s enabled by `nginx`. We don’t need it, so we remove it.

You can now run `vagrant provision`, `vagrant ssh`, and `sudo service nginx restart` to start the web server.

```bash
$ pwd
<CURR_DIRECTORY>/A3/vagrant
$ ls
Vagrantfile     a3.nginx.j2     site.yml        upstart.conf.j2
$ vagrant provision
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
...
TASK [Make sure nginx is running]
...
$ vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ cd A3
vagrant@vagrant-ubuntu-trusty-64:~/A3$ sudo service nginx restart
 * Restarting nginx nginx  						[OK]
```

You can now navigate to `http://192.168.33.10/` and see your application!

The next step in our tutorial is to deploy our application to Amazon AWS.

#### Amazon AWS

You will be required to create an Amazon Account and launch an EC2 instance.

This EC2 instance should be using `Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-7c22b41c` as an AMI.
This AMI will be the same type of OS that we used for our VM.

I would recommend that you choose the `t2.micro`, which is a small, free tier-eligible instance type.

After launching, download the the key-pair and name it `a3keypair` for the sake of consistency with this tutorial.

Launch your instance, ssh into your instance, and create a private key with the following commands:

```bash
$ chmod 400 <CURR_DIRECTORY>a3keypair.pem
$ ssh -i <CURR_DIRECTORY>a3keypair.pem ubuntu@<SERVER_PUBLIC_IP>
...
ubuntu@ip-<SERVER_PRIVATE_IP>:~$ exit
logout
Connection to <SERVER_PUBLIC_IP> closed.
$ ssh-keygen
enerating public/private rsa key pair.
Enter file in which to save the key (/Users/<YOUR_USERNAME>/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/<YOUR_USERNAME>/.ssh/id_rsa.
Your public key has been saved in /Users/<YOUR_USERNAME>/.ssh/id_rsa.pub.
...
```
Now that we have a VM setup on EC2, let’s configure Ansible to provision it.

Let’s first make sure Ansible can ping your host and use SSH to log in to it. Before we can begin, we need to set up the [inventory](http://docs.ansible.com/ansible/intro_inventory.html) file so Ansible knows about our server.

Create a file called hosts in your Ansible directory:

```bash
ubuntu@ip-<SERVER_PRIVATE_IP>:~$ exit
logout
Connection to <SERVER_PUBLIC_IP> closed.
$ pwd
<CURR_DIRECTORY>/A3/vagrant
$ touch hosts
```

Modify that file to look like this:

```yml
[webservers]
 <SERVER_PUBLIC_IP> ansible_ssh_user=ubuntu
```

In this file, we’re describing a group of servers: `webservers`, with a single host: the public server IP from our EC2.

Now run the following command:

```bash
$ ansible -m ping webservers --private-key=a3keypair.pem --inventory=hosts --user=ubuntu
<SERVER_PUBLIC_IP> | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Let's automate this `ssh` like process with an Ansible [config](http://docs.ansible.com/ansible/intro_configuration.html) file to automate it.
Create a file named `ansible.cfg` in your vagrant directory:

```bash
$ pwd
<CURR_DIRECTORY>/A3/vagrant
$ touch ansible.cfg
```

Modify that file to look like this:

```yml
[defaults]
inventory=hosts
remote_user=ubuntu
private_key_file=a3keypair.pem
```

Now if you can run your ansible-playbook on your server and navigate to your EC2's Public IP and see your app live!

```bash
$ ansible-playbook -v site.yml
...
PLAY RECAP ***************************************************************************
<SERVER_PUBLIC_IP>              : ok=11   changed=8    unreachable=0    failed=0

```

You now have an up and running EC2 instance!

For further extension, not necessary to know for the scope of this class, we will be looking about automating the deploying of the EC2 instance.

For this, we will use [Terraform](https://www.terraform.io/) to provision our infrastructure. There are many options, one big one is: [CloudFormation](https://aws.amazon.com/cloudformation/), but we will use Terrform for this tutorial.

#### Extensions [ Terraform Setup ]

To download `Terraform` you can download from [here](https://www.terraform.io/downloads.html) or just run `brew install terraform`.
To check that `Terraform` has been successfully installed run `terraform help`.

Basically how Terraform works is that we’ll define the different resources that we want to set up and then we’ll use `Terraform` to plan out what our changes will do and, finally, apply the plan.

Firstly you will make a directory for `Terraform` in your application and create a new file called: `main.tf`. The `.tf`extensions indicate that they’re using the Terraform syntax, which is similar to JSON.

```bash
$ pwd
<CURR_DIRECTORY>/A3
$ mkdir terraform
$ cd terraform
$ touch main.tf
```

Modify the `main.tf` file to include the following:

```tf
provider "aws" {
    access_key = "${var.aws_access_key}"
    secret_key = "${var.aws_secret_key}"
    region = "${var.aws_region}"
}

resource "aws_instance" "a3" {
    ami = "ami-7c22b41c"  # Ubuntu 14.04 for us-east-1
    instance_type = "t2.micro"
    vpc_security_group_ids = ["${aws_security_group.web.id}"]
    key_name = "${aws_key_pair.a3.id}"
}

resource "aws_security_group" "web" {
    name = "web"
    description = "Allow HTTP and SSH connections."
    ingress {
        from_port = 22
        to_port = 22
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    ingress {
        from_port = 80
        to_port = 80
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }
}

resource "aws_key_pair" "a3keypair" {
    key_name = "a3keypair"
    public_key = "${file(var.public_key_path)}"
}
```
In the first provider block, we specify our credentials for AWS. The `${var.aws_secret_key}` value, for example, tells `Terraform` to look for a variable named `aws_secret_key` and plug the value in there.

For our instance, we set the Amazon AMI ID to match the one we used previously in the tutorial. We also set up the key and security groups to reference other resources. `"${aws_security_group.web.id}"` tells `Terraform` to look for another `aws_security_group` resource named `web` and plug in its `id`.

Next, we set up the `security group` as we did in the tutorial above. We’re letting in SSH control (port 22), HTTP (port 80), and nothing else. We’re allowing all outbound traffic.

Finally, we create a new `aws_key_pair` to remotely ssh into our server, just as we did when setting it up manually above.

Now, we had defined a few [variables](https://www.terraform.io/intro/getting-started/variables.html) in our `main.tf` file, and we need to provide the values for those to `Terraform`. As such we will create a new file called variables.tf.

```bash
$ pwd
<CURR_DIRECTORY>/A3/terraform
$ touch variables.tf
```

Modify this file to have this:

```tf
variable "aws_access_key" {} # This variable come from the terraform.tfvars file
variable "aws_secret_key" {} # This variable come from the terraform.tfvars file

variable "aws_region" {
    description = "AWS region in which to launch the servers."
    default = "us-east-1"
}

variable "public_key_path" {
    default = "~/.ssh/id_rsa.pub"
}
```

Finally, we’ll need one more file, which we’ll want to be sure we don’t include in version control. This file will be `terraform.tfvars`, which, by convention, contains secret keys and such.

```bash
$ pwd
<CURR_DIRECTORY>/A3/terraform
$ touch terraform.tfvars
```

Mofidy this file to have this:

```tfvars
aws_access_key = "<YOUR_AWS_ACCESS_KEY>"
aws_secret_key = "<YOUR_AWS_SECRET_KEY>"
```

To get an AWS access and secret key it will require you to go to the IAM portion of in AWS and doing the following steps:

* Select the “Identity & Access Management” section
* Select Users
* Click the “Create New Users” button and following the prompts
* Select "Set permissions for <YOUR_USER>" and choose “admin" from the dropdown menu. Save your changes.
* Create user
* Grab Access Key and Secret Key

Next we would like to set it up using Ansible but, in order to do that, we need to know the ip address created by our `Terraform` plan.

To do this we will leverage [output variables](https://www.terraform.io/intro/getting-started/outputs.html).

As such we wil need to make one more file:

```bash
$ pwd
<CURR_DIRECTORY>/A3/terraform
$ touch output.tf
```

Modify this file to look like this:

```tf
output "ip" {
    value = "${aws_instance.a3.public_ip}"
}
```

This will reference our resources and show us the IP address for the created instance called `a3`.
Now you can terminate the instance you created and have `Terraform` handle it for you.

Now run the following:

```bash
$ pwd
<CURR_DIRECTORY>/A3/terraform
$ terraform plan; terraform apply; terraform output
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

aws_security_group.web: Refreshing state... (ID: sg-3f7b4c40)
aws_key_pair.a3: Refreshing state... (ID: a3keypair)
...
aws_instance.a3: Creation complete (ID: i-02158fa1be9c4980f)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
...
State path:

Outputs:

ip = <YOUR_NEW_PUBLIC_IP>
```

And add the result to your `host` file in your `vagrant` folder.

```yml
[webservers]
 <YOUR_NEW_PUBLIC_IP> ansible_ssh_user=ubuntu
```

Last step is to modify the settings in `Ansible` by changing the `ansible.cgf` to look like this:

```cfg
[defaults]
inventory=hosts
remote_user=ubuntu
private_key_file=a3keypair.pem

[ssh_connection]
pipelining = True
```

Now you can run `ansible-playbook -v site.yml` and you are all set!

```bash
$ pwd
<CURR_DIRECTORY>/A3/terraform
$ cd ..
$ cd vagrant
$ ansible-playbook -v site.yml
```

## Assignment Walkthrough

Assuming you have read the above or are aware of each of the technologies listed in [here](#learning-objectives) you may follow this guide to deploy your A2 application both on your `Vagrant` Ubuntu VM and on AWS EC2.

Follow the instructions carefully keeping the knowledge from above in mind:
```bash
# Clone your A2 repository that you submitted on CMS
$ git clone <YOUR_REPO>
# Change directory into your repository
$ cd <YOUR_REPO>
# Add a Vagrant directory where you will contain your Vagrant files
# as well as: ansible cookbooks, keypair pem files, and upstart scripts
$ mkdir vagrant
# add vagrant/* to your .gitignore either via a text editor or by running
# echo "vagrant/*" >> .gitignore
$ cd vagrant
# Initialize Vagrant with a Vagrantfile
$ vagrant init
$ ls
Vagrantfile
# You will now add the following files each of which should be familiar
# from the example walkthrough
$ touch site.yml
$ touch a3.nginx.j2
$ touch ansible.cfg
$ touch upstart.conf.j2
$ touch hosts
$ ls
Vagrantfile     a3.nginx.j2     ansible.cfg     hosts           site.yml        upstart.conf.j2
$ touch venv-requirements.txt
```

Make sure venv-requirements.txt include these fields:

venv-requirements.txt:
```txt
ansible==2.3.2.0
asn1crypto==0.22.0
awscli==1.11.138
bcrypt==3.1.3
botocore==1.6.5
cffi==1.10.0
colorama==0.3.7
cryptography==2.0.3
docutils==0.14
enum34==1.1.6
futures==3.1.1
idna==2.6
ipaddress==1.0.18
Jinja2==2.9.6
jmespath==0.9.3
MarkupSafe==1.0
paramiko==2.2.1
pyasn1==0.3.2
pycparser==2.18
pycrypto==2.6.1
PyNaCl==1.1.2
python-dateutil==2.6.1
python-vagrant==0.5.15
PyYAML==3.12
rsa==3.4.2
s3transfer==0.1.10
six==1.10.0
```

Next you will ensure that you are working with the proper working environment that this guide is using:
```bash
$ virtualenv venv
$ ls
Vagrantfile     a3.nginx.j2     ansible.cfg     hosts           site.yml        upstart.conf.j2   venv      venv-requirements.txt
$ source venv/bin/activate
$ pip install -r venv-requirements.txt
```

At this point in time you will be required to setup an AWS account and launch an EC2 instance.
Follow these steps to have a proper t2.micro instance setup:

1. Direct to AWS main site [here](https://aws.amazon.com/)
2. Sign in or Sign up
3. Direct to US-West (Oregon) [here](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2)
4. Go to EC2 Instance [NOC](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#Instances:sort=instanceState) or Network Operations Console
5. Launch Instance
6. Step 1: Under AMI choose `Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-7c22b41c`
7. Step 2: Choose t2.micro which is the free tier
8. Skip to Step 6 and do the following:
  * Create a new security group
  * Set security group name: `pobe`
  * Set description: `Backend Engineering Security Group`
  * Use the following configs:

| Type  | Ports | Protocol | Source          |
|-------|:-----:|:--------:|-----------------|
| HTTP  |   80  |    tcp   | 0.0.0.0/0, ::/0 |
| SSH   |   22  |    tcp   | 0.0.0.0/0, ::/0 |
| HTTPS | 443   |    tcp   | 0.0.0.0/0, ::/0 |
9. Step 7: Launch Instance
10. Download the keypair and place into your `vagrant` directory
  * Create a new key pair
  * Set Key pair name: `a3keypair`
  * Download Key Pair
  * Place key pair file into `/<YOUR_REPO>/vagrant/`
11. Wait until instance is green and with an instance state of: `running`
12. Grab IPv4 Public IP, which I will call `PublicIP` for the rest of tutorial

You will now run the following back in your project directory:
```bash
$ cd vagrant
# Checking to make sure you have included the keypair file
$ ls
Vagrantfile     a3.nginx.j2     a3keypair.pem   ansible.cfg     hosts           site.yml        upstart.conf.j2
# Ensure that you have a private key by running ssh-keygen using the AWS CLI
...
Succesfully installed
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/<YOUR_USERNAME>/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/<YOUR_USERNAME>/.ssh/id_rsa.
Your public key has been saved in /Users/<YOUR_USERNAME>/.ssh/id_rsa.pub.
...
```
I will now go through how each of your files should look for this app to be deployed:

Vagrantfile:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'site.yml'
    ansible.verbose = 'vvv'
  end
end
```

hosts:
```
[webservers]
 <PublicIP> ansible_ssh_user=ubuntu
```

upstart.conf.j2
```yml
description "upstart"

start on (filesystem)
stop on runlevel [016]

respawn
setuid root
setgid nogroup
chdir {{ repository_path }}/src

# For this portion I have included -e which are the environmental variables you have included in your .env file.
# Because you are gitignoring the vagrant directory you can put sensitive information here
# Put your respective environmental variables here
exec gunicorn app:app --bind=unix:/tmp/a3.sock --workers 3 --user=root --log-level=debug -e DB_USERNAME={{ DB_USERNAME }} -e DB_PASSWORD={{ DB_PASSWORD }} -e DB_NAME={{ DB_NAME }} -e DB_HOST={{ DB_HOST }} -e APP_SETTINGS={{ APP_SETTINGS }}
```

a3.nginx.j2:
```j2
server {
    listen 80;
    client_max_body_size 4G;
    location / {
        include proxy_params;
        proxy_pass http://unix:/tmp/a3.sock;
    }
}
```

ansible.cfg:
```cfg
[defaults]
inventory=hosts
remote_user=ubuntu
private_key_file='~/.ssh/id_rsa'

[ssh_connection]
pipelining = True
```

site.yml:
```yml
---
- name: Deploying your A2
  hosts: all
  remote_user: root
  become: true
  become_method: sudo
  environment:
    LC_ALL: en_US.UTF-8
    LANG: en_US.UTF-8
    LANGUAGE: en_US.UTF-8
  vars:
      repository_git_path: <GITHUB_USERNAME>/<PRIVATE_REPO_DIR>
      repository_path: /home/vagrant/<PRIVATE_REPO_DIR>
      DB_USERNAME: <DB_USER_NAME>
      DB_PASSWORD: <DB_PASSWORD>
      DB_NAME: pobe_a2_db
      DB_HOST: localhost
      APP_SETTINGS: config.ProductionConfig
      GITHUB_USER: <GITHUB_USERNAME>
      GITHUB_PASSWORD: <GITHUB_PASSWORD>
  tasks:
    - name: Install necessary packages
      apt: update_cache=yes name={{ item }} state=present
      with_items:
        - libatlas-base-dev
        - gfortran
        - g++
        - build-essential
        - libssl-dev
        - libffi-dev
        - python-dev
        - libmysqlclient-dev
        - mysql-server
        - libpq-dev
        - git
        - python-pip
        - nginx
        - python-numpy
        - python-scipy
        - gridsite-clients
    - name: Check if directory exists
      stat: path='{{ repository_path }}'
      register: cloned
    - name: Pull application repo
      git:
        repo: "https://{{ GITHUB_USER | urlencode }}:{{ GITHUB_PASSWORD }}@github.com/{{ repository_git_path }}.git"
        dest: '{{ repository_path }}'
        clone: no
      when: cloned.stat.exists
    - name: Get updated files from git repository
      git:
        repo: "https://{{ GITHUB_USER | urlencode }}:{{ GITHUB_PASSWORD }}@github.com/{{ repository_git_path }}.git"
        dest: '{{ repository_path }}'
      when: cloned.stat.exists == false
    - name: Install pip requirements
      command: pip install -r {{ repository_path }}/src/requirements.txt
    - name: Copy Upstart configuration
      template: src=upstart.conf.j2 dest=/etc/init/upstart.conf
    - name: Create a new database with name {{ DB_NAME }}
      mysql_db:
        name: '{{ DB_NAME }}'
        state: present
    - name: Create database user that you have defined above
      mysql_user: user={{ DB_USERNAME }} password={{ DB_PASSWORD }} priv=*.*:ALL state=present
    - name: Check if migration exists
      stat: path={{ repository_path }}/src/migrations
      register: migrations      
    - name: Run initialization script
      command: python manage.py db init  
      environment:
        DB_USERNAME: '{{ DB_USERNAME }}'
        DB_PASSWORD: '{{ DB_PASSWORD }}'
        DB_NAME: '{{ DB_NAME }}'
        DB_HOST: '{{ DB_HOST }}'
        APP_SETTINGS: '{{ APP_SETTINGS }}'
      args:
        chdir: '{{ repository_path }}/src/'
      when: migrations.stat.exists == false
    - name: Run migrations script
      command: python manage.py db migrate
      environment:
        DB_USERNAME: '{{ DB_USERNAME }}'
        DB_PASSWORD: '{{ DB_PASSWORD }}'
        DB_NAME: '{{ DB_NAME }}'
        DB_HOST: '{{ DB_HOST }}'
        APP_SETTINGS: '{{ APP_SETTINGS }}'
      args:
        chdir: '{{ repository_path }}/src/'
      when: migrations.stat.exists == false  
    - name: Run upgrade script
      command: python manage.py db upgrade
      environment:
        DB_USERNAME: '{{ DB_USERNAME }}'
        DB_PASSWORD: '{{ DB_PASSWORD }}'
        DB_NAME: '{{ DB_NAME }}'
        DB_HOST: '{{ DB_HOST }}'
        APP_SETTINGS: '{{ APP_SETTINGS }}'
      args:
        chdir: '{{ repository_path }}/src/'
    - name: Make sure our server is running
      service: name=upstart state=started
    - name: Copy Nginx site values
      template: src=a3.nginx.j2 dest=/etc/nginx/sites-enabled/a3
      notify:
        - restart nginx
    - name: Remove any default sites
      file: path=/etc/nginx/sites-enabled/default state=absent
      notify:
        - restart nginx
    - name: Make sure nginx is running
      service: name=nginx state=started
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

As you can the `.yml` contains sensitive information like your Github credentials and mySQL configs.
So make sure to `.gitignore` the file.

These files should be sufficient for setting up your environment. To proceed you will just run the following commands to launch you VM:

```bash
$ ls
README.md board.png src       vagrant
$ cd vagrant
$ ls
Vagrantfile           a3keypair.pem         hosts                 upstart.conf.j2       venv-requirements.txt
a3.nginx.j2           ansible.cfg           site.yml              venv
$ source venv/bin/activate
$ vagrant up
$ vagrant provision # nginx runs your app locally at: `192.168.33.10` on your Ubuntu VM
$ chmod 400 a3keypair.pem # make sure that your private key file isn't publicly viewable
$ ansible -m ping webservers --private-key=a3keypair.pem --inventory=hosts --user=ubuntu # ping and add host
The authenticity of host '<PublicIP> (<PublicIP>)' cant be established.
ECDSA key fingerprint is SHA256:Y35MHCMwSCH/GZyak+HWkd5bUCKm6JtMOuMCxMAnpkI.
Are you sure you want to continue connecting (yes/no)? yes
<PublicIP> | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
$ ansible-playbook -v site.yml # ansible runs your app on EC2
Using /Users/ilanfilonenko/flaskplate/vagrant/ansible.cfg as config file

PLAY [Deploying your A2] ****************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [<PublicIP>]
...
...
PLAY RECAP ******************************************************************************************************************************************************************************
<PublicIP>                 : ok=16   changed=12   unreachable=0    failed=0
```

Now your app is deployed on EC2 and up on your local VM. Yay!

## Expected Functionality
For this assignment we will be asking you to complete two things:
* simulate your A2 application in Vagrand on an `Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-7c22b41c` VM
* deploy your application onto an EC2 instance via a single `ansible` command

To verify, you will be submitting your EC2 public url in a `.txt`: which we will be accessing to verify the deployment.
You may use the exact file-structure that we have provided for you above or you may deviate from it with your own specifications.

## Extending the Assignment
To extend this assignment you may:
* automating the EC2 deployment with Terraform as noted in the extended section as part of the example walkthrough
* deploy with `Docker` and `Kubernetes` in `Minikube` and then `Google Cloud Engine` to build a scalable application that has load balancing

## Project Submission
You should submit a `.txt` file containing only the public EC2 url, along with a `readme.txt` for your citations, project setup information, and any extensions you might have done.

You can then submit this file to `CMS`.
