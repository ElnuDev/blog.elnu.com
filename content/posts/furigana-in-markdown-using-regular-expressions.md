---
title: "Furigana in Markdown Using Regular Expressions"
date: 2022-01-23
tags:
  - japanese
  - programming
description: "The Markdown spec doesn’t include support for furigana, more commonly known as ruby text, which can be troublesome if you want to write content in Japanese. In this tutorial, learn how to add furigana support to Hugo using regular expressions!"
---

> TL;DR: [Here](#creating-the-regular-expression)

### Background

As I was building ~~the current~~ *a previous* version of this website, I came across an issue. In the previous version of the site which I made with [Nuxt.js](https://nuxtjs.org/), a [Vue.js](https://vuejs.org/)-based JavaScript web development framework similar to the [React](https://reactjs.org/)-based [Next.js](https://nextjs.org/), I used [`markdown-it`](https://www.npmjs.com/package/markdown-it) for rendering my Markdown content. The great thing about `markdown-it` was how extensible it is: there are a vast number of available npm packages that extend its functionality beyond the base [CommonMark](https://commonmark.org/) specification.

One of the packages I used was [`furigana-markdown-it`](https://www.npmjs.com/package/furigana-markdown-it), which enabled [furigana](https://en.wikipedia.org/wiki/Furigana), or more widely known as [ruby characters](https://en.wikipedia.org/wiki/Ruby_character), which are reading information written beside logographic characters in East Asian languages. For example, in Japanese the reading for 猫, cat, is ねこ, and can be written with furigana as [猫]{ねこ}. The syntax for this was the main text in square parentheses, `[猫]`, followed by the furigana in curly brackets, {ねこ}, was quite convenient, and I wanted to be able to do the same thing in the new [Hugo](https://gohugo.io/) site.

Instead of `markdown-it`, Hugo by default uses [Goldmark](https://github.com/yuin/goldmark), a Markdown renderer written in Go (a language I don't know), and while extensible, I really didn't want to go through the effort to learn Go, figure out how to make a Goldmark extension, and get it working in Hugo. After looking into it some more, it turns out that in Hugo's templating there is a [`replaceRE`](https://gohugo.io/functions/replacere/) function that lets you find and replace content intelligently using regular expressions.

### Creating the regular expression

After watching [this](https://www.youtube.com/watch?v=rhzKDrUiJVk) useful tutorial by Web Dev Simplified on regular expressions to get an idea of how they work, I managed to create this regular expression:

```RE
\[([^\]]*)\]{([^\}]*)}
```

The first section, `\[([^\]]*)\]` creates a capturing group `()` around character surrounded by square brackets `[]`. The square brackets are escaped by the proceeding backslash (`\[`, `\]`) to make sure they aren't interpreted as regex syntax characters. Inside of the capturing group is `[^\]]*`. The negated set `[^\]]` means any character that isn't a right square bracket `]`, and the asterisk `*` repeats the previous token zero or more times in a row. In other words, the capturing group will end as soon as a right square bracket `]` is detected.

The second section, `{([^\}]*)`, is basically the same thing as the first, except the capturing group is surrounded by curly brackets `{}`. Again, they are escaped to make sure they aren't being interpreted as regex syntax characters.

You can test out the regular expression and see a breakdown of how all the parts of it work on [RegExr](https://regexr.com/6dspa), a super useful tool for building and testing regular expressions.

### Ruby text HTML syntax

The HTML syntax for furigana/ruby text is as follows. For more information, see the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ruby).

```HTML
<ruby lang="ja">猫<rp>(</rp><rt>ねこ</rt><rp>)</rp></ruby>
```

For me, all of my ruby text is going to be in Japanese, so I've added the attribute `lang="ja"` to ensure that the Japanese (not Chinese) character variants are rendered. For some characters, the way they are written in Japan and China slightly differs. For example, for the Unicode character [U+76F4](https://en.wiktionary.org/wiki/直), it is rendered as the Chinese variant, 直, by default but as <span lang="ja">直</span> when the language is explicitly specified to be Japanese, despite them being *the exact same Unicode character code.* 

### Adding the regular expression to Hugo

In Hugo templates, one can display the rendered Markdown content of a given page using `{{ .Content }}`. What we need to do is pass `.Content` into the aforementioned `replaceRE` function. Since Hugo by default escapes HTML syntax, we need to then pipe everything into the `safeHTML` function. The `$1` and `$2` are placeholders for the first and second capture groups in the regular expression, respectively. 

```HTML
{{ replaceRE `\[([^\]]*)\]{([^\}]*)}` `<ruby lang="ja">$1<rp>(</rp><rt>$2</rt><rp>)</rp></ruby>` .Content | safeHTML }}
```

All one needs to do now to get furigana rendering on all of one's page types is replace `{{ .Content }}` with this on all of their templates! To prevent code duplication, I put all of this into a [partial template](https://gohugo.io/templates/partials/).

### Conclusion

I hope you found this first blog post on this site helpful! If you're going to have any Japanese content in your site, being able to write ruby text in your markup is a must. While this tutorial was targeted toward Hugo, you can do this in **any static site generator that supports regular expressions in templates.**
