---
layout: post
title:  "How was this site build ?"
date:   2023-11-12
tags: jekyll blog github-pages   
---
When I was trying to build this blog, I would search blogs and stumble upon a good one and my first thought would be "I wish the author did a post on how he put this site together". It seems natural for me to do the first post on that very topic.

**Short story** &#9758; From your GitHub account (say `github.com/myGitAcc`), _Fork_ the [Jekyll-Minima][jekyll-minima] github repo and _rename_ the repo as the `myGitAcc.github.io` to create your _GitHub Page_. &#9758; _Clone_ the repo to a Linux workstation. &#9758; Add the custom code suggested by [miller-blog][miller-blog] to support tags. &#9758; Add/Edit content in the workstation, _rebuild_ the code, and _push_ changes to GitHub. 

**Long story** &#9758; From your GitHub account, _Fork_ the [Jekyll-Minima][jekyll-minima] repo and rename it as `myGitAcc.github.io`. For example, if your account name is `myblog` then, rename the repo as `myblog.github.io` and the page will be automatically hosted at `https://myblog.github.io` as your _GitHub Page_.

**Prepare your Linux Workstation** for building _Jekyll_ sites. Following is for `Ubuntu 22.04`:

&#9758; Install required packages:
{% highlight bash %}
  sudo apt-get install ruby-full build-essential zlib1g-dev
{% endhighlight %}

&#9758; Add the environment variables in `.bashrc` and source it 
{% highlight bash %}
  export GEM_HOME=$HOME/gems
  export PATH=$HOME/gems/bin:$PATH
{% endhighlight %}

Install jekyll and the bundler using gem:
{% highlight bash %}
  gem install jekyll bundler
{% endhighlight %}

**Clone** the GitHub repo using SSH URL (you need to be setup with SSH keys before doing that):
{% highlight bash %}
  git clone git@github.com:myblog/myblog.github.io.git
{% endhighlight %}

After the cloning is complete, the entire repo will be in a directory called `myblog.github.io`. Site-wise **Jekyll configuration** are done in `_config.yml` eg. title, author, email, description.. Check [this doc][jekyll-rundocs] for config  option. For **one time only**, after the clone, get all the dependencies:
{% highlight bash %}
  bundle install
  bundle update 
{% endhighlight %}

Now you can make any changes to the site and build it
{% highlight bash %}
  bundle exec jekyll build 
{% endhighlight %}
This will build the static site from the _markdown contents_ and ready to be hosted. Now you can _commit_ and _push_ the repo using `git`
{%highlight bash %}
   git commit --all [--allow-empty] -m "comment"
   git push
{% endhighlight %}

**Blog posts** are written in _markdown_ and placed in the `_posts` directory. The header of _this post_ looks like this
{%highlight markdown %}
---
layout: post
title:  "How was this site build ?"
date:   2023-10-08 19:12:34 +0530
---
Content... 
{% endhighlight %}

**Tags** are an important component of blogs to search and organize data. Unfortunately, Jekyll's _minima_ template does not support tags by default. Followed this [blog][miller-blog] by Jason Miller to introduce some custom code to support tags. _Summary_ of the blog:  &#9758; Include the keyword _tags_ followed by the tags in the _Front Matter_ of the posts. &#9758; Created the _Liquid_ command file `_includes/collecttags.html` which makes the list `site.tags`. &#9758; Added a liquid snippet in `_include/head.html` to include `collecttags.html`. &#9758; Added the html/liquid snippet in `_layouts/post.html` to display the tags in the post. &#9758; Created `_layouts/tagpage.html` to display each tag in a separate page. &#9758; Finally, a Python script `tag_generator.py` to create the tags. **NOTE** If python3 is not default in your installation, change the first line in the code to explicitly run python3 ie. `#!/usr/bin/env python3`. &#9758; The only change I had to do is _uncomment_ `header-pages:` in `_config.yml` to specifically display the pages in the header menu provided by the list under the `header-pages:` command. The tag generator creates a separate markdown file for each tag in the directory `tag`. Jekyll takes all these files and puts them in the header menu.


You can check [this repo][this-repo] to checkout the codes to create this blog. Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[this-repo]: https://github.com/sroutk-blog.github.io
[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[jekyll-minima]: https://github.com/jekyll/minima
[jekyll-rundocs]: https://jekyll-rtd-theme.rundocs.io/
[miller-blog]: http://www.jasonemiller.org/2020/12/23/tagging-posts-in-jekyll-minima.html
