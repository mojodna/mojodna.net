---
layout: post
title: My (public) Git Workflow
---

## {{ page.title }}

**The problem**: You want to contribute to a public project that's using git.

**The solution**: Follow conventions and keep clean branches to make them
easier to merge.

### First, Some Background

Picture this: you've just gotten excited about some really cool piece of
technology. It's just before Thanksgiving, and you're about to get on a plane
and cross North America (the wide part, but, thankfully, not diagonally).
You're visiting your parents, but you're not actually visiting your parents;
they're driving a few hours to meet you at an airport to drive a few more
hours to another state. The hotel (or motel) you're staying at in this state
doesn't have wireless (or 3G). You're traveling east. You're sharing a room
with your parents. You can't sleep, but neither can you turn the light on
(because you're considerate, you know).

The next morning, the plan is for all of you to drive a couple more hours into
yet another state and meet up with family you haven't seen for at least a
decade. Thanksgiving, you know. You're going to stuff yourself with food, then
stay at another hotel (or motel) that barely has 3G (still no wireless). It's
late, so it's not so bad.

The next day, you have an awkward brunch with the extended fam, get back in
the car, drive back to state number 2 (or 3), get on a ferry, and arrive
on an island with slightly better 3G (and DSL, but it's connected to your
host's computer and thus unavailable).

There's excitement (kinda), walking (on the beach), more eating, a (skittish)
cat, and downtime. But no internet (or barely). Which, on another vacation,
would be an absolute joy. But remember, you've just gotten really excited
about some really cool piece of technology.

What to do?

Fortunately, you foresaw some of this (maybe not the lying in bed, awake,
across the room from your parents, in the dark until you finally drift off at
2am) and remembered to pack some code. And docs; can't forget them when you're
offline. Oh, and an extra battery (lucky you).

Yay `git`. (You saw this in the title, so no apologies.) Yay VMware. Yay
Ubuntu.  Yay ZeroConf (avahi-daemon).

You're fine. You have everything you need (for some value of "need"). So code.
And you do, hacking late into the night.

However, when you get return after driving to another airport, saying farewell
(for 3 weeks) to your parents, and sitting on another plane (different airline
this time), coding to the sweet soothing sounds of Billy Banks, Rachel Maddow,
and some random show on the Discovery Channel, you run into a new problem: how
to package up your work (which includes enhancements and bugfixes to other
libraries) and publish it (for some value of "publish").

Turns out, you registered a couple of RubyForge projects, but in the end it'll turn out that you don't use them.  You decide to publish some gems on GitHub, re-publish gems for the libraries you modified (with dependencies on those versions), and attempt to get your changes merged upstream.

### Let's Go

For your own projects, it doesn't really matter what you do, at least until
other people start forking and attempting to hack on your stuff. In any case,
you should keep things simple. Topic branches for particular features or
refactoring if you want, but no big deal.

For other peoples' projects, you can (mostly) assume that they're
authoritative (until they're not, in which case you may have assumed the
mantle). Other contributors can be considered "outsiders."

*Something about using the Network Graph to determine "active" and "authoritative" forks.*

**First rule of thumb**: merge from authoritative contributors, cherry-pick
from outsiders.

When hacking on other peoples' stuff, your eventual goal is (probably) to get those changes applied.

**Second rule of thumb**: keep your changes easily apply-able to the
authoritative repository.

For a single line of development, this is easy: rebase against the
authoritative master branch whenever possible. Feel free to rewrite history
(`git push -f`) as long as you're confident no one is treating you as
authoritative.

If others *are* treating you as an authoritative source, create a separate
branch that gets periodically rebased and make it clear that it's not suitable
for tracking. Others' changes will eventually need to be rebased against those
changes, but probably not until your changes have been applied by a real
authoritative contributor.

For changes in multiple areas, this means keeping multiple branches active. I
usually use a branch per feature, plus a separate branch for bugfixes and
documentation. For minor changes, all branches can diverge from a common
commit (usually the tip of the authoritative repository's master branch).

For more drastic changes, you'll need to decide which is either the most
important feature or the least controversial change. Use that as the first
batch and rebase it against the authoritative repository's HEAD. Rebase
subsequent batches against the next most important (or least controversial).

Write down the order in which your branches should be merged. You can use your
project wiki or put it at the top of the README in your master branch. ([See
my fork of `xmpp4r` to see how I've done
it.](http://github.com/mojodna/xmpp4r/blob/06ec50260555ba242880dc2c69229ea5f8ff811e/README.rdoc))
At this point, your master branch is just the default view of your fork and
shouldn't be considerably different from the branch of the authoritative copy
you're tracking (see note below).

In some cases, you'll have branches that aren't quite ready for prime-time.
Treat them the same way as branches that are, but don't make apply-able
branches dependent on them (i.e. they should always be the last changes to be
applied). Include them in the summary in your README, but make it clear that
they're not ready yet.

**Third rule of thumb**: reset your history to match an authoritative
repository's history if their content is the same (or essentially the same).

If your history diverges frequently from an authoritative repository's history, you're going to have more trouble applying others' changes and sync'ing with authoritative repositories.

The simplest way is to delete your repository and re-fork. If you have minor
changes, generate a diff between them to apply once you've re-forked.

#### An Exception

Because GitHub will only build gems for changes on the master branch, if you
have changes that you want to publish (prepared neatly in topic branches, as
above), you'll need to break the rules somewhat.

To handle this scenario, start with your master branch matching the
authoritative branch you're tracking, then merge branches like the owner of
the authoritative branch would (this has the helpful side-effect of making it
clear whether your changes apply cleanly).

The tip of your master should always contain the most up-to-date gemspec (in
order to trigger GitHub to build and publish your fork) as well as a README
that contains information on the state of your repository (including branches,
as above, but also information about how you're treating the master branch).

I've been (mostly) following these rules with my contributions to the 

### Maintaining a Project

*Something about the dangers of using the Fork Queue to apply commits from other co-maintainers / authoritative repositories.*

*Using the GitHub gem and GitX to pull in changes, editing as appropriate (while maintaining authorship).*