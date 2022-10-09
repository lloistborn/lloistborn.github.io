# How to run locally

### Pre-requisites
Make sure to install all the dependencies in the gemfile using command `bundle install`. This step is only required if you run this first time.

### 1. Build

Run this command to build all the project
```
bundle exec jekyll build
```

In case of this error is happening when running this on M1 chips

```
bundler: failed to load command: jekyll (/usr/local/bin/jekyll)
/Library/Ruby/Gems/2.6.0/gems/nokogiri-1.13.8-x86_64-darwin/lib/nokogiri/extension.rb:30:in `require': cannot load such file -- nokogiri/nokogiri (LoadError)
```

run this command to set the specific target machine

```
bundle config set force_ruby_platform true
```

If everything is okay, the result of this build should be like this
```
Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 1.071 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
 ```

### 2. Serve
```
bundle exec jekyll serve
```

Result of this serve command should be like this and it should be accessible throught browser
```
Auto-regeneration: enabled for '/Users/faisalmorensya/github.com/lloistborn/lloistborn.github.io'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```