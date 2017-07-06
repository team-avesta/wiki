# Git workflow model
---

This article describes the git branching workflow that we follow and suits our need. Its not something we have thought of from scratch but is instead mostly adapted from [this blog post](http://nvie.com/posts/a-successful-git-branching-model/). Please read the blog post first as this article just makes some ammendments to suit our needs and does not describe everything in detail.


![git branching model](git-model.png)
image copyrights [nvie.com](http://nvie.com/posts/a-successful-git-branching-model/)

At core we have 2 branches :
* master
* development

Whenever any project's repo is initialized in git we will have master branch by default when cloned into local env. Before creating any files we should first checkout a development branch. Project structure should be then initialized into develop branch. After that is done, development branch should be pushed to origin (upstream repo) so that a new development branch is created upstream as well. Up stream here means the server that hosts your git repo. It can be Github, Bitbucket, Aws Codecommit or any git repo hosting service running on your local server.

Master branch will be empty uptill now and that's fine. All development work should be carried out on development and other branches which are discussed below. **Never make a push directly into master branch at any point of time**. Master is meant to contain only production ready code at all times.

### Supporting branches

* Feature branches
* Release branches
* Hotfix branches

##### Feature branch
------

Naming convention  - **feature-*** (example feature-usersCrud)

Checkout from - **development**

If you are working on a particular feature such as ex User's CRUD then instead of working directly on development branch, checkout the latest development branch and create a local branch in your machine and work in that branch. Reason for doing this is that this gives you the freedom to pause your current work (i.e. users CRUD) by commiting your code at any point of time when required and work on some other priority task in other branch or say a urgent bugfix. Without this you would have to take backup of your code or manually perform git stash and stuff like that to temporary switch from your dirty code.

Feature branches will most of the time be created locally only and we will not push them to origin unless there are multiple people involved with interlinked dependencies. In that case you will have to push your feature branch to origin and then the other person can pull that branch from origin and two of you can work together on that feature.

Once the feature development is completed, feature branch should be then merged into development and then consequently deleted from origin and local.

**Note** - for minor changes or bug fixes we can directly work on development branch as well.


##### Release branch
------
This branch should be created when the current stage of development branch is ready to be released for testing.

Naming convention  - **release-*** (example release-0.1.0)

Checkout from - **development**

Once created it will be released to testers for testing. Bug fixes for bugs reported during testing should be performed on this branch only as these bugs belong to 0.1.0. Once all bugs are fixed we merge the release branch into **master** and apply git tag declaring 0.1.0 release. This means we now have 0.1.0 onto production.

Also in order incorporate bug fixes of 0.1.0 into newer releases we need to merge release-0.1.0 into development branch as well.

##### Hotfix branch
------

This branch should be created if and only if there arises a situation when you need to fix a bug urgently in production and either there is no upcoming release in pipeline or if it is, it's not ready to be merged into master.

Naming convention  - **hotfix-*** (example hotfix-0.1.1)

Checkout from - **master**

Hotfix will be merged directly into master branch and pushed to prodution, changing version number of the project and applying git tag along the way.

If a release branch exists at the moment hotfix branch is merged into master, then merge hotfix into that release branch or else merge it back to development branch.

##### Few things to note
------
* Always use grunt tasks and not git GUI for performing git operations described in each projects readme
* Use GUI for git in case of merge conflicts
* Never push directly to master
* Follow naming conventions stritcly

(vb)