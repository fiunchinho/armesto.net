---
author: fiunchinho
categories:
- Code
date: 2018-12-05T23:47:55Z
tags:
- development
- kubernetes
- jenkins-x
title: Jenkins X Environments
url: /jenkins-x-environments/
thumbnail: "images/jenkins-x.png"
---

This year Jenkins X has been announced. If you haven't heard, Jenkins X is a tool that lets you automate the whole delivery process of your software to a Kubernetes cluster.
One key aspect of Jenkins X is that the deployment of applications is implemented following the [GitOps flow](https://www.weave.works/blog/what-is-gitops-really).
In this approach every environment is represented by a Git repository. In this post we'll see how Jenkins X environments work. 

<!--more-->

## Environments
An installation of Jenkins X consists of:

* A `Development Environment` which is a kubernetes namespace running tools like Jenkins, Nexus, etc. Each different team within an organization is meant to have its own development environment so that they can be as independent from each other as possible.
* Other `Permanent Environments` which are kubernetes namespaces where our applications will be deployed into. By default `Staging` and `Production` are created. Each team can have as many Permanent Environments as they wish and call them whatever they like.
* Optional `Preview Environments`, which are kubernetes namespaces where our applications will be deployed, just like Permanent Environments, but these are created on a PR basis, before you even merge your PR changes into master.

Under the covers this is implemented creating a custom Kubernetes resource `jenkins.io/v1/Environment`. 
Assuming that each team has a Kubernetes namespace assigned, each team will normally have a Development Environment and several different Permanent Environments. 
Each of these environments is represented by a `jenkins.io/v1/Environment` object on the namespace assigned to that team.
 
Teams are also represented by a `jenkins.io/v1/Team` object, but we will cover that on a different post.

These `Environment` objects contain configuration about which Git repository is associated with it, and which git branch to use.

There could be one namespace which is where all apps run across all teams - though from a kubernetes RBAC perspective, if you are working in microservices teams, it's better for each team to manage their own microservices in their own production environment and use service linking between teams.

### Development Environment
A `jenkins.io/v1/Environment` object created on the team’s namespace, which `spec.Kind` field is `Dev`. 
This environment is not linked to a Github repository, we will never promote changes here. 
It’s just a Kubernetes namespace used to run applications while developing, thanks to [skaffold](https://github.com/GoogleContainerTools/skaffold).

In the dev environment Jenkins X installs a number of core applications they believe are required at a minimum to start folks off with CI/CD on Kubernetes. They come with configuration that wires these services together meaning everything works together straight away.

* Jenkins — provides both Continuous Integration and Continuous Delivery automation.
* Nexus — acts as a dependency cache for applications to dramatically improve build times.
* Docker registry — an in cluster Docker registry where the pipelines push application images.
* Chartmuseum — a registry for publishing Helm charts.
* Monocular — a UI used for discovering and running Helm charts.

### Permanent Environments
These environments are where the applications will ran. By default `Staging` and `Production` are created, but more environments can be created. `Staging` has the Auto promote strategy and `Production` the Manual promote strategy. We'll see what that means in a minute. 

These environment are linked to

* A GitHub repository containing a Helm chart that defines which application charts are to be installed on the environment, which versions of them and any environment specific configuration and additional resources (e.g. Secrets or operational applications like Prometheus etc). 
* A Kubernetes namespace where the Helm chart from the repository will be installed.

When you want to deploy to an environment, a Pull Request must be made to that environment’s repository. 
We can manually create the PR to deploy an application to an environment, or we can use this handy `jx` command 

```
jx promote --app myapp --version 1.2.3 --env production
```

Typically we specify the environment while promoting to environments with the Manual promote strategy, like the `Production` environment.
Instead of specifying the environment, we can create a Pull Request in all the environments having the Auto promote strategy using this `jx` command

```
jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION) --no-wait
```

Changing the `Production` environment's promote strategy from Manual to Auto you can do Continuous Deployment.
 
When these PRs are merged into the environment's git repository, the environment pipeline runs, which then applies the Helm chart living on that repo to the environment's Kubernetes namespace.

### Preview Environments
Preview Environments are similar to Permanent Environments in that they are defined in a source code Git repository using Helm charts.
The main difference is that Preview Environments are configured inside the application repository in the ./chart/preview folder.
Also they are not permanent but created when a Pull Request is made to an application's git repository, and then deleted some time after (manually or via automatic garbage collection). 

When the Preview Environment is up and running Jenkins X will comment on your Pull Request with a link so in one click your team members can try out the preview. 

You can create a preview environments using

```
jx preview
```

This will create the `Environment` and `Namespace` objects for this environment. 
It will also build the application creating a new Docker image that will be deployed to the preview environment using the preview chart defined in the ./chart/preview folder.

## Managing environments
You can create new environments using jx

```
jx create env --name prod --label Production --namespace my-prod
```

Some interesting options are

* label: The Environment label which is a descriptive string like `Production` or `Staging`.
* prefix: Environment repo prefix, your Git repo will be of the form `environment-$prefix-$envName`.
* promotion: The promotion strategy, Auto or Manual.

You can then list all existing environments

```
jx get environments
```

Or edit an existing environment (-b for no interactive)

```
jx edit env -b --name prod --label Production --no-gitops --namespace my-prod
```

Or finally delete it

```
jx delete environment prod
```


## Deployment Flow 
In the workflow that Jenkins X proposes, we will have a repository for each application that we want to deploy. 
But also we will have a repository for each environment where you want to deploy these applications.
The repositories for the different environments describe all the applications that must be running on them. 

So while developing your application, the usual flow would be

* Create a Pull Request on the application repository: this will execute the application CI pipeline to build and test the application.
* Once the Pull Request on the application repository is merged, the application pipeline will release this newly built version to a Docker Registry and promote it to the environments with the Auto promote strategy using a `jx` command.
* If we want to deploy to production or any environment with the Manual promotion strategy, we need to execute the `jx` command targeting the desired environment.