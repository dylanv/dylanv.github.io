---
layout: post
title:  "Getting Started"
date:   2022-04-03 16:00:00
categories: general
---
<!--more-->
{% epigraph 'Writing about something, even something you know well, usually shows you that you didn’t know it as well as you thought. Putting ideas into words is a severe test. The first words you choose are usually wrong; you have to rewrite sentences over and over to get them exactly right. And your ideas won’t just be imprecise, but incomplete too. Half the ideas that end up in an essay will be ones you thought of while you were writing it. Indeed, that’s why I write them.' 'Paul Graham' ' "Putting Ideas into Words" '%}
{% newthought 'This is not my first attempt' %} at blogging, I have a bad habit of setting these up and then nuking them after a while because I'm not happy with the look and feel (my graphic design skills leave a lot to be desired). Which is a pity, I think expressing yourself is a genuinely useful skill and they only way to get better at anything is to practice. However,  due to my natural engineer aversion to writing, I have let this languish. 

This time I've decided to put the effort in and get a setup I actually like {% sidenote 'one' 'Sidenotes!!!'%} (more on this in a moment). I'm fairly happy with the current look and feel it's got all the bells and whistles like pretentious epigraphs and my beloved sidenotes. {% marginnote 'one' 'Fighting the urge to change things to use a [historiated initial](https://en.wikipedia.org/wiki/Historiated_initial) at the start of every post, like this is an illuminated manuscript and not a blog.'%}There isn't actually much that's missing, but I'm sure I'll find someway to tinker.

## Nuts and Bolts

{% newthought 'The theme' %} is based on [clayh53/tufte-jekyll](https://github.com/clayh53/tufte-jekyll) which is itself based on Edward Tufte's [tufte-css](https://github.com/edwardtufte/tufte-css). It has some great liquid tags for setting up side/margin notes, epigraphs, and various other stylish and useful bits and pieces already built in.

I did make some changes, most obvious is the colours. I've always loved the Financial Times colour scheme {% sidenote 'two' 'The actual physical newspaper was also [pink](https://qz.com/462285/why-the-financial-times-is-pink/) for interesting reasons.'%} and decided to use it here as well. Besides, how often do you get the chance to use a pink background?

{% newthought 'Installation' %} was relatively straightforward. I just followed the official Github Pages getting started guide and poked around a bit in the Jekyll docs. Once you're relatively on top of what Jekyll is all about download/clone/whatever the [theme](https://github.com/clayh53/tufte-jekyll) and move all the files into your repo.  You'll have to change the configs (pay attention to your base url especially) to match your site and do any style/template changes, and you should be off to the races. If you do change your background make sure you look at the code style background as well.

I did have to delete the `Gemfile.lock` {% sidenote "three" "This may have been a Windows problem (it normally is)." %} and add some gems to get things working correctly. You can then use `bundle exec jekyll serve -w` to run and test locally.

### Gotchas
{% newthought "The aforementioned" %} gemfile issue was solved by modifying it to:
```ruby
source 'https://rubygems.org'

gem 'jekyll'
gem 'rouge'
gem "webrick", "~> 1.7"
# Windows needs tzinfo for some reason
gem "tzinfo", "~> 2.0"
gem "tzinfo-data", "~> 1.2022"
# jekyll serve asked for this
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```
and then deleting the lock file and re-running `bundle install`.

{% newthought "Deploying" %} is also a bit tricky because Github Pages doesn't support the Liquid tags and plugins. To get around this create a new empty branch `gh-pages` and move the `_site` folder there and commit when you want to deploy. There is a `rake` file included with the theme that will do this for you, but it is a bit dangerous.

{% newthought "Otherwise" %} check out the list of [edge cases](https://clayh53.github.io/tufte-jekyll/articles/20/Edge-Cases) and pay special attention to the quote marks you use in the liquid tags.