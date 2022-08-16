# A basic jekyll blog

## Getting starting

First up, grab the requirements for Jekyll, guide [here](https://jekyllrb.com/docs/installation/#requirements).

 Then install any gems you're missing
```shell
bundle install
```

After that you should be good to go for dev with:
```shell
bundle exec jekyll serve

```

## Deployment
At the moment github pages can't automagically build all our liquid tags so we have to build manually.

First, build the site. This will put the static site in the `_site` folder
```shell
bundle exec jekyll build
```

We then need to move the site files to the `gh-pages` branch.
This can be done manually or there is also a Rake command in `Rakefile` if you need an idea of what must be done.

## Theme
Based on the tufte-jekyll theme from [clayh53/tufte-jekyll](https://github.com/clayh53/tufte-jekyll).
With the colours slightly changed to be more like the Financial Times.

And some overall changes to structure, headers, footers, that kind of thing.

## Twitter support
Inline tweets support is provided by the [jekyll-twitter-plugin](https://github.com/rob-murray/jekyll-twitter-plugin).
Nicely, this doesn't require any authentication and we get an easy to use liquid tag:
`{% twitter https://twitter.com/jekyllrb maxwidth=500 %}`


To install add the gem
```ruby
gem "jekyll-twitter-plugin"
```
and add `jekyll-twitter-plugin` to the `_config.yml`.

