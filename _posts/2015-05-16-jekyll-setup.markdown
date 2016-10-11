---
layout: post
title:  "Create blog in bitbucket using jekyll"
date:   2015-05-16 15:34:11
categories: jekyll
comments: True
---
Jekyll is a great way to quickly build a blog site.  I was looking for easier ways to setup a blog on bitbucket and stumbled upon Jekyll.  

In this post I documented some steps to quickly create a blog (along with some posts) site and deploy it to bitbucket.

Install jekyll and create a new project


{% highlight bash %}
$ gem install jeykyll
$ jeykyll jeykyll_blog
$ cd jeykyll_blog
$ jekyll serve
{% endhighlight %}

jekyll serve will generate all the files in the _site directory.

Open http://127.0.0.1:4000/ in a browser. This will render the blog.

{% highlight html %}
.
├── _config.yml
├── _includes
│   ├── footer.html
│   ├── head.html
│   └── header.html
├── _layouts
│   ├── default.html
│   ├── page.html
│   └── post.html
├── _posts
│   └── 2015-05-16-jekyll-setup.markdown
├── _sass
│   ├── _base.scss
│   ├── _layout.scss
│   └── _syntax-highlighting.scss
├── _site
│   ├── about
│   │   └── index.html
│   ├── css
│   │   └── main.css
│   ├── feed.xml
│   ├── index.html
│   └── jekyll
│       └── update
│           └── 2015
│               └── 05
│                   └── 16
│                       └── jekyll-setup.html
├── about.md
├── css
│   └── main.scss
├── feed.xml
└── index.html

{% endhighlight %}

This is the directory structure of the jeykll. 
config.yml => has the common settings that are used throughout the site.
posts => posts directory the individual posts. The jekyll will generate the post htmls that will be created in the jeykll/update directory.

update about.md, posts.

You can see the updates by refreshing the http://127.0.0.1:4000/

Now that we have the blog running successfullly locally, we need to deploy it to the bitbucket.

create a bitbucket repository with the following format, username.bitbucket.org
add all the files from _site folder to this repo locally and then push it to remote master.
This are the complete commands.

{% highlight bash %}
$ cd jekyll_blog/
$ jekyll build
$ cp -r _site/ ~/bitbucket/username.bitbucket.org/
$ cd ~/bitbucket/username.bitbucket.org
$ git config --global user.name "username"
$ git config --global user.email "user.name@gmail.com"
$ git remote set-url origin https://bitbucket.org/username/username.bitbucket.org
$ git config -l
$ git add --all
$ git commit -m "Jekyll post creation."
$ git push origin master
{% endhighlight %}



Now, open username.bitbucket.org and you should see your blog.

