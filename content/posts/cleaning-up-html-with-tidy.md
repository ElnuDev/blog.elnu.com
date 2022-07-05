---
title: "Cleaning Up HTML with Tidy"
date: 2022-01-24
tags:
  - programming
description: "Static site generators such as Hugo create incredibly messy HTML output. Luckily, HTML tidy can make your ugly markup pristine, even removing comments! Let’s learn how to use tidy and configure it to your needs."  
---

One issue with using static site generators that use templates like [Hugo](https://gohugo.io) is that their generated HTML files are often incredibly messy, with bad indentation, tons of unneeded whitespace, and overall unconcise formatting. While it *is* technically possible to fix this in your templates, it takes a lot of effort, makes your templates less readable, and frankly is a waste of time.

To my knowledge, Hugo and most other generators don't have built-in beautifiers. The reasoning for this is usually that since your HTML is hidden from your end-user, it doesn't matter if it isn't clean and not nicely formatted. While this is valid, if you're a obsessive perfectionist (like myself, unfortunately) you probably still want clean markup.

In addition, if your site or blog has a more technical audience that might possibly view your site's source, you're going to give a better impression if you have clean markup.

## HTML Tidy

`tidy` is the solution for this. Started in 2003 by Dave Raggett, it provides a simple command line tool with [a large number of configuration options](http://api.html-tidy.org/tidy/tidylib_api_5.2.0/tidy_config.html) for how you want your HTML to be "tidied." It is free and open source, with its source code available on [GitHub](https://github.com/htacg/tidy-html5).

On Debian-based GNU/Linux distributions, you can install it with:

```SH
sudo apt install tidy
```

For other platforms, see [HTML Tidy's official website](https://www.html-tidy.org/).

It's most simple usage is as follows, with first the output file being declared with the `-o` flag and then the input file at the end:

```
tidy -o output.html input.html
```

However, the default configuration will probably not be to your liking. We'll go over configuration in the next section.

## My `tidy` configuration for usage with Hugo

I have created a simple, one line command for first building my site with `hugo`, then cleaning up all of the generated files by first using `find` to get all `*.html` files in `public` (where Hugo dumps generated content), and then finally running `tidy` on each file.

```SH
hugo && find public -path "*.html" -type f -exec tidy --quiet yes --drop-empty-elements no -o {} {} \;
```

I have a couple configuration settings:

- `--quiet yes`: Since I'm parsing a large number of files, this prevents `tidy` from spitting out informational text every time the command is run.
- `--drop-empty-elements no`: In this site, I use the [FontAwesome](https://fontawesome.com/) icon set (which is fantastic, I definitely recommend it), which encodes icons as empty `<i>` tags with classes applied. By default, `tidy` removes any empty tags with no content, which removes all my icons, so I need to have this disabled.

And that's it! Now, every time I update my Hugo site, I can run this command and not have to worry about Hugo's messy HTML output.

I hope this helped anyone out there who dislikes dirty, unconcise HTML as much as I do!
