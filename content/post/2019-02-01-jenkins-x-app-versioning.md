---
author: fiunchinho
categories:
- Code
date: 2019-02-01T23:47:55Z
tags:
- development
- kubernetes
- jenkins-x
title: Automatically versioning your Application on Jenkins X
url: /automatically-versioning-your-application-on-jenkins-x/
thumbnail: "images/jenkins-x.png"
---

Having a good versioning strategy for our applications is key. 
Specially in Jenkins X, which follows the [GitOps flow](https://www.weave.works/blog/what-is-gitops-really) to deploy our applications, specifying which version of our application will be used on each environment.

But having to manually create tags or releases for our applications can be a tedious task. Jenkins X automatically takes care of versioning for us. 
It uses a tool called [jx-release-version](https://github.com/jenkins-x/jx-release-version) to figure out which is the next version to be released.
In order to do that it checks what's the current released version in the repository looking at the released Git tags. It can also read this version from your `pom.xml` file, or your `Makefile`.

If we use [semver semantics](https://semver.org/), and versions are written in the format major.minor.patch, jx-release-version will tell you which is the next patch version.

The cool thing is that you don't need to use Jenkins X to use [jx-release-version](https://github.com/jenkins-x/jx-release-version)!

Let's go through some examples.

<!--more-->

# Using git tags
Using git tags is probably the easiest way to handle our application versions. We can create a new git tag for every new version that we want to release.

If we try to use `jx-release-version` on a Git repository that has no tags, it will return that the next version number to release is `0.0.1`.
Next time we use `jx-release-version`, it will increase our patch number.
```bash
$ git --no-pager tag -l
$ # there are no tags just yet!
$ RELEASE_VERSION=`jx-release-version` && git tag -fa v${RELEASE_VERSION} -m 'Release version ${RELEASE_VERSION}'
$ git --no-pager tag -l
v0.0.1
$ RELEASE_VERSION=`jx-release-version` && git tag -fa v${RELEASE_VERSION} -m 'Release version ${RELEASE_VERSION}'
$ git --no-pager tag -l
  v0.0.1
  v0.0.2
```

# Using maven pom file
If you are using a `pom.xml` file that also tracks your current application version, you still can use `jx-release-version`. 
It will try to sync the git tags in your Git repository with the version that you specify in the `pom.xml` file.

Your release process needs to look something like this

```bash
# First we call the `jx-release-version` binary that will return the next version number to be released.
# Every time we call the binary it will try to figure out the next version number. Normally it's not a good idea to call it more than once.
RELEASE_VERSION=`jx-release-version`
echo "New release version ${RELEASE_VERSION}

# We update our current pom.xml file with this new version number.
mvn versions:set -DnewVersion=${RELEASE_VERSION}

# Changes to the pom.xml file need to be committed to our repository.
git commit -a -m 'release ${RELEASE_VERSION}'

# The git commit containing both our application changes and the change to the pom.xml file will be tagged using the same version number.
git tag -fa v${RELEASE_VERSION} -m 'Release version ${RELEASE_VERSION}'

# Push the commit and tag to the remote repository.
git push origin v${RELEASE_VERSION}
```

If we start a new git repository that has no tags, the `jx-release-version` will use the version in our `pom.xml` file.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.example</groupId>
    <artifactId>example</artifactId>
    <version>1.0-0-SNAPSHOT</version>
    <packaging>pom</packaging>
</project>
```

The release version for `1.0.0-SNAPSHOT` is `1.0.0`, so `jx-release-version` will return `1.0.0` as the next version number to use.
Similarly, if our `pom.xml` file had `<version>0.0-23-SNAPSHOT</version>` in it, `jx-release-version` would return `0.0.23` as the next version number to use.


# Using a Makefile
Let's say we have this Makefile that tracks the current version of our application

```bash
# This is the current version of your application
VERSION := 2.0.3-SNAPSHOT

# Use jx-release-version to calculate next version
RELEASE_VERSION := $(shell jx-release-version)

build:
	# The git commit containing our application changes will be tagged.
	git tag -fa v${RELEASE_VERSION} -m 'Release version ${RELEASE_VERSION}'

	# Push the tag to the remote repository.
	git push origin v${RELEASE_VERSION}

```

The current version of our application is `2.0.3-SNAPSHOT`. 
That means that the `jx-release-version` will return `2.0.3` as the next version number to use.


# Releasing a new major/minor version
Sometimes we don't want to just release a new patch version (like going from `1.0.5` to `1.0.6`). Instead, we want to release `1.1.0`, or even `2.0.0`.
We have said earlier that jx-release-version calculates the next version number based on the current Git tag, or current specified version on `pom.xml`/`Makefile`.
So if we need to release a new major/minor version, we just have to release a new Git tag or update the `pom.xml`/`Makefile`.

For example, if the current version is `0.0.2` and we want to release `0.1.0`, we first create a git tag for that and let `jx-release-version` do it's thing afterwards.

```bash
# Manually create new tag for the version that we want
git tag -fa v0.1.0 -m "Release version 0.1.0"
# Use jx-release version normally
$ RELEASE_VERSION=`jx-release-version` && git tag -fa v${RELEASE_VERSION} -m 'Release version ${RELEASE_VERSION}'
$ git --no-pager tag -l
v0.0.1
v0.0.2
v0.1.0
v0.1.1
```

As said in the [project's README](https://github.com/jenkins-x/jx-release-version)

- If your project is new or has no existing git tags then running `jx-release-version` will return a default version of `0.0.1`
- If your latest git tag is `1.2.3` and you Makefile or pom.xml is `1.2.0-SNAPSHOT` then `jx-release-version` will return `1.2.4`
- If your latest git tag is `1.2.3` and your Makefile or pom.xml is `2.0.0` then `jx-release-version` will return `2.0.0`
