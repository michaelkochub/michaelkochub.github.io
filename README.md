# Personal Site

I'll try to do some blog posts here, mostly about stuff I find interesting...

## How does this work?

GitHub Pages servers run [Jekyll][Jekyll], which is a static site generator written in [Ruby][Ruby].
Jekyll reads the configuration yaml at the root directory and uses a templating language called [Liquid][Liquid] to create a styled site with content. 
Jekyll is optimized for blog posts and prides itself on minimizing configuration time.

## How do I run this site locally?

I will assume you are asking how to build the site and run it on a local server.

```bash
$ gem install bundler
$ gem install jekyll
$ bundle exec jekyll serve
```

You can then navigate to `localhost:4000`, and by the way, your browser will be refreshed after any change!

## Some additional useful links
* [GitHub help][GithubHelp] with getting Jekyll running.
* Great [getting started guide][guide] to Jekyll with GitHub Pages.

[Jekyll]:https://jekyllrb.com/docs/usage/
[GithubHelp]:https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/#step-2-install-jekyll-using-bundler
[Liquid]:http://shopify.github.io/liquid/
[Ruby]:https://www.ruby-lang.org/en/
[guide]:http://jmcglone.com/guides/github-pages/
