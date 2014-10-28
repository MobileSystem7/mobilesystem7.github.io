---
layout: post
title: Replacing the Rails Asset Pipeline with Grunt, Bower and Browserify
author: Chase Courington
---

At Mobile System 7 we're always exploring ways to improve our process. Early this summer we had some disucssion about how to better decouple the UI, 99% Javascript, from the Rails application, which primarily powers our REST api. The opportunity to address this decoupling came when we added a story to address some UI build process issues.

Our UI was using combination of [Rails 3.x Asset Pipeline](http://guides.rubyonrails.org/v3.2.13/asset_pipeline.html) and [Grunt](http://gruntjs.com) to build the UI in development and production environments. We needed streamline this process and agreed that the UI shouldn't rely on Rails to build assets.

The first step in replacing the asset pipeline is identifying what we're relying on and what can we use as replacement. The objective is to decouple the UI from the Rails toolchain, maintaining benefits of the asset pipeline while improving configuration ability.

*Note: <!-- We've been using [Grunt](http://gruntjs.com) for some tasks like [linting](https://github.com/gruntjs/grunt-contrib-jshint) and [concatenation](https://github.com/gruntjs/grunt-contrib-concat) for some time.  -->Since we're already using Grunt and have Node/NPM installed and setup I won't be going over getting that going but you can get some help [here](https://www.joyent.com/blog/installing-node-and-npm/). NPM will be initialized in your /app/assets/ directory*


<hr>
## De-activate the Asset Pipeline

The asset pipeline allows us to specify in our `/app/assets/` directories what javascripts, stylesheets and images to compile/minify. We're going to replace most of this with [Grunt](http://gruntjs.com) tasks. Deactivating the asset pipeline is done with a simple boolean in the application configuration file.


####/config/application.rb

```
config.assets.enabled = false
```

We should also update our development environment config variable that expands compiled assets for debugging since Rails isn't handling any of the uglification/minification.


####/config/environments/development.rb

```
config.assets.debug = false
```

<hr>
## Update the Gemfile

With the asset pipeline enabled Rails automatically adds Gems to our Gemfile for certain dependencies. We have dependencies like [Bootstrap](http://getbootstrap.com) and [jQuery](http://jquery.com/) that we need for our UI but we want more control over these assets and with the asset pipeline de-activated it's doesn't make sense to have the Gemfile manage these assets.

We have an `:assets` group in our Gemfile that we no longer need. The same gems also belong to a `:spec` group, which is for our Javascript unit tests, but we'll be setting those up independent of the Rails app as well. We can remove this entire block in our Gemfile as well as remove the [Jasmine](http://jasmine.github.io/) gem from our `:development` group (your Gemfile may differ).


####/Gemfile

```
...
group :assets, :spec do
    gem 'jquery-rails', '2.2.1'
    gem 'less-rails', '2.2.2'
    gem 'therubyrhino', '2.0.2'
    gem 'twitter-bootstrap-rails', '2.2.3'
end

group :development do
    gem 'jasmine', '1.3.2'
end
...
```

We use [Jasmine](http://jasmine.github.io/) for Javascript unit tests. Since these don't rely on any part of the Rails app and we can run them independently with Grunt, we'll be setting that up as well.

We'll use [Bower](http://bower.io/) to replace those Gems in the `:assets` group. Bower gives us a little more control over the configuration of where we get the assets, where we install the assets, what to call the assets, etc. We'll setup Bower in just a little bit.

Once you've removed those gems/groups from your Gemfile, don't forget to run `$ bundle install` to get a fresh install of your Gemfile and restart your webserver `$ rails s`. Your app should look pretty different now without those assets.


<hr>
## Architecture of the UI

We'll be using [Node](http://nodejs.org/)/[NPM](https://www.npmjs.org/), [Grunt](http://gruntjs.com), [Bower](http://bower.io/) and a litte [Browserify](http://browserify.org/). Before we setup any Grunt tasks or load any Bower assets let's get our UI architecture in the Rails app setup.

We'll keep our development UI code in `/app/assets/` just as before, we'll add a couple of new things to make our UI a "independent Node-ish" application. The `/images/`, `/javascripts/` and `/stylesheets/` should look familiar.

We'll be storing vendor assets from Bower in `/vendor/assets/`, where Rails previously stored assets managed by the Gemfile. With Bower we can configure how this directory is structured a little better than we can relying on the Gemfile.

In our Rails app, `/public/` will contain the files that are served up to our application and it also includes some of the default Rails files like `404.html`. This will be where all our compiled, concatenated, uglified/minified files will end up.


####/app/assets/

```
|-- /app/assets/
|  |-- images/
|  |-- javascripts/
|  |-- stylesheets/
|  |-- tests/
|  |-- grunt_tasks/
|  |-- node_modules/
|  |-- Gruntfile.js
|  |-- package.json
```


####/vendor/assets/

```
|-- /vendor/assets/
|   |-- fonts/
|   |-- images/
|   |-- javascripts/
|   |-- stylesheets/
|   |-- less/
|   |-- tests/
```


####/public/

```
|-- /public/
|   |-- fonts/
|   |-- images/
|   |-- javascripts/
|   |-- stylesheets/
...
```


- **/app/assets/images/**

    We want to optimize the contents and copy optimized images to `/public/images/` to be served up for the application. We'll be using: 
    <!-- - [grunt-contrib-copy](https://github.com/gruntjs/grunt-contrib-copy) -->
    - [grunt-contrib-imagemin](https://github.com/gruntjs/grunt-contrib-imagemin)
    <!-- - [grunt-newer](https://github.com/tschaub/grunt-newer) -->


- **/app/assets/javascripts/**

    The bulk of our UI, we organize with sub-directories for our [Backbone.js](http://backbonejs.org/) app and [Handlebars](http://handlebarsjs.com/) templates. We need to lint, pre-compile, concatenate and copy the contents `/public/javascripts/` to be served to the application. Here we're using:
    - [grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint)
    - [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)
    - [grunt-browserify](https://github.com/jmreidy/grunt-browserify) with [hbsfy](https://github.com/epeli/node-hbsfy)


- **/app/assets/stylesheets/**

    We use [LESS](http://lesscss.org/) as our CSS pre-processor, you could use [SASS](http://sass-lang.com/) or any other option if you'd like. We chose LESS for a slight performance advantage over SASS because it uses Node instead of Ruby for compilation. In the effort to decouple from Rails, we want the pre-processor that doesn't require another dependency outside of our Node toolchain.

    We want to compile our LESS files to CSS, concatenate them with any other vendor stylesheets we use and copy the resulting stylesheet to our `/public/stylesheets/` to be served up to the application. We'll be using:
    - [grunt-contrib-less](https://github.com/gruntjs/grunt-contrib-less)
    - [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)
    - [grunt-contrib-cssmin](https://github.com/gruntjs/grunt-contrib-cssmin)


- **/app/assets/tests/**

    We're moving our Javascript unit tests away from being managed by the Rails toolchain so we can have them standalone and run more easily in development. We can write our tests and easily target our `/app/assets/javascripts/` for testing. Our tests use:
    - [grunt-contrib-jasmine](https://github.com/gruntjs/grunt-contrib-jasmine)


<hr>
## Setting up Node Modules

We need to initialize or update our existing `package.json` file to tell NPM what packages to install. In our case we'll be installing all our packages as `devDependencies` and configuring some of the `scripts` to automate global installs for [grunt-cli](https://github.com/gruntjs/grunt-cli) and [bower](http://bower.io/).

If you don't have a `package.json` then `$ cd /app/assets/`, remember this is our "node app", and `$ npm init` (You need Node/NPM installed).

If you have a `package.json` then make sure it's in `/app/assets/` and open it in your favorite text editor and add:

####/app/assets/package.json

```
...
"scripts": {
    "pre-install": "npm install -g grunt-cli bower"
},
...
```

Running `$ npm install` will now install **grunt-cli** and **bower** globally before installing any of your Node packages. *NPM has it's [opinion on install scripts](https://www.npmjs.org/doc/misc/npm-scripts.html), but we've found this to work for us and we're not publishing this as a package, tread lightly.*

You'll now have `$ grunt` and `$ bower` commands available to you for running tasks and installing packages.

Now that we have those in place we need to install our `devDependencies`, mostly all Grunt tasks. Here's what our `devDependencies` attribute in `package.json` should look something like.

####/app/assets/package.json

```
...
"devDependencies": {
    "grunt": "^0.4.5",
    "grunt-bower-task": "^0.4.0",
    "grunt-browserify": "^3.0.1",
    "grunt-concurrent": "^1.0.0",
    "grunt-contrib-clean": "^0.6.0",
    "grunt-contrib-concat": "^0.5.0",
    "grunt-contrib-connect": "^0.8.0",
    "grunt-contrib-copy": "^0.6.0",
    "grunt-contrib-cssmin": "^0.10.0",
    "grunt-contrib-imagemin": "^0.8.1",
    "grunt-contrib-jasmine": "^0.8.0",
    "grunt-contrib-jshint": "^0.10.0",
    "grunt-contrib-less": "^0.11.4",
    "grunt-contrib-uglify": "^0.6.0",
    "grunt-contrib-watch": "^0.6.1",
    "grunt-modernizr": "^0.6.0",
    "grunt-newer": "^0.7.0",
    "grunt-open": "^0.2.3",
    "handlebars": "^2.0.0",
    "hbsfy": "^2.2.0",
    "time-grunt": "^1.0.0"
}
...
```


<hr>
## Install Bower packages

Run `$ bower init` from your `/app/assets/` and it'll take you through creating a `Bower.json` file, that is a configuration file very similar to the `package.json` to tell Bower what packages to install. We can creat/edit a `.bowerrc` file to tell Bower where we want our `bower_components` installed, in our case we want to install to `/vendor/assets/`.

We use [grunt-bower-task](https://github.com/yatskevich/grunt-bower-task) to add a Grunt task that will install our packages and give us more control over what gets installed and where. We can configure the `Bower.json` with `exportsOverride` to install specific packages js, css, img, font where we want...in corresponding `/vendor/assets/` and then we use [grunt-contrib-copy](https://github.com/gruntjs/grunt-contrib-copy) to move files from `/vendor/` to `/public/` and - [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat) to concatenate javascripts and stylesheets together and move from `/vendor/` to `/public/`.

####/app/assets/bower.json


```
...
"dependencies": {
    "backbone": "1.0.0",
    "bootstrap": "2.3.2",
    "moment": "2.5.0",
    "d3": "~3.4.9",
    "leaflet": "~0.7.3",
    "underscore": "~1.6.0",
    "mustache": "~0.8.2",
    "jquery-ujs": "~1.0.0",
    "fontawesome": "3.2.1",
    "jquery": "2.0.3",
    "bootstrap-datepicker": "~1.2.0",
    "jquery-Mustache": "~0.2.7",
    "leaflet.markercluster": "~0.4.0",
    "modernizr": "~2.8.3"
},
"devDependencies": {
    "jasmine-jquery": "2.0.5",
    "sinon": "http://sinonjs.org/releases/sinon-1.10.3.js"
},
"exportsOverride": {
    "bootstrap": {
      "javascripts": "js/*.js",
      "stylesheets": "css/*.css",
      "less": "less/*.less",
      "images": "img/*.png"
    },
    "backbone": {
      "javascripts": "backbone.js"
    },
    "d3": {
      "javascripts": "d3.js"
    },
    "fontawesome": {
      "stylesheets": "css/*.css",
      "less": "less/*.less",
      "fonts": "font/*"
    },
    "leaflet.markercluster": {
      "stylesheets": "dist/*.css",
      "javascripts": "dist/*.js",
      "fonts": "font/*"
    },...
...
```


<hr>
## Setting up Grunt tasks

We have our `bower.json` setup, our `package.json` setup and finally we need to setup some Grunt tasks to work build our UI.

To keep our Gruntfile manageable we break up our Grunt tasks in the `/app/assets/grunt_tasks/` and load them in commonjs style to our `/app/assets/Gruntfile.js`.


We need a task that can prep our directories/files. The prep task will be the baseline task for:

- cleaning compiled code, `/vendor/assets/`, and `/public/`
- installing Bower assets to `/vendor/assets/`
- hinting our javascripts in `/app/assets/javascripts/`
- pre-compiling our handlebars templates 
- compiling our less to css
- concatenating our javascripts, vendor assets and stylesheets
- moving assets to `/public/` to be served
- optimizing our `/public/images/`
- running our javascript unit tests

####/app/assets/Gruntfile.js

```
module.exports = function (grunt) {

    // output task timing
    require('time-grunt')(grunt);

    // Project config
    grunt.initConfig({
        
        // read grunt tasks from npm
        pkg: grunt.file.readJSON('package.json'),
        
        // configure paths for grunt plugins
        paths: {
            assets: '../../vendor/assets',
            tests: 'tests',

            src_js: 'javascripts', 
            src_css: 'stylesheets',
            src_img: 'images',
            src_font: 'fonts',
            src_json: 'json',
            src_tmp: 'tmp',

            dist_js: '../../public/javascripts',
            dist_css: '../../public/stylesheets',
            dist_img: '../../public/images',
            dist_font: '../../public/fonts',
            dist_json: '../../public/json'
        }

    });

    // load grunt plugins from directory
    grunt.loadTasks('grunt_tasks');

    grunt.registerTask('prep',
        'Prepare project assets',
        ['clean:nuke', 'bower', 'jshint', 'browserify', 'less', 'concat', 'copy', 'newer:imagemin', 'jasmine:ci']
    );

    grunt.registerTask('dev',
        'Prepare project assets',
        ['prep', 'watch']
    );

    grunt.registerTask('prod',
        'Prepare project assets',
        ['prep', 'cssmin', 'uglify', 'clean:prod']
    );

    grunt.registerTask('default', ['prep', 'dev', 'prod']);

};
```

Once we configure those Grunt tasks We can run `$ grunt prep` and watch things build. From here we have `grunt dev` that adds a watch task in the mix to run while changes are made to files we're watching.

####/app/assets/grunt_tasks/contrib-watch.js

```
module.exports = function (grunt) {

    grunt.config.set('watch', {

        scripts: {
            files: [
                '!.grunt',
                '<%= paths.src_js %>/**/*.js',
                '<%= paths.src_css %>/less/**',
                '<%= paths.tests %>/**/*.js'
            ],
            tasks: ['newer:jshint', 'newer:browserify', 'newer:less', 'newer:concat', 'newer:copy', 'newer:imagemin', 'jasmine:ci'],
            options: {
                interrupt: true
            }
        }

    });

    grunt.loadNpmTasks('grunt-contrib-watch');

};
```

We also have a `grunt prod` task that will run our `grunt prep` task and then also run grunt-contrib-cssmin and grunt-contrib-uglify on our stylesheets and javascripts to minify and uglify. Then it will clean out the non-minified/uglified files from `/public/**` so that only those minified files are served up.


Let's take a look at `bower-task.js` to install our Bower packages. Below is what we've got.

####/app/assets/grunt_tasks/bower-task.js

```
module.exports = function(grunt) {

    grunt.config.set('bower', {

        install: {
            options: {
                targetDir: '<%= paths.assets %>',
                install: true,
                cleanTargetDir: true,  // clean the targetDir
                cleanBowerDir: true,  // clean the bower_components dir
                cleanup: true,  // set cleanBowerDir & cleanTargetDir
                copy: true,  // copy bower_components packages to targetDir
                layout: 'byType',  // format tree by component => js/css/less/img
                verbose: false,
                bowerOptions: {
                    forceLatest: false,  // Force latest version on conflict
                    production: false  // Do not install project devDependencies
                }
            }
        }

    });

    grunt.loadNpmTasks('grunt-bower-task');

};
```


The other heavy lifter of our Grunt tasks is the grunt-contrib-concat task. Concat takes all our vendor files, javascripts and stylesheets, and allows us to concatenate them together in a load order so that we can serve up a single file, reducing requests and making uglification/minification easier. It also handles a lot of the moving of files from our `/app/assets/**` to our `/public/**` with the destination concatenated files.

####/app/assets/grunt_tasks/contrib-concat.js

```
module.exports = function (grunt) {

    grunt.config.set('concat', {

        options: {},

        testhelpers: {
            src: [
                '<%= paths.assets %>/test/jasmine-jquery/jasmine-jquery.js',
                '<%= paths.assets %>/test/sinon/index.js',
            ],
            dest: '<%= paths.tests %>/sinon-jasmine-jquery.js'
        },


        app_js: {
            src: [
                '<%= paths.assets %>/js/modernizr/modernizr.custom.js',
                '<%= paths.assets %>/js/jquery/jquery.js',
                '<%= paths.assets %>/jquery-ujs/rails.js',
                '<%= paths.assets %>/js/bootstrap/*.js',
                '<%= paths.assets %>/bootstrap-datepicker/bootstrap-datepicker.js',
                '<%= paths.assets %>/js/d3/d3.js',
                '<%= paths.assets %>/js/underscore/underscore.js',
                '<%= paths.assets %>/js/backbone/backbone.js',
                '<%= paths.assets %>/js/mustache/mustache.js',
                '<%= paths.assets %>/js/jquery-Mustache/jquery.mustache.js',
                '<%= paths.assets %>/js/moment/moment.js',
                '<%= paths.assets %>/leaflet/leaflet.js',
                '<%= paths.assets %>/js/leaflet.markercluster/leaflet.markercluster.js',
                '<%= paths.src_js %>/vendor/bootstrap-overrides.js'
            ],
            dest: '<%= paths.dist_js %>/app.js'
        },


        styles: {
            src: [
                '<%= paths.src_css %>/compiled/app.css',
                '<%= paths.assets %>/bootstrap-datepicker/datepicker.css',
                '<%= paths.assets %>/leaflet/leaflet.css',
                '<%= paths.assets %>/css/leaflet.markercluster/MarkerCluster.css'
            ],
            dest: '<%= paths.dist_css%>/app.css'
        },
...
```


<hr>
## Serving up the UI

In our Rails app we still use .erb `stylesheet_link_tag` and `javascript_include_tag` since they target the `/public/` directory and automatically add a hash to aid in cache busting. Here's an example of how we're including styles and javascripts.

####/app/views/layouts/application.html.erb

```
<% if Rails.env =~ /production/ %>
    <%= stylesheet_link_tag "/stylesheets/app.min.css", media: "all" %>
    <%= javascript_include_tag "/javascripts/app.min.js" %>
<% else %>
    <%= stylesheet_link_tag "/stylesheets/app.css", media: "all" %>
    <%= javascript_include_tag "/javascripts/app.js" %>
<% end %>
```


<hr>
## Conclusion

While we're able to make some major strides in decoupling our UI from the rest of the Rails app there is still some work to be done. It would be good to get [grunt-filerev](https://github.com/yeoman/grunt-filerev) and [grunt-usemin](https://github.com/yeoman/grunt-usemin) included so we don't have to use the Rails asset hashing for cache busting and can also remove those ugly .erb tags. We'd like to introduce Browserify more into the process to reduce the size/complexity of our `contrib-concat.js` file and grunt-contrib-concat task by using commonjs style includes into our javascripts. All in all, decoupling the UI has worked well and helped to improve our UI development process by giving us more control over the configuration.

For questions or comments feel free to reach out [@chasecourington](https://twitter.com/chasecourington). Cheers!
