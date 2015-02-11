---
layout: post
title: Atom and Markdown Writer for Jekyll
date: "2015-02-10"
categories:
  - jekyll
  - blog
---

I started to use [Atom](https://atom.io/) with the [Markdown-Writer plug-in](https://atom.io/packages/markdown-writer) for writing my Jekyll blog posts.  I had previously been using [MarkdownPad2](http://markdownpad.com/) but found I was missing some [Jekyll]() specific features:

- Generating Posts with Jekyll [Front Matter](http://jekyllrb.com/docs/frontmatter/) auto populated
- Managing Categories
- Managing Tags

I still use MarkdownPad2 as a general purpose markdown editor but for Jekyll blogging the Markdown-Writer plug-in is a big improvement in my work flow.

## Markdown Writer Configuration
The [basic configuration](https://atom.io/packages/markdown-writer) for Markdown-Writer is fairly straight forward but does require a little extra work to make it work great with Jekyll.  You can find basic documentation at [https://atom.io/packages/markdown-writer](https://atom.io/packages/markdown-writer).

### Front matter configuration
To set up the [front matter](http://jekyllrb.com/docs/frontmatter/) you need to open the atom configuration (```ctrl+shift+p``` type ```config```) and add in:

```
'markdown-writer':
  /*may have other markdown-writer configuration here */
  'frontmatter': '---\nlayout: post\ntitle:  "<title>"\ndate:  "<date>"\ncategories:\n---'
```

### Tags and Categories
To set up category and tag lookups you have to add a few files to your Jekyll project.  These files will create JSON files with all the tags and categories from your blog which will be referenced by the plug-in.

Drop the [following scripts](https://gist.github.com/zhuochun/fe127356bcf8c07ae1fb) into the same place you store your css files.  When the site is compiled the categories and tags will be generated.

<script src="https://gist.github.com/zhuochun/fe127356bcf8c07ae1fb.js"></script>

After you republish you can open the markdown-writer configuration page (```ctrl-,``` and search for ```markdown-writer```) and point the setting ```URL to Categories/Tags JSON definition``` to the Url of the respective JSON files. For example:

```
www.yoursite.com/content/categories.json
```

## The Markdown-Writer Workflow
Using Atom and Markdown Writer greatly improved my workflow.  Before I would fumble while manually creating a file with the correct naming convention, manually create the front matter, guess at the correct naming for my categories/tags, and then finally start writing.  

My new workflow looks like this:

1. Open Atom
2. ```Ctrl-shift-p``` type ```new post``` to create new markdown writer post
3. ```Ctrl-shift-p``` type ```categories``` to add categories from a list directly from my blog.
4. Start writing.

Hope that helps improve your workflow.  Happy Blogging!
