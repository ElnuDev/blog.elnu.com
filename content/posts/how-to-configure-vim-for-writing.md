---
title: "How to Configure Vim for Writing"
date: 2022-06-11T12:00:52-07:00
tags:
  - programming
  - writing
  - vim
draft: true
---

There are a lot of tools commonly associated with programming and the more technically-minded community that are still very much useful for non-technical tasks. Take Git, for example. It was originally created by Linus Torvalds, the creator of Linux, to manage the project’s [version control](https://en.wikipedia.org/wiki/Version_control). While fundamentally it can be utilized to manage the history of any collection of text files, it is still unfortunately primarily used only in the programming sphere, and it is not uncommon for authors to manage their versions with a million draft copies such as `draft.doc`, `draft2.doc`, `draft3.doc`, `final.doc`, `final-final`, `final-for-real-this-time.doc`, etc. or Google Docs. Markdown has gotten wider adoption, but it still isn’t the go-to document format for most people. The fact is, most solutions created by programmers for programmers are much more effective than the “normie” alternatives, and we should be doing our best to increase their adoption.

This includes Vim. I switched over to Vim a couple weeks ago from nano and [VSCodium](https://vscodium.com/) (a free and open source build of VS Code), and I haven’t looked back since. However, Vim isn’t configured to be particularly great at anything out of the box besides perhaps configuration file editing. In this guide, I’ll go over how to configure Vim for writing, assuming you’re writing in plain text or Markdown files. I will cover:

- How to get Vim to have soft line breaks at words, not characters so no words are split across lines
- How to enable spellcheck
- How to use smart/typographic quotes (i.e. `“”` instead of the standard ASCII `""`)
- How to patch your dictionary to include words with typographic apostrophes (by default, words such as `doesn’t` will be flagged as misspelled but `does't` won’t be)
- How to prevent Chinese, Japanese, and Korean (CJK) characters from being flagged as spelling mistakes

With the introduction out of the way, let’s get right into it!

## Soft line breaks

Most Vim configurations are done in your `~/.vimrc` file, which is global to all Vim sessions regardless of file type. However, we are going to want our customizations to only apply to Markdown text files. Luckily, Vim has a feature that handles this: [file type plugins](https://vim.fandom.com/wiki/File_type_plugins), which are stored in `~/.vim/ftplugin/`. In our case, let’s create one for Markdown files: `~/.vim/ftplugin/markdown.vim`.

Add the following:

```VIM
set linebreak
```

And you’re done! Vim will now no longer split words across soft-wrapped lines. Now, let’s move on to typographic quotes.

## Typographic quotes

Vim doesn’t have any support for typographic quotes out of the box, so we’ll need to install the plugin [preservim/vim-textobj-quote](https://github.com/preservim/vim-textobj-quote). There are a few different ways to install plugins in Vim, but in my opinion the easiest one to use is Vundle. If you are on Arch Linux, there is an [AUR package](https://aur.archlinux.org/packages/vundle), otherwise, see the [quick start guide](https://github.com/VundleVim/Vundle.vim#quick-start) on Vundle’s GitHub repository.

Now that we have our Vim package manager installed, let’s install the plugin itself. Open your `~/.vimrc` or create it if it doesn’t exist yet, and add the following:

```VIM
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'kana/vim-textobj-user' " essential dependency for typographic quotes
Plugin 'preservim/vim-textobj-quote' " typographic quotes
call vundle#end()
filetype plugin indent on
```

Between `vundle#begin()` and `vundle#end()` are where all of your Vim plugins to be installed with Vundle are listed; the rest of the lines are various requirements for Vundle to work properly. By default, Vundle assumes the plugins to be GitHub repositories, so all you need to write down is the repository name. [kana/vim-textobj-user](https://github.com/kana/vim-textobj-user) is an essential dependency for our typographic quote plugin that handles text objects.

Now you’ve added this, save and exit, and open Vim. Whenever you add or remove Vundle plugins from your `~/.vimrc`, you have to tell Vundle to update your installed plugins. To do this, enter `:PluginInstall` and press enter. If you’ve done everything correctly, Vundle should pull all your plugins from GitHub, and once it’s done you can type `:q` to exit the installation window.

The typographic quotes plugin is now installed, but it won’t start up by default. We only want it to run on Markdown files, so open up `~/.vim/ftplugins/markdown.vim` again and add the following:

```VIM
call textobj#quote#init()
```

This will initialize the typographic quote plugin whenever you enter a Markdown file.

### Useful commands

- Sometimes you may want to override the plugin to type non-typographic quotes. To do this, press <kbd>Ctrl</kbd> + <kbd>V</kbd> (**V** for **V**erbatim) and then type the single or double quotation mark.
- With this plugin, you convert preexisting text to typographic quotes. There are [instructions on how to do this on the plugin’s repository](https://github.com/preservim/vim-textobj-quote#replace-support), but I was having some difficulty getting them to work.

  I ended up slightly using a slightly modified version of the instructions which maps <kbd>qc</kbd> to replacing with typographic “curly” quotes and <kbd>qs</kbd> to replacing with non-typographic “straight” quotes:

  ```VIM
  map qc <plug>ReplaceWithCurly
  map qs <plug>ReplaceWithStraight
  ```

  In normal mode, these commands replace all occurrences in the current paragraph, and in visual mode (how you do visual selection in Vim, press <kbd>v</kbd>) they replace all occurrences in the current selection. To replace through the entire document, Press <kbd>gg</kbd> to go to the top of the document, <kbd>v</kbd> to enter visual mode, <kbd>G</kbd> to jump to the last line of the document, <kbd>$</kbd> to go to the last character, selecting everything, and finally either <kbd>qc</kbd> or <kbd>qs</kbd> to replace. So, altogether: <kbd>ggvG$</kbd> + either <kbd>qc</kbd> for curly quotes or <kbd>qs</kbd> for straight quotes.
- For more information and advanced usage, [see the plugin’s README](https://github.com/preservim/vim-textobj-quote#vim-textobj-quote).

## Spellcheck

To enable spellcheck, add the following to your `~/.vim/ftplugin/.markdown.md`:

```VIM
set spell spelllang=en
set spelllang+=cjk " prevent CJK characters from being spellchecked
```

Of course, if you want spellchecking in another language, change `en` to whatever your language code is. The second line is optional, however if you often work with documents including Chinese, Japanese, or Korean (CJK) characters this is handy since otherwise vim will mark them all as spelling mistakes.

### Patching dictionary for typographic quotes

If you decided to skip the typographic quotes configuration from earlier, you can stop here, but otherwise we’re going to have to patch our Vim spellcheck dictionary to include words with typographic quotes (e.g. `doesn’t`) which will be all marked as incorrect otherwise.

[This](https://vi.stackexchange.com/q/118) thread on the Vi and Vim Stack Exchange was very helpful, so if you’re having any trouble please refer to the two answers there.

First, create the directory `~/.vim/spell` and enter it if it doesn’t exist already. We’re going to need two dictionary files, which available [here](https://cgit.freedesktop.org/libreoffice/dictionaries/tree/en). Please check to see if there has been a newer version on the website and `wget` that instead if the one I’m using (2020.12.07) has become out of date. I’m going to use the American English (`en_US`) dictionary file, but Canadian (`en_CA`) and Australian (`en_AU`) English dictionaries are also available.

```SH
$ mkdir -p ~/.vim/spell && cd ~/.vim/spell
$ wget http://downloads.sourceforge.net/wordlist/hunspell-en_US-2020.12.07.zip
$ unzip *.zip en_US* && rm *.zip
```

In order to patch the dictionary, run the following command (see [this](https://vi.stackexchange.com/a/172) for an explanation of how it works):

```SH
$ grep "'" *.dic | sed "s/'/’/g" >> en.utf-8.add
```

Then, still in the `~/.vim/spell` directory, start Vim, and run the `:mkspell! en en_US` command, which should create an `en.utf-8.spl` file.
