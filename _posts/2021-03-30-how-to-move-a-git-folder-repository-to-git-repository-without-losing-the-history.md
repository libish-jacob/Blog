---
layout: post
modified: '2021-03-30 16:38 +0530'
published: true
comments: true
title: >-
  How to move a git folder repository to git repository without losing the
  history.
categories:
  - Source control
description: >-
  Here in the scenario, what I have was a folder repository where users will
  commit the changes. Users will treat the folder which is in a network location
  and will commit the changes to this repository. Now I want this to be moved to
  a bit bucket server so that I can use the proper process to get the changes
  merged. Here the challenge is to retain the history of changes without losing
  it while porting it. Let’s see how this can be achieved.
date: '2021-03-30 16:38 +0530'
---
Here in the scenario, what I have was a folder repository where users will commit the changes. Users will treat the folder which is in a network location and will commit the changes to this repository.
Now I want this to be moved to a bit bucket server so that I can use the proper process to get the changes merged. Here the challenge is to retain the history of changes without losing it while porting it. Let’s see how this can be achieved.
for this what we need is 

- Existing repository URL
- A new blank repository in bitbucket.
- Source tree(optional)


Create a blank repository in bitbucket. This is the repository where you want to move the folder repository. Now once the repository is created, clone this to local.
Once it is cloned, open the terminal from source tree. [If you don’t have source tree then use a command prompt and cd into the local repository folder].
Now run the below command to add a remote to the repository.

git remote add -f old_repo OLD-FOLDER-REPO-URL

here old_repo is the name give to the remote. OLD-FOLDER-REPO-URL is the folder path to the .git folder(for folder repository. e.g. W:\Folder\.git). Make sure that the path is escaped of special characters. This includes the back slash such that the path will look like as in example W:\\Folder\\.git
The command will take some time to execute depending on how big the repository is. In case if you want to remove the new remote, then use command "git remote -v" to list all the remotes and use "git remote rm old_repo" to delete it.

![Capture1.JPG]({{site.baseurl}}/images/Capture1.JPG)


Now use the below command to merge the desired branch from old repository to new one by running the below command. Here the BRANCH-NAME should be replaced with the desired branch name in old repository. This will merge the branch to current branch in new repository. Since it is new and blank repository, it will be merged to mater branch. 
git merge old_repo/BRANCH-NAME

Now we are done, we will have to now push the changes to origin master. In Source tree do a push to master. [on command line run git push origin master]

![Capture.JPG]({{site.baseurl}}/images/Capture.JPG)


Now the changes including the history is pushed to server. Now you can manage the permission and control the merges to master via pull requests.



