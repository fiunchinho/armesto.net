---
author: fiunchinho
categories:
- Code
date: 2018-12-03T23:47:55Z
tags:
- development
- kubernetes
title: Layered architecture and Kubernetes operators
url: /layered-architecute-and-kubernetes-operators/
thumbnail: "images/kubernetes.png"
---

If you've been developing applications for some time, you've probably developed some kind of HTTP application.
This is the most solved problem there is in our industry, with lots of frameworks and tools that help you solve this problem easily.
Also, there are some patterns and practices that we apply when solving this, that come from years of experience learning the pain points while developing these applications.
I'm talking about things like not coupling your business rules to your framework, or using layers to separate things like data access objects from your domain model.

However, sometimes we forget that we can apply these patterns to almost any kind of software project.
Lately I've been involved in projects that contain some Kubernetes Operators and the code in there could benefit a lot from the patterns that we already apply to our typical HTTP applications.

<!--more-->

### Don't let your domain model depend on your framework
I created [this Kubernetes controller](https://github.com/fiunchinho/iam-role-annotator) some months ago.
There are several different frameworks to help you create your own Kubernetes Operator/Controller, but in this particular case, I decided to try [Kooper](https://github.com/spotahome/kooper).
If I search the keyword "kooper" in the codebase of my controller, there are only two matches: [the dependency file declaring the dependency on the kooper library](https://github.com/fiunchinho/iam-role-annotator/blob/master/Gopkg.toml#L17-L19), and the main file that bootstraps the controller.
All the other files (specially the `pkt/service.go` file containing all the business logic) don't depend on the kooper framework at all.
If I would switch to a different framework, I wouldn't need to change pretty much anything, only the bootstrapping of the process that takes care of all the wiring.
All the important bits, the business logic containing what my controller actually does, that wouldn't need to be rewritten or even touched.

### You are using a database, believe it or not
Kubernetes Operators and Controllers use the Kubernetes API to get the current state of the cluster, and store data on inside Kubernetes resources.
This means that these processes are using etcd as a database, which we normally access through the Kubernetes API using the [client-go library](https://github.com/kubernetes/client-go).

In other kind of applications, we have been creating specific objects that take care of accessing our data for years.
We usually call these objects the data access layer, but somehow we don't do it when accessing the data stored in Kubernetes objects.
It's pretty normal to find [code like this all over a Kubernetes operator or controller](https://github.com/fiunchinho/iam-role-annotator/blob/5d56a9b2801064d4d1d71f5d47cf8b496a4b37de/pkg/service.go#L73-L77)

{{< gist fiunchinho e99adce57a2d676e1d39fa3653a7e78a >}}

We are coupling our business logic to the client-go library that we use to talk to the Kubernetes API.
This makes our Kubernetes operators and controllers hard to test, because when this code is executed, it will try to connect to a real Kubernetes cluster.
[The client-go library offers some tooling](https://godoc.org/k8s.io/client-go/kubernetes/fake) to help you create Fake API's, but wouldn't be better if we wouldn't need to use that to test every part of our application?

By encapsulating the data access parts of your code on its own objects, you could write unit tests for the important bits of your application where the business logic is happening, easily stubbing the data access objects.
We could use [the Repository pattern](https://martinfowler.com/eaaCatalog/repository.html) for this.
[This is a repository to fetch `ConfigMaps`](https://github.com/fiunchinho/dmz-controller/blob/master/repository/configmap.go) from the Kubernetes API. That's the real implementation that my controller will use when it's started.
But [this is the implementation that my controller will use when running the unit tests](https://github.com/fiunchinho/dmz-controller/blob/master/repository/fake_configmap.go).
This fake implementation won't try to connect to a real Kubernetes cluster: it just stores the objects in memory, which is fine to run our tests.

We would still need to write some integration tests, to make sure that everything works well together. But all the complexity of testing your code depending on the client-go library, would only affect your repository objects.

### Dependency injection
In order to achieve what we've been talking about on this post, it's really important that we use [dependency injection](https://martinfowler.com/articles/injection.html) to pass the right objects to our methods.
We need to be able to pass either the real data access objects or the stubbed ones.

Instead of instantiating the objects that your service depends on inside its own functions, declare those dependencies as parameters that need to be passed when creating the service.
This way you can pass the right implementation that you need. On your unit tests, pass the stubbed implementation. On the real bootstrapping of your service, pass the real implementation that talks to the Kubernetes API, [like this](https://github.com/fiunchinho/iam-role-annotator/blob/5d56a9b2801064d4d1d71f5d47cf8b496a4b37de/pkg/service.go#L27-L34)

{{< gist fiunchinho f53e16bc8c9cdfffed8a9275bd288447 >}}

This service doesn't care if the `client` is the real one or the stubbed one.

### Conclusion
At the end of the day, it's just applying what we have already been applying to our applications for years. As Kubernetes controllers and operators get more complex, a better structure for our code is needed to keep the code maintainable.
