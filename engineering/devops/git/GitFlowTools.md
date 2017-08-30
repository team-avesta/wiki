# Tools used together with Git flow Branching model
---

When following the git workflow model described in [this](https://github.com/team-avesta/wiki/blob/master/engineering/devops/git/GitWorkflow.md) article there are a couple of tools that complement you in your daily workflow and automates few of the common commands that you would have to type out manually when working otherwise.

For example when creating a feature branch from development you need to perform following steps which are ideally required.

* Pull latest code from development branch (so that you don't start new feature on stale code)
* Checkout and create a new feature branch following specific naming convention
* Switch to the newly created feature branch from development

You can perform these steps manually but this kind of gets in your way during your daily activities. You need to take care that whenever you perform any git branching operation, you have followed all the requisite steps and executed them in order. Hence in order to eliminate routine commands which are fairly common we use [git-flow](https://github.com/nvie/gitflow). This plugins automates all the commands we require to run during any branching operations and presents you a summary of the commands it runs on your behalf informing you of what all it does behind the scenes.

The other tool recommended to use is [git-flow-hooks](https://github.com/jaspernbrouwer/git-flow-hooks). You will need to take care of the version numbers when creating release or hotfix branches. Ideally  version numbers should be maintained automatically as less the manual intervention, less the errors. Git flow hooks does this job for you. It provides us with hooks on release/hotfix start and end and automatically reads and bumps version numbers according to the branch.

So let's say you create a first release branch, the hooks will look at the git tags to find the version to bump. If no tags are found, it looks for the version-file. If that isn't found, it assumes the current version is 0.0.0. Version file here is a simple text file which will just contain a version
number written inside it and it does not have a file type, just the name VERSION.

Just a note - in the setting up instructions below we use a [fork](https://github.com/team-avesta/git-flow-hooks/tree/notify) of the [original repo](https://github.com/jaspernbrouwer/git-flow-hooks) as we need the feature which sends updates to slack channels about all the operations being performed on the repo.

### How to set up

Before setting this up you need to have git flow enabled in your repository. Once done clone the repo at any location on your disk using following command
```
git clone https://github.com/team-avesta/git-flow-hooks.git
git checkout notify #the latest code is in notify branch
```

Once cloning is done, we go back to our repo. Now to tell git flow to use hooks provided by the repo cloned above, run below command
```
git config gitflow.path.hooks /path/to/git-flow-hooks
```

```
git config gitflow.hotfix.finish.message "Hotfix %tag%"
git config gitflow.release.finish.message "Release %tag%"
```


Copy the file /path/to/git-flow-hooks/modules/git-flow-hooks-config.sh.dist to /path/to/git-flow-hooks/git-flow-hooks-config.sh and add the Slack Channel url to send messages to slack.

(vb)
