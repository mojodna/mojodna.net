---
layout: default
title: My (work) Git Workflow
---

## {{ page.title }}

**The problem**: you want to track multiple patchsets against an upstream
Subversion repository easily.

**The solution**: Use `git-svn` to track it and create topic branches for
local changes.

[Jump here for my actual workflow](#the_sizzle)

### A Not-So-Brief Introduction (with code!)

I started using [`git`](http://git-scm.com/) and
[`git-svn`](http://www.kernel.org/pub/software/scm/git/docs/git-svn.html) a
little over a year ago. I was an Amtrak regular at the time, so the ability to
make offline commits seemed helpful. Plus, it was shiny, and I'd seen a demo
of [GitHub](http://github.com/), so I had a hunch that it was going to be big.

One day, I sat down over lunch and watched most of [Linus Torvalds' git talk
at Google](http://www.youtube.com/watch?v=4XpnKHJAok8). Until then, I hadn't
really considered DVCSes, staying in my comfortable little Subversion world,
branching out to `cvs` and `bzr` as necessary. I don't really remember what I
got out of the talk, but somehow it inspired me to start reading up on how
`git` could be used with `svn`, much like `svk`.

`git-svn` made its debut performance on our team's IRC channel about a month
later; I had been doing some major refactoring on a local git branch with lots
of commits (squashing commits was mystery to be discovered later) and the time
had come to merge it into *trunk*. The merge went fine (lucky, I guess), so I
went for a `git svn dcommit`. And spammed the IRC channel for the next 30
seconds or so.

Flash forward to the following winter and new (remote) contributors to our
project. It was time to start doing legitimate code reviews, in part to get
the new folks up to speed with the code-base.

In the process of setting up [Review Board](http://www.review-board.org/), I
discovered that it doesn't play well with diffs generated with `git`. Bummer.
They're nice. I spent some time poking around RB's source tree to see why that
was the case. Long story short, no dice. However, on one of the message boards
where the problem had been raised (no resolution, natch), I saw a snippet to
transform `git` diffs into `svn` diffs, revision numbers and all. Bingo. After
fiddling with it for a while, I ended up with the following, which has been
working well.

<script src="http://gist.github.com/44537.js">
// nothing here
</script>

Add the following to your `~/.gitconfig` to enable `git svn-diff`. Awesome. It
even looks like it belongs there. (`git-svn-diff` must be in your `PATH`.)

{% highlight ini %}
# ~/.gitconfig
[alias]
  svn-diff = !git-svn-diff
{% endhighlight %}

<div class="caption">add <code>git svn-diff</code> as an alias for
<code>git-svn-diff</code></div>

#### Bonus Points

Here are the relevant pieces of my `~/.bashrc` to get tab completion and a
colorized prompt that includes `git branch` information in it:

<script src="http://gist.github.com/69960.js">
// nothing here
</script>

### The Sizzle

This is essentially the `git-svn` variation of [Josh Susser's pure `git`
workflow](http://blog.hasmanythrough.com/2008/12/18/agile-git-and-the-story-branch-pattern).

#### The Setup

Clone the target Subversion repository:

{% highlight bash %}
$ git svn clone svn+ssh://svn.host/path/to/repo -s
{% endhighlight %}

<div class="caption">assuming a standard Subversion layout</div>

Alternately, if you're planning on sharing topic branches with fellow
developers, have one person do the clone and pass it out (by copying it, to
retain Subversion metadata).

#### Fixing a Bug / Implementing a Feature

Make sure you're up-to-date:

{% highlight bash %}
[master] $ git svn rebase
{% endhighlight %}

Create a topic branch and check it out:

{% highlight bash %}
[master] $ git checkout -b bug-42
{% endhighlight %}

Attempt a bug fix (write a test, make it pass):

{% highlight bash %}
[bug-42] $ ...
{% endhighlight %}

Check it in:

{% highlight bash %}
[bug-42] $ git commit -a
{% endhighlight %}

Generate a patch for review:

{% highlight bash %}
[bug-42] $ git svn-diff > bug-42.patch
{% endhighlight %}

<div class="caption">this will diff against the checked out trunk
revision</div>

Post it for review.

*Time passes... Your diff hasn't been reviewed, development continues...*

Update the tracking branch:

{% highlight bash %}
[master] $ git svn rebase
{% endhighlight %}

Rebase your topic branch against the current trunk:

{% highlight bash %}
[master] $ git checkout bug-42
[bug-42] $ git rebase master
[bug-42] $ # resolve conflicts; `git mergetool` is handy
{% endhighlight %}

Regenerate the patch:

{% highlight bash %}
[bug-42] $ git svn-diff > bug-42-2.patch
{% endhighlight %}

Post it for review.

*Time passes...  Your diff has been reviewed and been found wanting...*

Make changes to your topic branch:

{% highlight bash %}
[bug-42] $ ...
{% endhighlight %}

Check them in:

{% highlight bash %}
[bug-42] $ git commit -a
{% endhighlight %}

Regenerate the patch:

{% highlight bash %}
[bug-42] $ git svn-diff > bug-42-3.patch
{% endhighlight %}

Post it for review.

#### Landing a Reviewed Patch

You have a few options here.

You can apply the patch directly to the tracking branch:

{% highlight bash %}
[master] $ git apply bug-42-3.patch
[master] $ git commit -a
{% endhighlight %}

If you want to preserve history (i.e. multiple commits that tell a story),
update the tracking branch and rebase your topic branch against it before
merging:

{% highlight bash %}
[master] $ git svn rebase
[master] $ git checkout bug-42
[bug-42] $ git rebase master
[bug-42] $ # resolve conflicts
[bug-42] $ git checkout master
[master] $ git merge bug-42
{% endhighlight %}

If you want to get fancy (and remove your frustrated profanity), do an
interactive rebase on the topic branch before merging:

{% highlight bash %}
[bug-42] $ git rebase -i
[bug-42] $ git checkout master
[master] $ git merge bug-42
{% endhighlight %}

Whew. Almost done. You'll want to update the upstream Subversion repository,
lest you risk your hard work being wasted:

{% highlight bash %}
[master] $ git svn dcommit
{% endhighlight %}

<div class="caption">this will <code>git svn rebase</code> if necessary</div>

If you don't have write access to the upstream repository, submit the patch by
mail instead.

Finally, once the patch has been merged, you can clean up your local
repository by removing the topic branch:

{% highlight bash %}
[master] $ git branch -d bug-42
{% endhighlight %}

That's it! You can keep as many topic branches active as you want (or need
to), rebasing them against your tracking branch as necessary. I use this
approach for bugs, features, and exploratory refactoring that may never see
the light of day.