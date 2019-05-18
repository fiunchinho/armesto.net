---
author: fiunchinho
categories:
- Code
date: 2019-05-18T23:47:55Z
tags:
- development
- kubernetes
- kind
title: Continuous Integrating my Kubernetes controller using Kind
url: /continuous-integrating-my-kubernetes-controller-using-kind/
thumbnail: "images/kind.png"
---

[Kubernetes controllers](https://stackoverflow.com/questions/47848258/kubernetes-controller-vs-kubernetes-operator/47857073#47857073) are processes that react to changes in the state of some objects saved on the Kubernetes API. 
They do this by watching the Kubernetes API so that they get notified whenever the state changes.

But how do we know if our Kubernetes controller is doing what's supposed to do? 
We can write [unit tests](https://codeclimate.com/github/fiunchinho/iam-role-annotator) that test our logic in isolation to get some confidence, but it would be great if we could add some [end to end tests](https://martinfowler.com/articles/practical-test-pyramid.html) using the real Kubernetes API.

However, deploying Kubernetes is not an easy task. And even if we managed to accomplish it, it would take a long time to deploy a Kubernetes cluster so we could test every change to our code base.

In this post, I'm going to explain how I used [kind](https://kind.sigs.k8s.io/) to test that [my controller](https://github.com/fiunchinho/iam-role-annotator) is working as expected, using a real Kubernetes API.

<!--more-->

## Creating the cluster using kind
[My Kubernetes controller](https://github.com/fiunchinho/iam-role-annotator) is pretty simple. Whenever a Deployment object is created containing an specific annotation, it will add an annotation to the Pods belonging to that Deployment.
I created this controller because we needed a way for developers to specify that the application that they were deploying required AWS IAM credentials to use AWS services, but we didn't want developers to have to know things like the AWS account id to use.
By using this controller, the Kubernetes cluster administrators configure which AWS Account to use on every Kubernetes namespace. And developers just indicate whether their application requires IAM authentication or not, by adding an annotation. Simple.

Anyway, I wanted to create an end to end test that would do the following:

- Create a Kubernetes cluster
- Install the controller using the version that we are testing
- Create a Deployment that contains the annotation that will trigger the controller
- Test that the Pods of the Deployment contain the IAM annotation

For the Kubernetes cluster I used [kind](https://kind.sigs.k8s.io/), a recent project that let's you deploy a Kubernetes cluster using only a Docker container. Obviously, it's not meant to be used on production, but it's perfect for testing purposes. To create a cluster with kind, you just need to

```
$ kind create cluster
``` 

And that's it! It's that simple. We can see the cluster is running inside a single container
```
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                  NAMES
c27f2c8fa39e        kindest/node:v1.14.1   "/usr/local/bin/entr…"   22 minutes ago      Up 22 minutes       63711/tcp, 127.0.0.1:63711->6443/tcp   kind-control-plane
```

Now you just need to make `kubectl` send requests to that cluster, using the command

```
$ export KUBECONFIG="$(kind get kubeconfig-path)"
```

I'm also creating a new Namespace where I'll run all the things required for this test. This is not important during Continuous Integration (CI), where everything will be deleted when the CI job is finished. 
But I do it this way because I can then easily run the same test locally on other Kubernetes clusters like Minikube, while not on CI. 
When the test is done, I just remove the namespace to leave no traces of the test execution.


## Installing my controller
Now I need to install the piece of code that I want to test: my controller. The controller repository contains a [Helm](https://helm.sh/) chart to make it easy to install it, so I'll just use it. That way, I'm also testing whether or not the Helm chart is working as expected.

```
$ helm upgrade --tiller-namespace ${NAMESPACE} --namespace "${NAMESPACE}" --wait --install "iam-role-annotator" "./charts/iam-role-annotator" --set image.tag="${TRAVIS_COMMIT:-latest}" --set awsAccountId="${AWS_ACCOUNT_ID}"
```

The value for the `AWS_ACCOUNT_ID` variable is important. It's the same AWS Account Id that needs to be added as annotation on applications.
Notice that I passed the `--wait` argument to Helm. This will block the command until my controller is running and ready. If it fails to start for whatever reason, the test would automatically fail.

## Create annotated deploy and check the results
My controller is ready and waiting for applications that contain the right annotation.
Creating a simple hello world application containing that annotation is enough to trigger my controller. Let's use a simple nginx deployment
```
$ cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: ${NAMESPACE}
  labels:
    app: nginx
  annotations:
    armesto.net/iam-role-annotator: "true"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        prometheus.io/scheme: http
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
EOF
```
Notice the important annotation for my controller: `armesto.net/iam-role-annotator: true`.

Now I just need to wait for my controller to react.

Then I can test if the pods belonging to the hello world deployment contain the required annotation. For that I used `jq`, a handly command line tool to inspect json data.

```
POD_NAME=$(kubectl get pods --namespace ${NAMESPACE} --field-selector=status.phase=Running -l "app=nginx" -o jsonpath="{.items[0].metadata.name}")

if [[ $(kubectl get pod --namespace ${NAMESPACE} ${POD_NAME} -o json | jq '.metadata.annotations' | jq 'contains({"iam.amazonaws.com/role"})') == 'true' ]]; then
  if [[ $(kubectl get pods --namespace ${NAMESPACE} ${POD_NAME} -o json | jq -r '.metadata.annotations."iam.amazonaws.com/role"') == "arn:aws:iam::${AWS_ACCOUNT_ID}:role/${DEPLOYMENT_NAME}" ]]; then
    echo "SUCCESS!"
    exit 0
  else
    echo "ERROR: the annotation contains the wrong value"
    kubectl get pod --namespace ${NAMESPACE} ${POD_NAME} -o json | jq '.'
    exit 1
  fi
else
  echo "ERROR: the POD does not contain the expected annotation"
  kubectl get pod --namespace ${NAMESPACE} ${POD_NAME} -o json | jq '.'
  exit 1
fi
```

## Conclusion
Every time I push some changes [to this controller](https://github.com/fiunchinho/iam-role-annotator), the CI pipeline kicks in, and [my end to end test](https://github.com/fiunchinho/iam-role-annotator/blob/master/e2e_test.sh) gets executed creating a Kubernetes cluster thanks to [kind](https://kind.sigs.k8s.io/).

Now I have much more confidence that my changes will work as expected when deploying the controller to our production clusters, and that makes a huge difference.


