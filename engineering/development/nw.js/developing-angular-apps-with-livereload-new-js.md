# Guide to setup your development workflow when working with Nw.js

When working with nw applications it becomes critical to carry out your active development in a desktop app itself and not as a regular web app in a browser. By desktop app means the native app that nw js creates which runs chromium under the hood along with access to node.js.

Question may arise that what difference does it create if I carry out my development in a web browser rather than going through the pain of running dev code in native desktop app if both of them are going to run on the same chromium browser ? Following reasons may convice you to do that

* The environment for both the cases are completely different. As it happened in our case we were using a 3rd party bower package [jsSHA](https://github.com/Caligatio/jsSHA). This package used to work correctly when developing in browser (it exposes jsSha object on global window object) but the same thing didn't work when bundled as a desktop application. Reason was - the author exposed the object on global window object but if node.js env was available (which is true in case of nw desktop app) it exports the object as a js module and hence it is no more available as a property on window object. This was not caught by unit tests and went undetected until our QA team tested the actual desktop app.

* The file loading protocols are different for both the cases. While browser mostly fetches the file from a webserver via HTTP or from disk over `file:///` protocal, nw.js uses `app:///` protocol.

* When making cross domain ajax calls in case of browsers you get CORS errors as browser doesn't allow cross origin requests by default but the same issue doesn't occur in case of nw js apps as it is a browser running inside a native application.

* Its always efficient to have your development environment mimick the eventual production environment because that can give you a heads up on issues that otherwise would have never cropped up.

In order to launch a desktop app with your code in it you need to have the following things:
* `nw` npm module installed globally with following command (there is some issue with running nw v0.25.0 on linux hence i am sticking to 0.24.4 & nwjs_build_type flag is neccessary to allow access to chrome dev tools for debugging):
```bash
npm install -g nw@0.24.4 --nwjs_build_type=sdk
```

* package.json file in your working directory such as below
```json
{
  "name": "FileExplorer",
  "description": "Simple file explorer.",
  "version": "0.1.0",
  "main": "index.html",
  "window": {
    "toolbar": false,
    "width": 660,
    "height": 500
  }
}
```
* index.html file in your working directory

# Using [Angular Generator](https://github.com/team-avesta/generator-angm) with nw.js
-------
For angular applications we use a custom yeoman generator and in order to make this project work with nw we need to make few changes to files that are created by yeoman.

Firstly package.json needs to be modified to contain `main` attribute (`window` attribute is optional). Doing this should be enough to launch a desktop application by running following command in your application directory
```bash
nw
```

In order to enable **live reload** we use [nw-dev](https://www.npmjs.com/package/nw-dev) module. After installing nw through npm we need to manually insert a script in our index.html as mentioned in nw-dev package.
```html
...
<!--live reload inside nw.js desktop app-->
<script src="node_modules/nw-dev/lib/dev.js"></script>
<!-- build:js(.) scripts/scripts.js -->
<!-- injector:js -->
<script src="/app/app.js"></script>
<script src="/app/app.config.js"></script>
...
```

Normally when developing using a browser we use grunt to start our development server. The yeaoman scaffolds a ready to use grunt file which has basic tasks for injecting all source code files and bower_components automatically in index.html, compiling sass etc. We will want to use all this services when developing on desktop app too. Hence i have modified `GruntFile.js` with a task named **serveNw**. Code added to the existing GruntFile.js is as below
```js
...
grunt.registerTask('serveNw', 'serves angular app in nw js desktop env', function() {
		grunt.task.run([
			'clean:server',         //cleans tmp directory
			'wiredep',              //injects bower files in index.html
			'injector:dev',         //injects app source files in index.html
			'concurrent:server',    //runs compass to compile scss into css
			'postcss:server',       //runs auto prefixer over compiled css from above step
			'copy:nw',              //copies application.css from .tmp to and into styles/ directory. This is required as we include application.css from styles folder in index.html
			'exec'                  //runs shell command to open nw app
		]);
	});
...
```

# Limitations on working with Angular Generator at this time
-------

We don't watch the css files for changes right now and hence if there's a change to any sass file it will not be compiled and reflected in desktop app in realtime and we need to re run `grunt serveNw` command.

Also adding any new files in app/ folder or adding a new bower component will not be available in the app directly and re-running grunt task is the only option.

(vb)
