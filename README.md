# Forked from https://github.com/DandyDev/graphite-statsd-ansible-vagrant
This adds an EC2 deployment script for installing to AWS. The script that this was forked from has a solid setup for Graphite/Statd, and the choices made for what services to install makes it easy to scale if needed. 

It uses Postgres for the data store, if this becomes a bottleneck it can easily be migrated to RDS. 

It also uses memcached which can be migrated to ElasiCache.

In addition to the Docs below Several additional things had to be setup to make this work. 

## Running the EC2 deploy
ansible-playbook -i local ec2.yml

## Configuring the Script
There are several options in the ec2.yml that may need to be adjusted.
####image: ami-b08b6cd8 
This is Ubuntu 12.04 LTS base install from us-east-1. https://cloud-images.ubuntu.com/locator/ec2/ lists other Ubuntu images.
####keypair: aws-key
This is the key pair that the new instance will setup for connection via SSH. It needs already be setup in your EC2 configuration. 
####instance_type: t1.micro
This the smallest available instance, and may not have enough resources.

## SSH
If you have multiple SSH keys to manage, there is an environment variable to tell Ansible which one to use. 

export ANSIBLE_SSH_ARGS="-i /Users/<username>/.ssh/aws-key.pem"

## AWS environment variables to set up

export EC2_REGION=us-east-1

~/.aws/credentials
[Credentials]
aws_access_key_id = <aws key>
aws_secret_access_key = <aws secret>

The Key and secret could also be setup using the following Environment Variables:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

## Boto
Sometimes on OSX it can be difficult to get the correct python setup to work. Boto was giving me issues with finding its config file. Setting it's environment variable fixed the issue.

export BOTO_CONFIG=~/.boto 

# The rest of this document is from the original script

# Graphite & StatsD with Ansible

NOTE: If you want an even more complete monitoring solution that includes [Sentry](http://getsentry.com), the error logging tool, take a look at [Statserver](https://github.com/DandyDev/statserver)

This playbook makes it really easy to setup [Graphite](http://graphite.readthedocs.org/en/latest/) and [StatsD](https://github.com/etsy/statsd/) on a server (VPS or Dedicated). You can also optionally install it on a Virtual Machine using Vagrant so you can play around with it. It uses [Ansible](http://www.ansible.com/), a great configuration management tool written in Python, to automatically install the applications and all dependencies and configure everything to work optimally.

What gets installed:

*  PostgreSQL database
*  NginX webserver/reverse proxy
*  Python, Pip & VirtualEnv
*  NodeJS
*  Memcached
*  The 3 core Graphite components:
	* [Carbon](https://github.com/graphite-project/carbon)
	* [Whisper](https://github.com/graphite-project/whisper)
	* [The Graphite webapp](https://github.com/graphite-project/graphite-web)
* StatsD  

## Let's do this!

If you want to install Graphite on a VM using Vagrant, you first need to install [Vagrant](http://www.vagrantup.com/) and a Virtual Machine provider of choice ([VirtualBox](https://www.virtualbox.org/) is free and works out of the box with Vagrant).

You can configure your install by modifying the variables in the _graphite.yml_ file before provisioning.

Then: 

```
$ git clone https://github.com/DandyDev/graphite-statsd-ansible-vagrant
$ cd /path/to/graphite-statsd-ansible-vagrant
$ vagrant up
```

## Different OSes

By default, the Vagrant box runs Ubuntu 12.04, but the playbook supports Debian 7 and CentOS 6.4 as well! To try those out, uncomment the appropriate lines in the Vagrantfile and comment out the Debian lines.

## Using the playbook standalone

You can of course also use the playbook without Vagrant. In that case you must provide your own inventory file specifying the host on which to install Sentry. The playbook has been tested on Ubuntu 12.04, Debian 7 and CentOS 6.4. Other flavors of Linux might work as well.

## Secret key

On production environments you will want to set the ``secret_key`` setting under the ``graphite`` namespace to a unique key that acts as a signing token. Generate a secret key for [here](http://www.miniwebtool.com/django-secret-key-generator/)

## Superuser

To create a superuser, log in as root, or turn to root on your server, and issue the following command:

```
export PYTHONPATH=/opt/graphite/webapp/; /opt/graphite/bin/python /opt/graphite/bin/django-admin.py createsuperuser --settings=graphite.settings
```

## Known issues / TODO

* This hasn't been tested on other Providers than VirtualBox yet
* On CentOS, the firewall is completely closed by default (at least on the box I tried it with). So you have to manually open the relevant port (80) using `iptables`, or if you're not concerned about security (because you're running it locally through Vagrant), you can always flush the firewall with `iptables -F`.

## Contribute

If you have any suggestions, feel free to create an issue here on Github and/or fork this repo, make changes and submit a pull request!
