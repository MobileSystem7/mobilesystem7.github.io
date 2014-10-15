devblog
=======

MS7 Public Developer Blog

Jekyll does not play well with JRuby, and nokogiri/iconv are a little different on OS X. Here's what I had to do to get things working. YMMV.

- rvm use system
- sudo gem install bundle
- sudo gem install nokogiri -- --with-iconv-dir=/usr/local/Cellar/libiconv/1.14
- bundle install
