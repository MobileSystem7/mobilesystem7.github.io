---
layout: post
title: "Automated UI Testing with Sauce Labs, The Intern and Grunt"
author: Chase Courington
tags: ui testing javascript selenium theintern gruntjs saucelabs
category: DevOps
---

Manually testing your UI across multiple browsers and platforms is time consuming. Your time should be spent writing code and building UI components, not running through scenarios in Chrome 39 on Mac OSX 10.10.1, and then again on Windows 8 and then testing IE! Let alone managing your own [Selenium](//www.seleniumhq.org) test farm&hellip;

Thankfully we have tools like [Sauce Labs Connect](//docs.saucelabs.com/reference/sauce-connect/) and [The Intern](//www.theintern.io) to help conduct our tests. I'll walk through how we've setup these two with [Grunt](//www.gruntjs.com) to test the UI in our [Rails](//rubyonrails.org/) app.

----
## Setup Sauce Labs

To get started you'll need a [Sauce Labs](//www.saucelabs.com) account. If you don't have one you can [sign up](//www.saucelabs.com/signup/trial) for a free trial. Sauce Labs frees us from setup and management of test vms. As a bonus it's founded by the creator of Selenium, so it's safe to assume they know what they're doing.

----
## Install Intern
[Intern](//www.theintern.io) interacts with Sauce Labs' API and give us a framework to write functional (and unit) tests for Selenium. Intern gives us tons of flexibility and a stable backing with [Site Pen](//www.sitepen.com) the creators of Dojo. Install to your **package.json** development dependencies: ```$ npm install intern --save-d```

----
## Grunt and Intern
Getting Intern to work with [Grunt](//gruntjs.com) is easy since support is [baked into Intern](//github.com/theintern/intern/wiki/Using-Intern-with-Grunt). We created an Intern task and wired it into our **gruntfile.js**:

**/grunt_tasks/intern.js**

```
module.exports = function(grunt) {  
    grunt.config.set('intern', {
        runner: {
            options: {
                config: 'tests/intern/intern',
                runType: 'runner'
            }
        }
    });

    grunt.loadNpmTasks('intern');
};
```

Once we create the intern task in our **/tasks/** directory we can run that task from the ci with ```$ grunt intern```. This will run our tests in a test runner. Our **gruntfile.js** might look something like:

**/gruntfile.js**

```
module.exports = function (grunt) {

    // Project config
    grunt.initConfig({
        
        // read grunt tasks from npm
        pkg: grunt.file.readJSON('package.json'),

    });

    // load grunt plugins from directory
    grunt.loadTasks('tasks');

    // package up product for production
    grunt.registerTask('prod',
        'Prepare project assets',
        ['clean', 'bowercopy', 'jshint', 'modernizr', 'browserify', 'less', 'cssmin', 'concat', 'uglify', 'copy', 'imagemin', 'jasmine', 'intern', 'clean:prod']
    );

    grunt.registerTask('default', ['prod']);

};
```

We run ```$ grunt intern``` in a **"prod"** task. We still use [Jasmine](//jasmine.github.io/2.1/introduction.html) for our unit tests and for development (for now). We run UI tests when we build for a release since the the UI tests cost more (time and money).

----
## Configure Intern
The [Intern docs](//github.com/theintern/intern/wiki/Configuring-Intern) are pretty great and help us get going with configuration. Areas of interest to get going are:

- **capabilities** namely the **screen-resolution**
- **environments**
- **tunnel** && **tunnelOptions** You'll need to get a Sauce Labs access key from your account panel on the web.
- **functionSuites** This is where we point the AMD loader to our tests

**/tests/intern.js**

```
    ...
    capabilities: {
        'selenium-version': '2.41.0',
        "screen-resolution": "1280x1024"
    },
    ...

    ...
    environments: [
        { browserName: 'internet explorer', version: '11', platform: 'Windows 8.1' },
        { browserName: 'internet explorer', version: '10', platform: 'Windows 8' },
        { browserName: 'internet explorer', version: '9', platform: 'Windows 7' },
        { browserName: 'firefox', version: '29', platform: [ 'Windows 7' ] },
        { browserName: 'chrome', version: '39', platform: [ 'Windows 7' ] },
        { browserName: 'safari', version: '6', platform: 'OS X 10.8' },
        { browserName: 'safari', version: '7', platform: 'OS X 10.9' }
    ],
    ...

    ...
    tunnel: 'SauceLabsTunnel',
    tunnelOptions: {
        username: 'sauceLabsUserName',
        accessKey: 'sauceLabsAccessKey'
    },
    ...

    ...
    functionalSuites: ['tests/intern/functional/login', 'tests/intern/functional/user-subjects'],
    ...
});
```

----
## Write a Functional Test

Writing functional tests with The Intern is extremely simple. Intern comes with [Chai](http://chaijs.com/) which is a great assertion library or you can specify a different one. [Leadfoot](https://theintern.github.io/leadfoot/index.html) has a very nice API for navigating around your remote browser with some great documentation.

Here's a gist of what our test looks like [login.js](https://gist.github.com/courington/1c4ca6043a12432278aa).

We're simply testing some routing for unauthorized users, user sign in, launching a modal, launching a popup inside a modal, and finally navigating to a different page inside our app.

----
## Run the Tests

This is now as easy as ```$ grunt intern```. 

We get ci output giving you status and reporting test results back to you. You can see from the image we have 1 failure.
<img src='/static/img/sauceLabs_intern_ciOutput.png'>

We even see in your Sauce Labs browser ui that you have tests queuing up.
<img src='/static/img/sauceLabs_intern_uiQueue.png'>

We see our UI is failing on Windows 7 in IE 9.
<img src='/static/img/sauceLabs_intern_uiFail.png'>

We can go and step through the tests and view screenshots that are automatically captured along the way. (Windows 8, IE 10)
<img src='/static/img/sauceLabs_intern_login.png'>

----
## Conclusion

Setting appropriate timeouts between actions is imperative. Sometimes tests can fail because you're trying to assert something before it has had time to take place. I've set a default timeout to provide plenty of time for things to happen across the network and in the different vms.

Initially I encountered some issues testing our Rails app, running at port 3000, via the proxy. While Intern and Sauce Labs start you at a port you specificy in your config (:9000 by default). You can use Leadfoot to navigate the remote browser to your local app server (localhost:3000) and then start your tests from there. This adds some time and complexity because you'll need to log in to your app on the remote browser repeatedly for different suites to run if they need to be behind authentication. A little bit of a hack, maybe there's a better way to do this, but I found this to work.

Running functional tests can get costly in time and money. We run Jasmine unit tests as part of our Grunt development process with grunt-contrib-watch and then run a whole functional test suite with Intern and Sauce Labs pre-release. We think this process works pretty well.

The Intern is fairly powerful in that we can also write our unit tests with it. Since we've already got some 300+ Jasmine unit tests we're not jumping at the opportunity to refactor those into the Intern. However we may move that direction eventually to simplify our development and testing process.

For questions, comments, suggestions, etc. please reach out. [@chasecourington](https://twitter.com/chasecourington)

----
## Resources
- [Sauce Labs webdriver api](https://docs.saucelabs.com/reference/test-configuration/#webdriver-api)
- [The Intern configuration](https://github.com/theintern/intern/wiki/Configuring-Intern)
- [Leadfoot /Command](https://theintern.github.io/leadfoot/Command.html)
- [Chai assert](http://chaijs.com/api/assert/)
- [Grunt + Intern](https://github.com/theintern/intern/wiki/Using-Intern-with-Grunt)
- [Intern examples](https://github.com/theintern/intern-examples)
- [Selenium docs](http://docs.seleniumhq.org/docs/index.jsp)

