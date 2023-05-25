# Open Source Encore: LiquidJS

Through my work on my website, <hubelbauer.net>, I've come to learn more about
Jekyll, Liquid and other stuff one should know when looking to use GitHub Pages,
do some minor modifications, but otherwise stay on the default path as much as
possible.

Let's go over what I learnt - or at least what stuck with me for the past couple
of days.

GitHub Pages are hosted from a static file hosting (or at least that's how one
should think about it - it doesn't matter if there is a real server behind, the
semantics of the feature is that the pages are built offline and deployed as
effectively static files).

They can be built using plain HTML, but there is also a handy feature for making
it possible to render MarkDown based READMEs to HTML so that the GitHub Pages
source could potentially be all prose, no code.

This feature is enabled via Jekyll, a static site builder written in Ruby and
created by Tom Preston-Werner, a name which might ring a bell as the former CEO
of GitHub (a title currently held by Nat Friedman).

It is not surprise Jekyll is built on Ruby, because GitHub itself is and there
is other incredible software built by GitHub folks and in Ruby.
One pick I feel is worth mentioning very much is Scientist:
https://github.com/github/scientist
But that's a digression too far.

Jekyll allows the use of a templating language named Liquid.
This language seems to have been built by Shopify and I am not clear on the
order of events in terms of how that relates to Jekyll etc.
But! As of current, the version of Jekyll GitHub Pages uses supports Liquid and
I believe it is more or less built-in in any Jekyll deployment out there?

Speaking of versions, GitHub Pages ships a relatively ancient version of Jekyll:
https://pages.github.com/versions/

- Jekyll: 3.9.3
- Liquid: 4.0.4

Current Jekyll is at 4.x and current Liquid is at 5.x which brings some problems
when one sets out to use these.

I have not run into any particular issues with the older version of Jekyll, but
the older version of Liquid lacks a simplified form of inline comments:

```liquid
{? # This is an inline comment as can be enjoyed in Liquid 5.4+ ?}

{% comment %}
This is an old-school comment you'll be forced to use if you wanna comment your
Liquid code when running as a part of the GitHub Pages Jekyll instance.
{% endcomment %}
```

Note that I have had to use question marks `?` instead of percent signs `%` in
the new style comment sample because otherwise Jekyll would try to interpret
it when the site was building and fail. :/

I wish I could use these, but I managed to get by with the old-school version.
I don't support GitHub is in rush to bump the Jekyll or Liquid version on
GitHub Pages otherwise they'd have already done it, so I am not holding my
breath to see these upgrades.
Another indication on how fast or not GitHub is in approaching upgrades across
their product offering, just take a look at the JavaScript-based custom GitHub
Actions runtime.
It is running Node 16, some of the default Actions are still running Node 12 and
being converted to Node 16 as we speak and at the time of writing, Node 16 is
just a tiny bit over three months away from being out of support.
In fact, I expect GitHub will just straight to Node 20 on this runtime skipping
Node 18 altogether, which while a good thing in the end shows the general speed
at which GitHub bumps these things.
So, for now and a long while still probably, get comfortable with these older
versions when using GitHub Pages.

But how does all this related to LiquidJS?
Quite simply in fact!

I want to write a stand-alone post on this, but in a nutshell, the structure of
how this site runs is quite unusual.

The GitHub repository backing this site has no actual content, it is just the
structure - the HTML of the main page, an empty JavaScript file in case I need
to do something interactive in the future, a basic stylesheet, nothing crazy.

Where the content comes in is in the build process.
I list the content that should show on the site in Git Modules of the main site
repository.
Whenever I write a new post, I add it as a submodule of the site's repository.
I've built a custom GitHub Actions to make adding and removing submodules doable
without checking out the main site repository locally even.
It is just a matter of adding or removing a few lines.
Each post is its own repository and since it is added as a submodule, when Pages
are deployed, the modules are checked out and show up as directories on the site
and Jekyll sees these just as normal directories while building the site prior
to the deployment.

This means that in order to get a live feed of posts on the main page, I can go
over the `.gitmodules` file, list the entries recorded there and build the post
list on the main page.
There are some extra metadata that find their way there as well but that's not
important for this post.

To go over the `.gitmodules` file, I include it as part of the Jekyll build
(Jekyll ignores dotfiles by default) and then use `{? include_relative ?}` in a
`{? capture ?}` to store the contents of the file as a string in a varible.
I then parse the string and build pseudo-objects representing the posts.
I use these objects to render the link list and voila!
A simple site is born.

Again, note that the Liquid syntax is changed here to use `?` over `%` because
GitHub Pages tries to interpret the code and fails.

For the future though, I want to make the list of the posts on the main page
show up in a random order after each new build of the site or maybe even hourly
or something like that.

In order to achieve that, I need to do quite a lot in Liquid.
Take a look at what's already going on there:
<https://github.com/TomasHubelbauer/hubelbauer.net/blob/main/index.html>

All of this code was debugging by making changes, waiting for the build and
deployment (which is pretty fast but it is not seconds so it adds up to a bit of
annoyance when stuff refuses to work and I need to keep adding logs and trying
random ideas).
This took a lot of time!

I could install Jekyll locally and speed stuff up but I'd rather not install
stuff on my computer.
What I could do, though, is copy that Liquid code, find some sort of an online
playground to run it in and see if it does what I need it to do and then tweak
it there and bring the whole thing over once it is ready!

I was elated to find this is not just a dream, but an actual thing one can do,
today:
<https://liquidjs.com/playground.html>

Unfortunately though, there are some filters which are built-in to Liquid and
this library implements them all, and then there are some which are added to
Liquid by Jekyll and this library lacks those.

I need `push` to be able to an array of arrays (to represent the parsed post and
their metadata being the nested items) and I can't use `concat` because it will
flatten an array when being concatenated onto another so it is impossible to
make a nested array in this Playground at the moment.

I also will need `sample` to be able to randomize the array before I go and
render the posts onto the main page.

These two features were missing so I set out to contribute them to LiquidJS so
that they become available in the Playground and allowed me to use a faster way
to debug the Liquid code than just comitting to the repository like a madman,
waiting for the build and deployment and checking the site and its source code
for secret debugging HTML comments.

Adding stuff to LiquidJS is _easy_.
I did not even clone the repository when I added `push` at first.
You can see my PR here:
<https://github.com/harttle/liquidjs/pull/611>

Later on I came back to be a good netizen and add some tests:
<https://github.com/harttle/liquidjs/pull/614>

I also contributed the `sample` filter, this time with tests built-in:
<https://github.com/harttle/liquidjs/pull/612>

When I came in looking to contribute this, the first step was of course to see
if there was already an open PR or an open issue asking for this.
I do this every time, I swear; and don't go reading the previous Open Source
Encore if you actually believed me.
<https://hubelbauer.net/open-source-encore-2023-05-23-vs-code/>

I did find an open ticket asking for this here:
<https://github.com/harttle/liquidjs/issues/443>

This ticket is about general support for Jekyll Liquid filters (so the ones that
don't come built-in but almost any Jekyll site has them anyway) and it named a
few but I decided to go over all of them based on the link here and see which
ones were already there, which could be added and how and which could probably
not be added (because they are dependent on the context and state of the Jekyll
site):
<https://jekyllrb.com/docs/liquid/filters/>

Turns out the majority of these should be add-able.
Like I said, I contributed `push` and `sample` and the maintainer was so kind he
merged my `push` PR even in the first iteration without tests.

Unfortunately though, even though the GitHub Pages site hosting the Playground
has its deployment tied to a CI workflow in GitHub Actions and as such got a new
version deployed once my PR was merged, my `push` filter does not work there.

I later added the tests so I know the problem is not in the implementation of
the filter, rather it is likely that there is a missing step I need to do in
order to make the filter work in the Playground.

Right now it is not even syntax-highlighting, it is just in the state of not
failing as an invalid filter name, but also doesn't do anything beyond that.

I still have the `sample` PR open and the maintainer seems active so in a couple
of days I am hoping to have some more information on what I should do next as
well as a review for the `sample` filter.

Once I sort this out, the Playground should have all I need for my planned next
version of the Liquid code used to generate the site's main page.
When that's done, it will be much simpler to actually prototype and test that
code than what I've been doing up until this point.

I am looking forward to this!
And I liked contributing to LiquidJS enough that I am thinking of spending more
time on some of the other Jekyll-based filters that could be added to LiquidJS!
