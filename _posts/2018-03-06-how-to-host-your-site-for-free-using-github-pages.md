---
title: How to host your site for free using GitHub Pages
description: And not just host, but also set your site up to work on a custom
    domain and automate deployment using Travis CI. Automatic spell and grammar
    check included!
tags: [web]
---

There's at least a hundred different ways to create your own blog/personal site,
whether it's using shared hosting with any of the myriad of free content
management systems, using an existing platform such as Medium or Tumblr
(although who in their right mind would do that? :kissing_smiling_eyes:
:musical_note:), but why not try something more extravagant?

For rebooting my site, I chose a solution based on GitHub Pages. It's definitely
not a solution for everyone as it's quite geeky at times, but maybe someone will
find the description of my journey interesting or even decides to follow in my
footsteps.

## Installing the necessary software

Obviously, in order to work with a Jekyll project, I had to install the Jekyll
binary itself, and a few more dependencies that were required by the particular
template I used.

{% highlight shell %}
$ brew install node rbenv
$ rbenv install 2.4.3
$ rbenv global 2.4.3
$ gem install bundler jekyll travis
{% endhighlight %}

I like to have my Ruby versions maintained by `rbenv` so I can theoretically
switch to another Ruby version in different directory, if needed.

## What is GitHub Pages?

Let's quickly go over what GitHub Pages actually is. In layman's terms, it
allows you to transform your Markdown[^1] documents into a full-fledged HTML
pages that can be presented to the end user. The process of transformation is
performed by a program called [Jekyll](https://jekyllrb.com/) and every step of
it can be customized. This post, however, does not intend to focus on Jekyll
directly, so if you're interested in that topic, I recommend checking out their
[documentation](https://jekyllrb.com/docs/home/), it's truly comprehensive.

[^1]: Jekyll is actually not limited to having Markdown as an input for the
    transformation, but can also process Textile and Liquid documents or even
    raw HTML and CSS.

GitHub Pages automatically converts your Jekyll project into a HTML site, with
one caveat: it supports only a limited number of Jekyll plug-ins and templates;
unfortunately for me, the template I chose, a beautifully simple template
called [Chalk](https://github.com/nielsenramon/chalk), was not one of the
supported ones. That means I had to adjust the typical Jekyll workflow a little
bit:

- I had to fork the template's repository and build my work on top of that
    instead of creating an empty Jekyll project from the command line;
- I had to run `npm run setup` to configure my environment;
- I had to start using `npm run local` instead of `bundle exec jekyll serve` to
    preview the site locally[^2];
- and perhaps most importantly, I could not have GitHub Pages compile the site
    for me -- I'll get back to this point in a bit.

[^2]: Technically, I did not *have to* do this. It's just a convenience method
    that Chalk provides.

## Configuring the repository

After forking the template's repository, I decided how my Git workflow would
look like: I'd protect the `master` branch from pushing any commits into it
directly[^3] and force myself to instead have a pull request for any group of
changes I intended to do on the site, including writing a new article. Why?
Because each commit that's a part of a pull request would be processed by
Travis, whom I'd instruct to check the spelling and grammar for any modified
Markdown files and check all hyperlinks in the resulting HTML pages for any dead
links -- and put the results of these checks as a comment in the pull request.
Only after these checks would pass, a merge back to `master` would be allowed.

[^3]: I accomplished this by going to the repository's settings &#8605; Branches
    &#8605; Protected branches &#8605;`master` and checking the *Protect this
    branch*, *Require status checks to pass before merging*, *Require branches
    to be up to date before merging*, *continuous-integration/travis-ci* and
    *Include administrators* options. The second-to-last option will, of course,
    only appear after Travis is enabled for your repository.

And speaking of merging to `master`, that action would also trigger Travis one
more time. This time, on top of all the checks mentioned before, Travis would
yet again compile the site from Markdown to HTML, remove any unnecessary files,
checkout the `gh-pages` branch and force-push the finished site into that
branch, as GitHub Pages serves the content from the `gh-pages` branch as the
website[^4].

[^4]: Theoretically, GitHub Pages can also serve the content of the `master`
    branch or from the `docs` subdirectory of the `master` branch. For my
    use-case, keeping the site's source code in the `master` branch and the
    compiled result in the `gh-pages` branch felt, however, like the cleanest
    possible solution. All of these options can be configured in the
    repository's settings.

## Configuring Travis

How would such Travis configuration look like? For me, this `.travis.yml`
configuration file provided sufficient:

{% highlight yaml %}
language: ruby
branches:
  only:
    - master
cache:
  bundler: true
  pip: true
  yarn: true
  directories:
    - _assets/yarn
rvm: 2.4.3
node_js: "8"
python: 3.4
install:
  - ./bin/setup
before_script:
  - sudo apt-get -qq update
  - sudo apt-get install -y aspell aspell-en
  - pip install --user proselint
  - npm install -g Cellane/node-markdown-spellcheck
script:
  - bundle exec jekyll build --trace
  - bundle exec htmlproofer ./_site --only-4xx --allow-hash-href --assume-extension --check-opengraph --url-ignore "feed.xml"
  - bundle exec danger
after_success:
  - test $TRAVIS_PULL_REQUEST == "false" && test $TRAVIS_BRANCH == "master" && "./bin/automated"
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES: true
  - GITHUB_USERNAME="Milan Vit"
  - GITHUB_EMAIL: milanvit@milanvit.net
  - CLOUDFLARE_ZONE_ID: 9f7f8ee361f355357f2463e64a170d0f
  - CLOUDFLARE_AUTH_EMAIL: milanvit@milanvit.net
  - secure: nrpfexcWiW1VhDpJlFCwXu1BjiLC/YfNdGkFBqW50n88Zz3Kt0ydaHYqy1TiJSkC0uxo7zVV4CHRaVjQhuEjpcPAft+X5JCgKaTBq2aG0Xc1BOYk3b/mJwVtv0NM5mkp1k+/kAINjcAO2s+X/pyaaakxaArEyZQmz+KuQuih/wJCJlP0EZLpXSaf/9CfOgidwJOd56m41/KuJTZ6mSazSnjI9uno02HnYxhwFjO1sSTkMxawRLrOHtJIEZEqoPkzzxlIx+19kcimkdthMAIrMse4fBA2gC7f9+PnTJvWf7NlyOfCc/mID6HI2wTL/StAkbaNp6WywK8w6++c7l5mz+V8qkLy5v29MGvPQ5gdilWyyrTsqVD1tUTjK109ShNrjtgWNvbJe5WPhYG8SCphlxBDOr7uHYwtcjaUaiLZSvTH1fAK77RMPd3vodJMB0hNIyIJ0uTtKU5UXYZyNaq1xzfREWhEDEA2qpThNwJoMrnwHo/ZIc7TJsnqPcgesuBwBUlaJ9jGcQBQrtkmGPPUd1ACVLtMg48uHGY3OuxfDnbm4OyxsJFx3KajYUDQ6gv+YoQnWBdovsFKJFSOMICrhGMuGIz7MsgBFtPNzLVUVJ+RObzMn7MZthJMI2gXEe+QcjSSd0oNdzTFF1fTvYFU+I+C+zR4ptd48Ln+zNTCgx0=
  - secure: akdwPqMO4CGFjd6+meiNxasDVh/6fvsEFQu783leTtsb50y1njqaorTDLcLRHfjzCLRYfB5vlA2ptKVUGmkiI5LU0AejEoVI2Jy3DEkEr9Zc7X/+L25g1dtcVZEvq0lg0xMWQnbn6c5qF9/7yKcyBkiQjLONU9YR4qePRFDBuXbCRRiqt6lDQhJgutqE+lkRMUQg4/7h+eXwEchAI6mM/Cdw+twuIQCAP4yB8xOhOcd3n8hCh8s11Xc6gYOFwFfaDOPFKzig+Iq33Q6RdW9p5AxukDxjyN/p0Ju7G5gY8Vke5qxfp2iIuo9xYH1PRD+ZY6I5dEC9cpOwYPAfURxJihWGt2z5V//MRELWD1k34bqJNdB+cGZiBAov4alfi2ckJb0mUPNbipqoqob1e+A2qpucHoNE+szIf+Kj60IwEGehIj3NR22C0F2MCKOAr7FAGT6dUhWBpq9GETfbr46tJAKF7e3utXub2Z0QEPr/OfrGDuvj13lD4QAMqvURfVWfIFRqt0FSb0kdbqWLKsWf0WOjRaGfnvObLFSJVxB8SLxFki0y3JgphuWkliwFceYAAGU7ITb0bcCuEc/pKzBvPC6wJ7JIw02yHaWHd1/9rrhH3qv4fhNP1ZGZ+DNGzVQcZD4J3IAH9DiBHNVkvU5z0OawVeU0UMerwG3uZZQqo7g=
notifications:
  email: false
{% endhighlight %}

I believe most of the file is pretty self-explanatory (poke me in the comments
below if you disagree), I'll just briefly touch on parts of the environment
section. `GITHUB_USERNAME` and `GITHUB_EMAIL` variables are used for pushing
compiled website by automated script mentioned just below these.
`CLOUDFLARE_ZONE_ID` and `CLOUDFLARE_AUTH_EMAIL` are used to identify my zone
(domain) and me as an user for when I want to purge Cloudflare's cache at the
end of the build.

The two lines towards the end of the file that start with `secret` contain
encrypted GitHub token (`GITHUB_TOKEN`) for the force-push into the `gh-pages`
branch and a Cloudflare token (`CLOUDFLARE_AUTH_KEY`). You can obtain the GitHub
token [here](https://github.com/settings/tokens) (select the entire `repo`
category as your scope) and you can add it into your configuration file by
running following command:

{% highlight shell %}
$ travis encrypt VARIABLE_NAME=VARIABLE_VALUE --add
{% endhighlight %}

The automated deployment script that's run in the `after_success` block
resides in `bin/automated` and looks something like this:

{% highlight shell %}
#!/bin/bash

# Automated deploy script with Travis CI

# Exit if any subcommand fails
set -e

# Variables
ORIGIN_URL=`git config --get remote.origin.url`
ORIGIN_CREDENTIALS=${ORIGIN_URL/\/\/github.com/\/\/$GITHUB_TOKEN@github.com}
COMMIT_MESSAGE=$(git log -1 --pretty=%B)

echo "Started deploying…"

# Checkout gh-pages branch
if [ `git branch | grep gh-pages` ]
then
  git branch -D gh-pages
fi
git checkout -b gh-pages

# Build site
yarn install --modules-folder ./_assets/yarn
bundle exec jekyll build

# Delete and move files
find . -maxdepth 1 ! -name '_site' ! -name '.git' ! -name '.gitignore' -exec rm -rf {} \;
mv _site/* .
rm -R _site/

# Push to gh-pages
git config user.name "$GITHUB_USERNAME"
git config user.email "$GITHUB_EMAIL"

git add -fA
git commit --allow-empty -m "$COMMIT_MESSAGE [ci skip]"
git push -f -q $ORIGIN_CREDENTIALS gh-pages

# Move back to previous branch
git checkout -
yarn install --modules-folder ./_assets/yarn

# Purge Cloudflare cache
echo "Purging Cloudflare cache…"
curl -X "DELETE" "https://api.cloudflare.com/client/v4/zones/$CLOUDFLARE_ZONE_ID/purge_cache" \
     -H 'Content-Type: application/json' \
     -H "X-Auth-Email: $CLOUDFLARE_AUTH_EMAIL" \
     -H "X-Auth-Key: $CLOUDFLARE_AUTH_KEY" \
     -d $'{
  "purge_everything": true
}'
echo

echo "Deployed successfully!"

exit 0
{% endhighlight %}

If you don't use Cloudflare for your domain, you can obviously skip all the bits
related to Cloudflare by deleting relevant parts of `.travis.yml` and
`bin/automated`; however, in my next article, I'll describe how to configure
Cloudflare for custom domain to get both money-free and hassle-free SSL
certificate, so maybe hold on until then.

One last thing I needed to set up in Travis administration panel was a token
for Danger to use when it wishes to spit out the spelling and grammar check
results into a comment on a pull request -- that one is obtained on the
[same page](https://github.com/settings/tokens) as the token for force pushing,
except this time the scope should really be only `public_repo`, and the token
should be stored under the name `DANGER_GITHUB_API_TOKEN`. You can't actually
add it to your `.travis.yml` file as GitHub would detect it during your next
push, send you a scary e-mail and deactivate the token. Sneaky bastard!

Danger itself is configured in `Dangerfile` like this:

{% highlight ruby %}
markdown_files = (git.modified_files + git.added_files).select do |line|
  line.end_with?(".md")
end

ignored_words = []

File.open('.spelling').each_line do |line|
  line.chomp!
  next if line.empty? || line.start_with?('#')
  ignored_words.push(line)
end

prose.ignored_words = ignored_words
prose.ignore_numbers = true
prose.ignore_acronyms = true
prose.lint_files markdown_files
prose.check_spelling markdown_files
{% endhighlight %}

## Configuring the domain

Fortunately, this part was fairly easy. I wanted my site to be available at both
*milanvit.net* and *www.milanvit.net*, so I created two `A` records, one
pointing at `192.30.252.153`, the other one pointing at `192.30.252.154` --
that's the first case taken care of.

After that, I created a `CNAME` entry `www` pointing at `cellane.github.io`,
which takes care of the `www` route.

Once I was done with that, I entered `www.milanvit.net` into the `CNAME` file at
the root of the repository, committed that and after a push, GitHub was able to
tell which site to serve and how.

## Recap: the entire workflow

After painful waiting for DNS records to refresh all the way down to my
computer, I had the site available on my domain. Success!

Whenever I want to make a change (such as writing this exact article), I create
a new Git branch, start making changes, start committing and pushing them and at
some point, open a
[pull request](https://github.com/Cellane/milanvit.net/pulls?utf8=✓&q=). That
triggers Travis to create a build out of that pull request, checking my
spelling, grammar and verifying that all links work, all images have a textual
description and all generated HTML is valid. The results of the build get
presented via Danger back to the pull request and from there, I can at any
point decide that I want to merge the pull request into `master`, thus
triggering another build that in the end results in the compiled site being
pushed into the `gh-pages` branch from which GitHub Pages serves it to visitors
like you.

You might ask, why would one pick such a complicated route to creating a
website?

I think a better question is... why not? Putting all these pieces of puzzle
together was really fun, and isn't that the most important thing when it comes
to anything new in IT? :smirk:

Should you have any questions regarding this setup, feel free to ask in the
comments or get inspired by looking at the
[source code of this website](https://github.com/Cellane/milanvit.net).
