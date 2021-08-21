---
title: "Jenkins Pull Request Diff"
date: 2021-08-15T10:27:09Z
draft: false
---

## Introduction

By default when you have a multibranch pipeline in Jenkins that runs "Change Jobs" (i.e. jobs against Pull Requests) you'll have a strange error if you try to do a diff against any branch other than your own.

{{<highlight shell>}}
$ git diff --name-only HEAD main
fatal: ambiguous argument 'main': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
{{</highlight>}}

This poses a few questions.

1. Why would you want to diff inside a Jenkins job?
2. Why do we get this error when it 'works fine on my machine'?
3. How do we fix it?

### Why would you want to to run a diff inside a Jenkins job?

I've had two good reasons to need to be able to run diffs inside Jenkins:

1. Determining whether it's worth running tests based on whether the files in a particular subfolder have changed or not
2. [SonarQube's `Reference Branch`](https://docs.sonarqube.org/latest/project-administration/new-code-period/) New Code definition

### Why do we get this error?

We get this error because by default Jenkins only fetches a git refspec for the branch in question.

> A refspec tells git how to map references from a remote to the local repo.  
> *\- Mark Adelsberger - [StackOverflow](https://stackoverflow.com/questions/44333437/git-what-is-refspec)*

In the case of a Jenkins PR triggered job it might look like this:

```
+refs/pull/1/head:refs/remotes/origin/PR-1
```

Everything before the `:` is the source pattern, and everything after is the remote pattern (notice the `origin` nestled in there)

This (in contrast to a *default* job refspec) grabs only the references needed for our pull request.
A [default respec](https://plugins.jenkins.io/git/) for reference looks like:
```
+refs/heads/*:refs/remotes/origin/ 
```

So that means that when inside your PR job you try to do a diff against `main`, git is going to respond with "Sorry buddy, no idea what an `origin/main` is, best I can do is `origin/PR-1`".

## How do we fix it?

You add a refspec to the Jenkins Job's Branch Sources Configuration.
1. `Add` (underneath Behaviours, not `Add Property` underneath Property Strategy nor `Add Source` under Branches Sources)
2. Specific `refspec`.
3. Accept the default of `+refs/heads/*:refs/remotes/@{remote}/*`
3. (Optional but SO important if you don't have ephemeral agents) Add > Wipe out repository and force clone

![Specify Ref Spec inside Jenkins Job's Branch Sources configuration](/images/2021/08/jenkins-refspec.png)

### Running Jenkins Locally

If you don't have a Jenkins server to hand you can spin one up in Docker with:

{{<highlight shell>}}
$ docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
{{</highlight>}}

If you then open `http://localhost:8080` in your browser you'll be presented with a setup wizard and a few defaults to run through and accept.




git fetch --no-tags --force --progress -- https://github.com/Sam-Martin/jenkins-pr-diff-example.git +refs/pull/1/head:refs/remotes/origin/PR-1 +refs/heads/main:refs/remotes/origin/main # timeout=10
Merging remotes/origin/main commit ca00b8c141567d0ea52ab5540823f04bf8d72fac into PR head commit 5061dd1f2c60fd849f7eca7762c32b9f9126fc63


## Further Reading 

* https://support.cloudbees.com/hc/en-us/articles/226122247-How-to-customize-Checkout-for-Pipeline-Multibranch
* https://issues.jenkins.io/browse/JENKINS-62562
* https://stackoverflow.com/questions/37864542/jenkins-pipeline-notserializableexception-groovy-json-internal-lazymap 
