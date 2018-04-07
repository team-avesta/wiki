# Tips to optimize Angular 1.x apps for performance
---

In most cases performance optimization's are introduced at a later stage when your application starts to hog ton of memory and then eventually crashes after a prolonged use of time.

We had run into a lot of such use cases where we had to dive deep into internals and 3rd party plugins that we had used to squeeze out all the tiny bits and remove memory leaks to make our apps performant. So this article documents all those tips for other developers so that some we can have some of these optimizations already in place when developing the app itself.


* Whenever you choose a 3rd party plugin/module/script/directive etc from anywhere, look into it's documentation and always search for a way to clear things up when that particular thing is no longer in use.
* All $timeout's and $interval's must be cleared in $destroy method of respective component or wherever your are using it. Uncanceled timeouts keep running in the background and prevent the scope and hence DOM from being garbage collected. See example code below:
```js
//clearing a timeout
var promise = $timeout(function() {}, 1000);
// Stop the pending timeout
$timeout.cancel(promise);

//clearing $interval
promise = $interval(setRandomizedCollection, 1000);
// Stop the pending interval
$interval.cancel(promise);
```
* Use ng-if instead of ng-show and ng-hide because ng-show and ng-hide just add a `display:hidden` class to dom elements and do not actually remove them from DOM hence if you have complex UI with lot of DOM nodes, that will eat up uneccessary RAM as user spends more and more time in your application without refreshing the browser. NOTE - there are possible pitfalls too when following this method so choose wisely according to your needs. Refer to [this](http://www.codelord.net/2015/07/28/angular-performance-ng-show-vs-ng-if/) article for more info.
* Disable debugInfoEnable flag before going into production. This flag is enabled by defualt in angular js apps the reason being *AngularJS attaches information about binding and scopes to DOM nodes, and adds CSS classes to data-bound elements*. Refer [this](https://docs.angularjs.org/guide/production) for additional info.
```js
angular.module('app.core.config', [])
.config(['$compileProvider', function ($compileProvider) {
    // disable debug info
    $compileProvider.debugInfoEnabled(false);
}]);
```
* If possible never reference a DOM node from controller or service directly.**All dom related operations should be performed from directive only**. If you have to take a reference then make sure that you are not taking the reference on a global variable in controller. If you have to do that too then make absolutely sure that you set that variable to `null` on destroy event as that will allow the node to be GC'ed.
* Most of the times you will need to listen and emit or broadcast events on angular's $scope variable only. But if you are attaching event handler's in any other way or for a 3rd party, make sure to de-register those event listeners when destroy event gets called. Example code below 
```js
//deregister jquery events registered using element.on()
 $scope.$on("$destroy",function() {
    $( window ).off( "resize.Viewport" );
 });
```
* Have a look any 3rd party's repository on github and look for issues pertaining to memory leaks and thier status before deciding to use it your project.
* Remove all `console.log` statements from your code before going into production as if you have and logs which reference a DOM node or closure which references DOM nodes then the entire dom tree will not be gc'ed as browser needs to keep the reference to that object alive because it is logged in console.
* Use one-time-bindings instead of 2 way binding (which is default in angular) to reduce number of watcher's angularjs has to maintain each time there is a change in your state. Read [this](https://blog.thoughtram.io/angularjs/2014/10/14/exploring-angular-1.3-one-time-bindings.html) to know more in depth about performance implications and technicals behind this step. **NOTE -** one time binding is not suitable for all use cases and are helpful only in those cases where you do not expect the data to change and you don't need to re-render the view. So use wisely !
* Wheneve profiling for memory leaks always perform those tests in icognito or private browsing mode so that other browser extensions or 3rd party plugins do not give false negatives.
* There are/were open issues ([here](https://github.com/angular/angular.js/issues/4864) and [here](https://github.com/angular-ui/ui-router/issues/545) respectively) about ngAnimate and angular-ui-router plugin leaking memory but of late in the latest versions 1.6.9 and 1.0.15 they have been fixed so kindly check out versions of these packages in your applications.


References
____

- Fantatic article (probaly the best we have found) - https://www.dwmkerr.com/fixing-memory-leaks-in-angularjs-applications/
- 

(vb)
