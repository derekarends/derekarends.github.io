---
layout: post
title:  "Git Commands For Beginners"
date:   2020-05-12 17:09:39 -0500
categories: git beginners
author: Derek Arends
---

Git is becoming a popular version control for all the things.  I have used it primarily in my day to day activities as a software engineer and wanted to help those getting started.  To do so, I plan to do a series of posts to help better understand git and how it works.

The documentation for [git] is helpful when wanting to have a deeper understanding of how these commands work.

## Git Init

If you are looking to get started with an empty repository: [git init]

{% highlight ruby %}
git init
{% endhighlight %}

This will turn whatever directory you are in when you run the command into a git repository.  Allowing you to start versioning your files in that directory.  

## Git Clone

If there is a repository already created and you would like bring it on to your machine: [git clone]

{% highlight ruby %}
git clone path/to/repository
{% endhighlight %}

This will pull the repository down onto your local machine allowing you to make changes locally without impacting the eternal repository.

## Git Add

Now you have made some changes to your documents lets make sure they get versioned!  To get started we will want to stage the changes allowing us to make sure the changes we have are good before saving them to git: [git add]

{% highlight ruby %}
git add -A
{% endhighlight %}

Git add has a lot of options allowing fine turning which files are getting added to staging.  I tend to use -A as I want all the changed files include but you could specify a specific file or even use a wild card to filter which files get added.

## Git Status

So we have the files staged but not yet saved or better known in the git world "committed".  We want to make sure the files we are adding look to be as expected. To do this we will use [git status]

{% highlight ruby %}
git status
{% endhighlight %}

This will allow us to see all the files that have been added, which ones have been modified and not added, or which ones were removed.

## Git Commit

We have verified the files we want to commit.  Now we need to commit them to git: [git commit]

{% highlight ruby %}
git commit -m "My commit message"
{% endhighlight %}

Again this one has a lot of options so I recommend doing a bit of research but this will get you started.  We are now telling git to commit our staged changes with a message of "My commit message". So others will know what was changed and why.

## Git Add Remote

If you did clone your project or are looking to commit your changes to an external source you may have to let git know it exists: [git add remote]

{% highlight ruby %}
git remote add origin path/to/repository
{% endhighlight %}

This lets git know where the external repository exists or will be created.  This allows you to then push your code to that repository.  You will only have to do this once per repository if your origin does not change.

## Git Push

This is used to push your changes to an external source.  Usually done when having cloned a repository or looking to create the repository on the external source: [git push]

{% highlight ruby %}
git push -u origin master
{% endhighlight %}

This is telling git to push your changes to an external source with a repository named master.  

## Until Next Time

This should give you the basics for starting with git. There are many ways to call each one of the examples and I recommend looking at the git documentation if you have the chance but I believe these will help provide guidance on where to start.

[git]: https://git-scm.com/
[git init]: https://git-scm.com/docs/git-init
[git clone]: https://git-scm.com/docs/git-clone
[git commit]: https://git-scm.com/docs/git-commit
[git add]: https://git-scm.com/docs/git-add
[git status]: https://git-scm.com/docs/git-status
[git add remote]: https://git-scm.com/docs/git-remote
[git push]: https://git-scm.com/docs/git-push
