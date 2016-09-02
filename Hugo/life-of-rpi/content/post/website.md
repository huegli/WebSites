+++
Description = ""
Tags = ["Offtopic","Blog","CI","Hugo","Go"]
date = "2016-09-01T21:08:31-07:00"
title = "How this Blog is updated (Part I)"

+++
Not really related to Raspberry PI's, but worth sharing anyway.

This blog is created with [Hugo](https://gohugo.io "Hugo"), a static
website generation tool written in the
[Go Programing Language](https://golang.org "Go Programing Language").
I use Hugo commands to create new entries for the blog, this
is the command that created the starting point for this particular entry:
```
hugo new post/website.md
```
After the entry is created, I add text to the file using 
[Markdown](https://daringfireball.net/projects/markdown/syntax "Markdown")
syntax using any regular text editor. I, of course prefer to use
[Vim](https://www.vim.org "Vim").

While doing the updates to the Markdown file, the following command 
is running

```
hugo server -D -w
```

which allows me to see a draft of the blog at `http:\\localhost:1313`. 
(`-D` means to also render draft entries and `-w` will dynamically
regenerate all the files of the blog when a source file is changed).

The entire directory tree for this blog is part of a 
[GitHub](https://www.github.com "GitHub"), repository, when I am happy
with what I have entered, I simply commit the changes and push it to
the remote repository

```
git add content\post\website.md
git commit -m "Entry on blog updates"
git push
```

Stay tunned for another entry on how the deployment of the blog updates.
