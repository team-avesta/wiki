# wiki
This repository contains articles &amp; useful information about technologies what we are using. #TeamAvesta-KnowledgeBase



#### Table of Content

* [engineering][] -- All the engineering related topics like devops, sysops, development will go here.
* [training][] -- Training related meterials will go here.
* [links][] -- Links about some useful articles, blogs will go here.

[engineering]: https://github.com/team-avesta/wiki/blob/master/engineering/readme.md
[training]: https://github.com/team-avesta/wiki/blob/master/training/README.md
[links]: https://github.com/team-avesta/wiki/blob/master/links/readme.md


## How to Contribute

Step 1: Clone avesta/wiki repository to your local machine.

```
$ git clone https://github.com/team-avesta/wiki.git
```
Step 2: Install Sublime plugin for Markdown Preview Using Package Control.

 1. [Install](https://packagecontrol.io/installation) Package Control if you haven't yet.
 2. Use <kbd>control</kbd>+<kbd>shift</kbd>+<kbd>P</kbd> then `Package Control: Install Package`
 3. Look for [Markdown Preview](https://github.com/revolunet/sublimetext-markdown-preview) and install it.
 4. You can use it to preview of your post. For showing preview of your article, open that file into sublime text and follow given steps.

	 - optionally select some of your markdown for conversion
	 - use <kbd>control</kbd>+<kbd>shift</kbd>+<kbd>P</kbd> then `Markdown Preview` to show the follow commands (you will be prompted to select which parser you prefer):
		- Markdown Preview: Preview in Browser
		- Markdown Preview: Export HTML in Sublime Text
		- Markdown Preview: Copy to Clipboard
		- Markdown Preview: Open Markdown Cheat sheet


Step 3: Make changes on existing topic or write new topic.

 - Please create new article into appropriate folder. If you want to write on training related topic, create file into training folder.
 - You have to write in markdown template. Here is the [Markdown-cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

Step 4: After making some changes you can add it to git branch.

```
$ git add .
```

Step 5: Commit your changes to git repo.

```
$ git commit -m "Commit Message"
```

Step 6: Pull new changes from online repo.

```
$ git pull master origin
```

Step 7: Push your changes to online repo.

```
$ git push master origin
```

