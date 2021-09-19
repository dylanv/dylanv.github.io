# A empty blog
Note much here yet but getting there :)

## What's here
Based on [Jekyll](https://jekyllrb.com/) and the [amplify](https://github.com/ageitgey/amplify) theme.

## Dev instructions
Make sure you gave ruby and ruby-dev installed
`sudo apt install ruby ruby-dev`
Install the jekyll and bundler gems:
`gem install jekyll bundler`
To run the site locally run: `bundle exec jekyll serve` and navigate to localhost:4000

## A quick note on getting Latex equations to display easily
Kramdown expects changes to be in blocks delimited with `$$` see [here](https://kramdown.gettalong.org/syntax.html#math-blocks).
But wait! There's more! The Latex is rendered with [Mathjax](https://www.mathjax.org/) so you need to add something like:
```javascript
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
```
to your `post.html`. 

Remember randomly copying code off the internet is always safe.
