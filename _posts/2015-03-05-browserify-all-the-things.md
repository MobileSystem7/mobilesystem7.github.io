---
layout: post
title: "Browserifying our Backbone app"
author: Chase Courington
tags: ui javascript browserify backbone
category: js
---

A couple months ago [we updated our front-end build process](/blog/post/replacing-rails-asset-pipeline-with-grunt-bower-browserify/) from using the built-in Rails asset pipeline to using [Grunt](http://gruntjs.com/) with [Browserify](https://www.npmjs.com/package/browserify). In this post I'll quickly cover how we've updated our [Backbone](http://backbonejs.org/) app to use Browserify for template pre-compilation and a more modular architecture.

----
## Install and Config

We use [grunt-browserify](https://github.com/jmreidy/grunt-browserify) which gives us a simple integration with Grunt and our ui tasks. We need to install this module to our devDependencies:

```
$ npm install grunt-browserify --save-d
```

We then configure our [browserify.js](https://gist.github.com/courington/a26024fc04fbc9223a20#file-browserify-js) task that gets called in our [gruntfile.js](https://gist.github.com/courington/a26024fc04fbc9223a20#file-gruntfile-js).

In the __browserify.js__ we can define multiple options, one of which is `transform`. We also can pass in many different transform options but we only use one for our [Handlebars template compilation](https://github.com/epeli/node-hbsfy#usage).

```
options: {
    transform: ['hbsfy']
}
```

Define the files for Browserify to start compiling your app. Here we're looking at `cn.home.js` and then following the `require()` chain to create our output file, `home.concat.js`.

```
files: {
    '<%= paths.dist_js %>/dist/home.concat.js' : 'node_modules/app/cn.home.js',
    ...
}
```

----
## Our UI Architecture

We utilize "controllers" in our Backbone app, which are Backbone views that load children and handle some basic event delegation. This helps us to build more modular widgets, partials, mixins and decorators; so our code is comprised of many smaller, interchangable modules.

This architecture also works well with Browserify. Where we were previously using `grunt-contrib-concat` on huge arrays of our modules we now simply `require()` direct dependencies ("modules") and Browserify takes care of the rest. An immediate benefit is with lower maintanence overhead, no longer getting load order bugs from our concat order.

In our example [__cn.home.js__](https://gist.github.com/courington/a26024fc04fbc9223a20#file-cn-home-js) we require just the direct dependencies for this controller.

```
...
require('app/v.widget.date.range.inspection');
require('app/v.widget.home.overview');
 
Controllers.Home = Backbone.View.extend({
    tpl: require('app/t.page.home.hbs'),
...
```

We do the same with other files that have modules they require, [__v.widget.date.range.inspection.js__](https://gist.github.com/courington/a26024fc04fbc9223a20#file-v-widget-date-range-inspection-js).

```
...
require('app/v.partial.popup.js');
require('app/v.popup.container.js');
require('app/v.widget.date.range.js');
 
Views.Widgets.DateRangeInspection = Backbone.View.extend({
    tpl: require('app/t.widget.dateRangeInspection.hbs'),
...
```

Currently we assign our Controllers, Views, Models, etc. to globals, this is something that we carried over and will most likely be phasing out. So where in [v.widget.date.range.inspection.js, line 61](https://gist.github.com/courington/a26024fc04fbc9223a20#file-v-widget-date-range-inspection-js-L61) we have `this.childViews.popup = new Views.PopupWidgetContainer({` which refers to a global that is set in the `require('app/v.popup.container.js');` on line 2. We'll be assigning that `require()` to a local variable, like `var myWidgetContainer = require('app/v.popup.container.js');` and then line 61 would read `this.childViews.popup = new myWidgetContainer({`.

----
## Compile
Browserify enters our controller and starts crawling back through `require()` statements and compiling all our modules into a single output file that will be loaded in the client.

There is one catch, our js architecture somewhat throws Browserify off, with all it's directories and subdirectories. We easily solve this with tasks in our [`grunt-contrib-copy`](https://gist.github.com/courington/a26024fc04fbc9223a20#file-contrib-copy-js) and [`grunt-contrib-clean`](https://gist.github.com/courington/a26024fc04fbc9223a20#file-contrib-clean-js). Our js modules are copied to `/node_modules/app` giving Browserify a clear path to all our javascript modules. Once Browserify has compiled the output to `<%= paths.dist_js %>/dist/` we clean `/node_modules/app`.

Now running `$ grunt prep` calls Browserify as part of our front-end build. It copies our modules from where they can be safely edited to where they'll get compiled. Browserify then crawls our files looking for all `require()` statements and compiles the javascript and Handlebars templates with the modules in `/node_modules/app`, once complete that directory is cleaned.

```
Running "copy:browserify" (copy) task
Copied 577 files

Running "browserify:dist" (browserify) task

Running "clean:browserify" (clean) task
>> 1 path cleaned.

Execution Time (2015-03-05 18:27:27 UTC)
browserify:dist        14.7s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 33%
```

----
## Conclusion

This workflow helps us keep our code small and organized as well as helping us keep from committing any compiled code. With this flow we only commit our modules and anything that is compiled gets ignored helping to avoid merge conflicts.

Refactoring a large Backbone app from relying on concatenation and globals for writing modules is tedious. If like us you have lots of unit tests add some time for getting those working properly, you may find that you're doing a lot of find/replace and copy/paste to get your tests passing with Browserify. In the end we've found the few days effort to refactor worth it.

This format and flow works for us. It has added some considerable time to the UI build on first pass, when we run `$ grunt dev` and are watching for changes we only browserify on newer files. Improving this build time is something we'll be addressing in the immediate future.

We are just scratching the surface in regards to the benefits of Browserify. But, so far, it has proven to be a very powerful and useful tool that we'll continue to explore. If you have any questions, comments, improvements feel free to reach out to [@chasecourington](https://twitter.com/chasecourington). à votre santé

----
## Resources
- [Example Gists](https://gist.github.com/courington/a26024fc04fbc9223a20)
- [Browserify](http://browserify.org/)
- [Browserify handbook](https://github.com/substack/browserify-handbook)
- [Grunt-browserify](https://github.com/jmreidy/grunt-browserify)
- [HBSFY](https://github.com/epeli/node-hbsfy)
- [Grunt-contrib-clean](https://github.com/gruntjs/grunt-contrib-clean)
- [Grunt-contrib-copy](https://github.com/gruntjs/grunt-contrib-copy)