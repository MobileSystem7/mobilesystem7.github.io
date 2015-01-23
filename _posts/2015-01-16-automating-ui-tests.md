---
layout: post
title: "Automating UI Tests with SauceLabs, TheIntern.io and GruntJS"
author: Chase Courington
tags: ui testing javascript selenium theintern gruntjs saucelabs
category: DevOps
---

Manually testing your UI across multiple browsers and platforms is lame. That's why your team should have a QA Engineer, but alas they don't. This means the onus to test your UI falls on you, if you test across multiple browsers and platforms at all. Your time should be spent writing code and building UI components, not running through scenarios in Chrome 39 on Mac OSX 10.10.1, and then again on Windows 8 and then again on Mac OSX 10.9.5. Let alone managing your own [Selenium](//seleniumhq.org) test farm or testing across mobile devices.

Thankfully we have tools like [Sauce Labs](//saucelabs.com) and [The Intern](//theintern.io) to help us UI Developers conduct our tests. I'm going to walk through how we're using these two in our [Grunt](//gruntjs.com) build process to test the UI.

1. Setup a SauceLabs Account- a free one works for this

2. Install [Intern](//theintern.io) to your _package.json_ development dependencies: `$ npm install intern --save-d`

3. Install Grunt-Intern

4. Configure The Intern
  - environments
  - token
  - resolutions
  - suites
  - grunt options


5. Write a functional test

6. Run your tests

7. Rejoice!

8. Other stuff you can do with Intern

9. The landscape (browserstack, casperjs, nighthawkjs, selenium)

9. Resources

<!-- <img src='/static/img/overview-ia-aac.png'> -->