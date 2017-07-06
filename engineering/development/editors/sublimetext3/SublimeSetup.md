Sublime Text 3 setup
===========================

Install Instructions (Linux/Ubuntu)

Step 1: Install the GPG key
```
$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```
Step 2: Select the stable channel as source:
```
$ echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```
Step 3: Update apt sources and install Sublime Text:
```
$ sudo apt-get update
$ sudo apt-get install sublime-text
```

After completing above instructions you will bare metal installation of sublime text working. In order to do normal development we need couple of more plugins that make our life easier in day to day development.

To get started with installing plugins we first need to install [Package Control](https://packagecontrol.io/installation) which is package manager for ST3 much like npm is for Node.js. Go to the site and install Package Control following the instructions given.

After installing Package Control we now can begin installing individual Plugins and packages by searching them. But in our case to make this process easier what we do is we replace the **User** folder in Sublime Text Browse packages directory with a set of files and folders. These files and folders contain a list of plugins and packages which are already setup. Once these files are placed in correct location, restarting ST3 is all you need. It will automatically detect these files and begin installing missing plugins and packages.

Here are the steps for performing above process

**Step 1** : Open User Folder.
```
In ST3 go to Preferences -> Browse Packages -> User
```
Once the file explorer is open, close and exit ST3

**Step 2** : Delete everything inside User. This folder basically contains everything thats specific to your sublime text installation. Your settings, active theme, custom keyboard shortcuts etc etc all are persisted in this folder. We will replace your bare metal settings with already configured settings in next few steps.

**Step 3**: Clone [this](https://github.com/vinaynb/mysublime) repo which contains sublime text settings which mirror to my local ST3 installation.

**Step 4**: Once cloning is completed, copy all files and folders from cloned directory into Preferences -> Browse Packages -> User directory for ST3.

**Step 5**: Restart ST3 and let Sublime do its magic (i.e install missing plugins and packages) !