---
author: fiunchinho
categories:
- Code
date: 2017-05-20T21:00:31Z
tags:
- deployment
- wimpy
- aws
title: Introducing Wimpy
url: /introducing-wimpy/
thumbnail: "images/wimpy.png"
---

Today I want to share with you the project I've been working on for the last year in my free time. I think the project has already been a success: it's a pet project that has allowed me to learn a lot about Amazon Web Services, Docker and Ansible. But letting that aside, I believe is a useful project that can solve deployment problems for you.


<!--more-->
[Wimpy is an open source Platform as a Service](https://github.com/wimpy) that you run from your terminal to deploy your applications to AWS, following cloud best practices.

Heavily inspired on [Ansistrano](https://github.com/ansistrano/deploy), it's built as a set of Ansible roles and it's executed using an Ansible playbook in your application's repository.

Wimpy's goal is to make it easy to follow AWS best practices, embracing immutable infrastructure patterns where your servers are considered to be immutable. Whenever a new version is released, the servers get never updated but replaced by new servers containing the new released version.

## Usage

Let's see an example for a playbook, where you can configure how Wimpy will deploy your application:

```yaml
- hosts: all
  connection: local
  vars:
    # Name of the application, used to name resources in AWS
    wimpy_application_name: "awesome-application"
    # Where your application is listening for HTTP requests
    wimpy_application_port: "80"
    # It will create a new DNS for awesome-application.armesto.net
    wimpy_aws_hosted_zone_name: "armesto.net"
  roles:
    - role: wimpy.environment
    - role: wimpy.build
    - role: wimpy.deploy
```

Now you can just run the playbook using Ansible, or, if you don't have Ansible installed, you can just use our Docker image that contains Ansible and all Wimpy roles:

```bash
$ docker run -v /var/run/docker.sock:/var/run/docker.sock \
	-v "$PWD:/app" fiunchinho/wimpy /app/deploy.yml \
    --extra-vars "wimpy_release_version=`git rev-parse HEAD` \
                  wimpy_deployment_environment=production"
```

In this case, there are two variables that we want to pass on every deploy we make: the version being deployed and in which environment we want to deploy it. In this case, the version being deployed is the SHA1 commit hash from Git, but you can choose any arbitrary tag for your version, like "v1.4.2" or whatever.

Once executed, you can browse to [http://awesome-application.armesto.net](http://awesome-application.armesto.net) to see your application, and you will have a bunch of resources in your AWS account.

## What happened
As I said earlier, Wimpy has been build with modularity in mind, so it's not an all-or-nothing kind of tool. You can choose which roles to execute.

### wimpy.environment
First of all, this role will enable CloudTrail for your account, an audit log so you can track *who-did-what-when* in your account.
It also registers a master key in KMS for applications to encrypt and decrypt secrets.

Your AWS account must be prepared before you can deploy your applications. By executing this role, your accout gets two different isolated environments called `staging` and `production`. Each environment gets its own Virtual Private Cloud, meaning that applications in one environment can't connect to applications in a different environment. Total isolation between environments right from the start.

The same applies for S3. Wimpy creates a single bucket for all your applications, but each application will only be able to access the application folder in the environment where is running. For example, if your S3 bucket is called `wimpy_storage_bucket`, the application called `cats-api` will have access to the `wimpy_storage_bucket/production/cats-api/` folder when deployed in `production`, but it will have access to `wimpy_storage_bucket/staging/cats-api/` when deployed in the `staging` environment.

Last but not least, this role creates the needed security groups to allow traffic to your applications:

```
The Internet --> Load Balancers port --> Application port --> Databases
```

### wimpy.build
Docker is not only great for running applications but for packaging as well. By default, this role will create an Elastic Container Registry on AWS to store your applications images, but you can choose any Docker Registry that you like.

On every deployment, it will package your application as a Docker image that can be run anywhere.

### wimpy.deploy
This is probably the most interesting part!
This role will deploy your application as an Auto Scaling Group with Scaling Policies that will scale up and down the number of instances. Each instance will run the application's Docker container, supervised by systemd.

If you want, it will create a new DNS entry pointing to your Load Balancer, which will balance the traffic between all the instances in your Auto Scaling Group.

## More info
You can learn more about the project [in the documentation page](https://wimpy.github.io/docs/).
Feel free to read through [the code in the Github organization](https://github.com/wimpy). Contributions are really welcomed!

## Acknowledgements
Finally, I'd like to thank all the people that have helped me during the development of this project. This would've never happened without them, specially:
- Ismael Fernández
- Alberto Ramírez
- Víctor Caldentey
- Fabrizio Di Napoli