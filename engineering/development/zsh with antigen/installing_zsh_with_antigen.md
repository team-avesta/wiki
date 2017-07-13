Instructions to set up Zsh as your terminal along with antigen plugin manager
===========================

Install Instructions (Linux/Ubuntu)

Step 1: If you have oh-my-zsh installed already the we need to uninstall it as antigen uses it under the hood hence we don't require standalone installation anymore. Use below command to unistall oh-my-zsh. Carry on to next step directly if you don't have oh-my-zsh installed.
```
$ uninstall_oh_my_zsh
```
Step 2: Install zsh (linux/ubuntu)
```
$ sudo apt-get update
$ sudo apt-get install zsh
```
Step 3: Install antigen plugin manager (linux/ubuntu):
```
$ curl -L git.io/antigen > $HOME/antigen.zsh
```
Step 4: modify .zshrc file located inside your /home path in ubuntu and paste the following code in it:
```
source $HOME/antigen.zsh

# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Bundles from the default repo (robbyrussell's oh-my-zsh).
antigen bundle git
antigen bundle npm

#provides autosuggestions bundle.
antigen bundle zsh-users/zsh-autosuggestions

antigen bundle zsh-users/zsh-completions
antigen bundle git-extras

#provides commands for git flow
antigen bundle git-flow

# Syntax highlighting bundle.
antigen bundle zsh-users/zsh-syntax-highlighting

# Load the theme.
antigen theme robbyrussell

# Tell Antigen that you're done.
antigen apply

# If you come from bash you might have to change your $PATH.
#export PATH=/home/your_user_name/.npm-packages/bin:$PATH

#if you use nvm for different node version installation then keep the commands related to nvm intact if they are already #present in .zshrc. Uncomment below lines.
#export NVM_DIR="/home/your_user_name/.nvm"
#[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```
Step 5: Restart your terminal and let antigen install all the plugins and required packages

Step 6: Install git-flow (linux/ubuntu)
```
$ sudo apt-get install git-flow
```

After completing above instructions, your terminal should show all the command suggestions and autocomplete features.

Useful links:
* [Git aliases](https://github.com/robbyrussell/oh-my-zsh/wiki/Cheatsheet#git)
* [Git flow commands](https://github.com/nvie/gitflow#initialization)
