# Deploying Front end projects to Cloudfront
---

This guideline relates to front end projects created using [Angular Material Generator](https://github.com/team-avesta/generator-angm). Also this guideline assumes that we are not entirely hosting the website on a S3 bucket using cloudfront but instead we use a http server to serve index.html and all the static assets inside index.html point to cloudfront. Hence the http server doesn't not have to bear the load of all those resources and instead they are served from cloudfront. The reason for not going entirely on s3 and cloudfront is because custom domain ssl certs on cloudfront are too expensive at the moment. Hence our  approach cuts down on costs.

Before preparing the app for deployment we need to build and generate production ready minified and unglified code. Also we need to replace the paths of static assets everywhere to point them to cloudfront. The project generated through yeoman generator has two tasks that do both the jobs for us.

```
grunt build
```
Running grunt build  will generate a production ready build in /dist folder of your app directory. But the paths of static assets are still relative and do not point to cloudfront.
```
grunt cdnProduction
```
Running above command will scan the /dist directory and replace all the neccessary paths to static files and point modify them to point to cloudfront. There is a variable `cdnUrl` in grunt file which has the value of cloudfront url. All the paths will be prefixed with the value supplied in `cdnUrl`. It changes urls in following places

- script tags in index.html
- link tags in index.html
- src attribute in <img> tags in html files
- image urls in background and background-image css properties
- templateUrls in UI router state configurations

Make sure to have *https* in `cdnUrl` value as we normally tell cloudfront to contact our origin server to fetch our source files over https only. So when cloudfront will in first attempt try to access our file and if it is over http then as it won't have it initally in the edge cache it will contact origin server and by default it will try to access the file using the same protocol as the one recieved from client. Hence it will try to get file over http and origin server which will not be permitted as cloudfront was told to use https only by us.

Hence always use https protocol in cloudfront urls. Also make sure that if your origin is an S3 website (s3 bucket set to be served as a static site) and if you have set behaviour settings in cloudfront to allow cloudfront to access s3 only on https then it will not work as s3 cannot serve https requests in static site mode.

