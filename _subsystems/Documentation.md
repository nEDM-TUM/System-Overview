---
title: Documentation
description: Writing documentation for different systems
layout: basic
---

The documentation for different systems is written and publicized using
[GitHub Pages](https://pages.github.com/).  This means that each repository
(i.e. system) can have its own set of documentation pages.  It is important to
note that the documentation for a repository is always in the `gh-pages` branch
of a repository and should be edited there.  Most of these repositories use
[jekyll](https://help.github.com/articles/using-jekyll-with-pages/) to produce
the documentation in
[Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

### Set up new documentation for a repository

General instructions for setting up documentation for a new repository using
the standard [layout schemes](https://github.com/nEDM-TUM/web-layouts) for the
nEDM experiment.  Note, that this basically follows instructions on the
[pages site](https://pages.github.com/).

{% highlight bash %}
git checkout --orphan gh-pages
git submodule add https://github.com/nEDM-TUM/web-layouts.git _layouts # pull in standard layouts
cp /path/to/Python-Slow-Control/_config.yml . # Copy in another _config.yml file, change to your needs
mkdir _subsystems # Store in this directory sub-pages (markdown) if needed
# edit markdown pages, see different repositories for examples
git push origin gh-pages # push back to github to publish
{% endhighlight %}

### Documentation for project

The overall documentation for the project should be edited in the
[nEDM-TUM.github.io repository](https://github.com/nEDM-TUM/nEDM-TUM.github.io).
Layouts should updated/added in the [web-layouts repository](https://github.com/nEDM-TUM/web-layouts).

Note that if a layout has been changed, then all repositories with
documentation must be forced to rebuild by submitting a commit (may be empty).
A script to do this:

{% highlight bash %}
#!/bin/bash

# It is assumed this is a script running in a set of folders that looks like:
#
# ./update_layout.sh # this script
# nEDM-TUM.github.io/ # folder synced to nEDM-TUM.github.io repo
# Munin-Docker/ # folder synced to Munin-Docker repo
# FileServer-Docker/ # folder synced to FileServer-Docker repo
# etc ...
#

function update() {
 git submodule update --remote
 git add _layouts && git commit -m 'Update layout'
 git push
}

# First do the overall Web documentation, this uses the 'master' branch
pushd nEDM-TUM.github.io
update
popd
for i in Munin-Docker FileServer-Docker '+ other folders!'; do
 pushd $i
 git co gh-pages
 update
 popd
done
{% endhighlight %}

