+++
author = "Sam Martin"
date = 0001-01-01T00:00:00Z
description = ""
draft = false
image = "/images/2019/03/Screenshot-2019-03-10-at-18.12.40.png"
slug = "testing-submodules-commit-path-within-jenkins"
title = "Testing Submodules' Commit  Path within Jenkins Pipeline"

+++

# Submodules?

I've never been a massive fan of submodules as they come with their own set of complexities and surprises, and embed dependency management within your SCM when really that should be the job of your packaging tool. But sometimes they are the right tool for the job when other tools are absent or your deploy is atypical.

I've been using them recently to split out code that needed to be shared across repos but which wasn't available to be distributed via a package repository of any kind. The shared code is available and versioned in one repo, but also present in multiple others simply by referring to the first repo as a 'submodule'.

# What are Submodules

When you create a submodule in your git repo, you are cloning another repo into a subfolder within the first one. The entire commit history of that repo doesn't get pulled into your history however, just a little file called `.gitmodules` which specifies the origin location and the commit id to which the submodule (sub repo) is pinned.

## Example

Let's say we have code in RepoA that we want available in RepoB. Let's

```
cd ~git/repob/
```

and do

```
git submodule add git@github.com/RepoA remote_code
```

This pulls the code from RepoA into RepoB under the folder `remote_code` and creates the `.gitmodules` file which we can then commit in RepoB to pin the current commit in RepoB (the submodule) to be the contents of `remote_code`.

When you want to update to a newer version (commit) of the submodule, you can

```
cd remote_code
```

(where you'll find all the contents of RepoA), do

```
git pull master
```

to get the latest code and then

```
cd ..
```

back up a level into the parent, where RepoA has recognised the new commit that's been pulled into `remote_code` and allow you to commit an updated `.gitmodules` that contains the new commit id!

# Why test?

So far so good right? But what if we want to test some code that hasn't been merged into master in RepoA (remote_code) with the code in RepoB? We

```
cd remote_code
```

do a

```
git checkout feature/my_new_code
```

then go off and write our updated code, do our tests, and commit and push the updated code along with the updated `.gitmodules` file.

But... That code isn't in master, we're now pushing a dependency on unintegrated code from `RepoA` into `RepoB`! We don't even know if this feature branch will get merged! Some poor unsuspecting person might want to use the latest code from RepoA's master branch a few months down the line, and unwittingly break the code we've written that depended the unmerged feature branch!

It's not even obvious that we've pinned our submodule to a commit from an unmerged feature branch when someone does a PR review on our code in RepoB, as `.gitmodules` only shows the commit id, not the branch, so unless our reviewer is super vigilant, they may just assume we've updated to a more recent version of master!

# Jenkins Tests

This is why we test! In Jenkins we need to do 3 things.

1. Clone RepoB
2. Update the submodule (RepoA)
3. Get RepoA's current commit sha
4. Get RepoA's master branch log history
5. Check the current commit sha of RepoA from `remote_code` appears in RepoA's master branch history

## Clone RepoB

Okay you're probably doing this as you're (presumably) integrating this into your existing pipeline, but here's an example.

```
git branch: 'master',
      credentialsId: 'RepoB',
      url: 'git@ithub.com:example/RepoB.git'
```

## Update the submodule

```
withCredentials([sshUserPrivateKey(credentialsId: 'RepoB', keyFileVariable: 'GIT_SSH_KEY')]) {
  sh """
   export GIT_SSH_COMMAND='ssh -i $GIT_SSH_KEY'
    cd build/sut
    git submodule update --init --recursive
  """
}
```

It's actually preferable to do this with the [SSHAgent Plugin](https://wiki.jenkins.io/display/JENKINS/SSH+Agent+Plugin)  _but_ the above will work without additional plugins (which can be hard to get if you don't control the Jenkins box) assuming your SSH key doesn't have a password (which it really should).

## Get RepoA's current commit SHA

```
def getCommitSha(path='remote_code/.git') {
  sh """
    git --git-dir=${path} rev-parse HEAD > .git/current-commit
  """
  return readFile(".git/current-commit").trim()
}
```

The command `git --git-dir=${path} rev-parse HEAD > .git/current-commit` gets the current commit from `${path}` (`remote_code`) and pipes it to `.git/current-commit` as a text file.

This is a Groovy function, so you can just paste it at the top of your pipeline file prior to any `node{}` etc.

## Get RepoA's master commit history

```
def remoteCodeInMaster(){
   sh """
    git --git-dir=remote_code/.git log --format='%H' master > .git/remote_code_master_log
   """
   def remoteCodeCommit = readFile(".git/remote_code_master_log").trim()
   return remoteCodeCommit.contains(getCommitSha('build/sut/remote_code/.git'))
}
```

The command: `git --git-dir=remote_code/.git log --format='%H' master` prints out a list of commit SHAs for the branch master in the `remote_code` folder.

## Final Step: Check the current commit SHA in remote_code is one that's in the master branch

```
if(remoteCodeInMaster()){
  echo("${getCommitSha()} found in remote_code and it's part of the Master branch!")
}else{
  error("remote_code is pinned to a commit ('${getCommitSha()}') which is not in the master branch of RepoA")
}
```

This goes inside one of your steps and calls the previous two methods we defined in order to fail the Jenkins job if the current commit SHA in RepoA (`remote_code`) is one that's in the master branch.

# Putting it all together

The above all put together in one job looks like so:

```
def getCommitSha(path='remote_code/.git') {
  sh """
    git --git-dir=${path} rev-parse HEAD > .git/current-commit
  """
  return readFile(".git/current-commit").trim()
}

def remoteCodeInMaster(){
   sh """
    git --git-dir=remote_code/.git log --format='%H' master > .git/remote_code_master_log
   """
   def remoteCodeCommit = readFile(".git/remote_code_master_log").trim()
   return remoteCodeCommit.contains(getCommitSha('build/sut/remote_code/.git'))
}

node {
  stage('Preparation'){
    git branch: 'master',
      credentialsId: 'RepoB',
      url: 'git@ithub.com:example/RepoB.git'
    // Update submodules
    withCredentials([sshUserPrivateKey(credentialsId: 'RepoB', keyFileVariable: 'GIT_SSH_KEY')]) {
      sh """
       export GIT_SSH_COMMAND='ssh -i $GIT_SSH_KEY'
        cd build/sut
        git submodule update --init --recursive
      """
    }
  }
  stage('Test') {
    if(remoteCodeInMaster()){
      echo("${getCommitSha()} found in remote_code and it's part of the Master branch!")
    }else{
      error("remote_code is pinned to a commit ('${getCommitSha()}') which is not in the master branch of RepoA")
    }
  }
}
```



