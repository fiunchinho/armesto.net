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

Today I want to share with you [the project I've been working on for the last year](https://github.com/wimpy) in my free time. For me, the project has already been a success: it's a pet project that has allowed me to learn a lot about Amazon Web Services, Docker and Ansible. But letting that aside, I believe is a useful project that can help you deploy your software better!

<!--more-->
[Wimpy is an open source Platform as a Service](https://github.com/wimpy) that you run from your terminal to deploy your applications to AWS, following cloud best practices.

Heavily inspired on the great [Ansistrano](https://github.com/ansistrano/deploy), it's built as a set of Ansible roles and it's executed using an Ansible playbook in your application's repository.

Wimpy's goal is to make it easy to follow AWS best practices, embracing [immutable infrastructure patterns](https://martinfowler.com/bliki/ImmutableServer.html) where your servers are considered to be immutable. Whenever a new version is released, the servers get never updated but replaced by new servers containing the new released version.

## Usage

Let's see an example for a playbook, where you can configure how Wimpy will deploy your application:

```
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

Now you can just run the playbook using Ansible, or, if you don't have Ansible installed, you can just use [our Docker image that contains Ansible and all Wimpy roles](https://hub.docker.com/r/fiunchinho/wimpy/):

```bash
$ docker run -v /var/run/docker.sock:/var/run/docker.sock \
    -v "$PWD:/app" fiunchinho/wimpy /app/deploy.yml \
    --extra-vars "wimpy_release_version=`git rev-parse HEAD` \
                  wimpy_deployment_environment=production"

```

In this case, there are two variables that we want to pass on every deploy we make: the version being deployed and in which environment we want to deploy it. In this case, the version being deployed is the SHA1 commit hash from Git, but you can choose any arbitrary tag for your version, like "v1.4.2" or whatever.

Once executed, you can browse to [http://awesome-application.armesto.net](http://awesome-application.armesto.net) to see your application, and you will have a bunch of resources in your AWS account.

## What happened
As I said earlier, Wimpy has been build with modularity in mind, so it's not an *all-or-nothing* kind of tool. You can choose which roles to execute. Let's see what these roles do.

### wimpy.environment
First of all, this role will enable [CloudTrail for your account](https://aws.amazon.com/cloudtrail/), an audit log so you can track *who-did-what-when* in your account.
It also registers [a master key in KMS](https://aws.amazon.com/kms/) for applications to encrypt and decrypt secrets.

Your AWS account must be prepared before you can deploy your applications. By executing this role, your accout gets two different isolated environments called `staging` and `production`. Each environment gets its own [Virtual Private Cloud](https://aws.amazon.com/vpc/), meaning that applications in one environment can't connect to applications in a different environment. Total isolation between environments right from the start.

The same applies for [S3](https://aws.amazon.com/s3/). Wimpy creates a single bucket for all your applications, but each application will only be able to access the application folder in the environment where is running. For example, if your S3 bucket is called `storage`, the application called `cats-api` will have access to the `storage/production/cats-api/` folder when deployed in `production`, but it will have access only to `storage/staging/cats-api/` when deployed in the `staging` environment.

Last but not least, this role creates the needed security groups to allow traffic to your applications:

- From the internet to the Load Balancer specified port.
- From the Load Balancer to your application exposed port.
- From the application to the databases.

By default, any other access it not allowed, so for example, databases can't be accesed from the internet.

### wimpy.build
[Docker](https://www.docker.com/) is not only great for running applications but for packaging as well. By using Docker for packaging, Wimpy is language agnostic and can deploy any application written in any language.

By default, this role will use an [Elastic Container Registry](https://aws.amazon.com/ecr/) on AWS to store your applications images, but you can choose any Docker Registry that you like.

On every deployment, it will package your application as a Docker image that can be run anywhere. Not only is available for the deployments in production, but different teams within your organization can start sharing their applications to make development easier.

### wimpy.deploy
This is probably the most interesting part!
This role will deploy your application as an [Auto Scaling Group](https://docs.aws.amazon.com/autoscaling/latest/userguide/AutoScalingGroup.html) with [Scaling Policies](https://docs.aws.amazon.com/autoscaling/latest/userguide/policy_creating.html) that will scale up and down the number of instances. Each instance will run the application's Docker container, supervised by systemd.

If you want, it will create a new [Route53 DNS](https://aws.amazon.com/route53/) register pointing to your [Load Balancer](https://aws.amazon.com/elasticloadbalancing/), which will balance the traffic between all the instances in your Auto Scaling Group.

The end result would be something like this:

![](/images/wimpy_deploy.png)

The role offers two different deployment strategies.

#### Rolling Update
Using this deployment strategy, we rely [on the built-in Rolling Update mechanism for Auto Scaling Groups](https://cloudonaut.io/rolling-update-with-aws-cloudformation/), where each instance of the Auto Scaling Group is replaced by a new instance that contains the version being deployed.

If something fails while replacing old instances with new ones, AWS will handle the rollback for us.

#### Blue / Green (Red / Black) deployment
In this strategy, we create a new CloudFormation for every deploy. [This means that every deploy will generate a new Auto Scaling Group](https://martinfowler.com/bliki/BlueGreenDeployment.html) (which instances contain the version being deployed), a new Load Balancer and a new DNS register. Since now we have two different registers for the same domain name (one for each version deployed), Wimpy uses Route53 weighted registers to control how much traffic goes to every version.

You can tune how much traffic goes to new versions, so you can use this feature for [canary releases](https://martinfowler.com/bliki/CanaryRelease.html). This is great for testing in production with real traffic.

## Example
I've two examples that show Wimpy in action

- [Fork of the Symfony demo project](https://github.com/wimpy/symfony-demo), but adding Wimpy to deploy the application from Travis.
- [Fork of the Spring pet-clinic demo project](https://github.com/wimpy/spring-petclinic), also just adding Wimpy to be executed on every merge to master.

## Summary
Wimpy tries to help you automate your deployments and infrastructure best practices in the AWS, making sure that different teams within your organization all deploy applications the same way.

It's composed by different roles that you can combine, or even extend, building your own roles/tasks.

Everything is created on your own AWS account. That means that you are in full control of your resources, and if you don't want to use Wimpy anymore, you won't lose anything.

You can learn more about the project [in the documentation page](https://wimpy.github.io/docs/).
Feel free to read through [the code in the Github organization](https://github.com/wimpy). Contributions are really welcomed!

<iframe src="https://docs.google.com/presentation/d/1vywHZrOgDfkpKeE_AaUQ5M9ZiJ1uspaDYwKGjxq99ZE/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="299" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Acknowledgements
Finally, I'd like to thank all the people that have helped me during the development of this project. This would've never happened without them, specially:

- [Ismael Fernández](https://github.com/ismFerDev)
- [Alberto Ramírez](https://github.com/aramirez-es)
- [Víctor Caldentey](https://github.com/victuxbb)
- [Fabrizio Di Napoli](https://github.com/Hyunk3l)
