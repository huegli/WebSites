+++
Description = ""
Tags = ["Offtopic","Blog","CI","Hugo","Go"]
date = "2016-09-04T14:58:52Z"
title = "How this Blog is updated (Part II)"

+++
This part is where all the magic happens.

In the last step of the previous [post](/2016/09/01/website/ "Part 1"), the updates to the 
blog were uploaded to a GitHub repository. 

GitHub has this nice feature that it can trigger events upon check-in. This can be used with a
[Continous Integration](https://en.wikipedia.org/wiki/Continuous_integration "CI") site like
[drone.io](https://drone.io/ "drone.io") to then do something with the checked in content.

In this particular case, I want to build the blog through [Hugo](https://gohugo.io "Hugo") and
then publish it to the actual host (
[Life of Raspberry PI](http://life-of-rpi.nikolaischlegel.com "Life of Raspberry PI")
is hosted on a 
[One.com](https://www.one.com/en/ "One.com") website which I originally set up to have my own
domain name and website for my resume).

After setting up a free account with [drone.io](https://drone.io/ "drone.io"), I created what 
they call a **Project** for building this blog. A project has a seperate **Build & Test** phase as
well as a **Deployment** phase.

Now [drone.io](https://drone.io/ "drone.io") does not support [Hugo](https://gohugo.io "Hugo")
directly, but it does support [Go](https://golang.org "Go"). So what I can do in the build step
is to instruct it to first build [Hugo](https://gohugo.io "Hugo") itself before building the
blog:

```
go get -v github.com/spf13/hugo
rm -Rf ./src ./pkg
cd Hugo/life-of-rpi/themes
git clone https://github.com/keichi/vienna
cd ..
../../bin/hugo
cd ../..
rm -Rf ./bin
mv Hugo/life-of-rpi/public .
rm -Rf Hugo
```
A couple of things to note:

1. While I had to manually clone the theme used for this blog
([Vienna](https://github.com/keichi/vienna "Vienna")), I did not have to clone the blog itself,
[drone.io](https://drone.io/ "drone.io") takes care of that automatically
2. Once the blog is build, I delete everything except the `public` folder containing the
generated HTML for the blog, that way I don't deploy stuff I ultimately don't need

Here is the **Deployment** part of the project:
```
\rm -Rf /www/life-of-rpi/*
cp -R public/* /www/life-of-rpi/
cache-purge http://life-of-rpi.nikolaischlegel.com
```
Notes:

1. Before the above commands are run, [drone.io](https://drone.io/ "drone.io") will run
an `rsync` via ssh of the `public` folder to the deployment host
2. For the `rysync` to work, the host site has to have an SSH key from
[drone.io](https://drone.io/ "drone.io") installed in `./ssh/authorized_keys` 
2. The `cache-purge` command is something that my host requires to ensure that the new
site is propagated in their caching infrastructure.

And there you go, automated building and deployment of this blog with a simple `git push`
and some magic behind the scenes.
