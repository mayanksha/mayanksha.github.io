---
title: "The Ultimate Javascript, Golang and C Family Vim experience (Part 1)"
toc: true
toc_label: "Table of Contents"
toc_icon: "file-alt"
toc_sticky: true
---


## Intro
How many times have you met someone boasting about Vim (or Emacs or Nano), and the subsequent productivity boosts that they claim to have achieved? If you belong to the programming fraternity, I'm sure it's quite a lot. I'm not here to wage some kind of editor War, yet my allegiance lay with Vim, simply because that's what I've been using from my fresher year.

I don't know if I suffer from OCD (Obsessive Compulsive Disorder), but deep down I have this knack for perfection and I simply can't take my mind off some thing until I have mastered it to perfection.

My philosophy of configuring Vim is really simple --

1. Avoid movement of fingers to Arrow Keys, and completely eliminate their usage. The farthest your little finger is allowed to travel is when pressing a backspace. Period.
2. Avoid un-necessary keystrokes but don't introduce hard to put into muscle memory mappings. In short, stick to as much default as possible when it comes to mappings.

I'll be separating this Article over 4 parts, for I can't do justice to a language's configuration in just one. I promise, by the end of this, you'll have immensely improved your Vim experience, and it will pay itself off in the long run.

Now, let's shift our gears towards Vim and talk about configuring it.


## The Vanilla Vim configuration - Setting the `set` commands

Vim, by default provides a host of options ranging from setting the `folds` and `foldmethods`, to changing the `tabstop` width to allowing for `expandtab`s. Don't feel hazy if you don't understand what these options mean - I'll do no man's job and explain them out.

### Indentation, Line Numbering and Highlighted Searches

The line numbering can be enabled with `set number` (or its shorter version `set nu`), and this generally is the first option I toggle whenever I have to use vanilla Vim. For relative numbering, we have the `set relativenumber` option, but I find it pretty annoying since the numbers change constantly when scrolling using h, j, k or l.

Most of the projects use either a 2 or 4 level indented code, and it's best to stick with these two only, otherwise new contributors may have to scratch their heads fixing indentation for any code changes they make. I don't use Tabs or Spaces uniformly across my own Projects since it's just a matter of simple `set expandtab` to change from Tabs to Spaces, or `set noexpandtab` for vice-versa. (GNU guys, please use [this link](https://gcc.gnu.org/wiki/FormattingCodeForGCC))

The `tabstop` option sets the width of Tab when you press the physical Tab key and `shiftwidth` option does so when you use `>>` in Normal mode to indent your code. The `autoindent` and `smarttab` are quite handy when you're using different blocks of code and automatically ident the next line with 1 extra Tab when you press enter at the start of an `if` or a `while` or a `for` block.

For large code bases, its imperative that you learn how to navigate quickly through code and how to quickly spot what you need. The highlighted searching option (`set hlsearch`) is extremely essential and I absolutely love it. Moreover, you should also turn on incremental searching which shows matching words while you type.

### Folds, History, UndoLevels, Wildmenu and Advanced indentation using cinoptions

When you deal with files having a dozen functions, along with structs and what not,navigation and visibility becomes a mess. You can't focus on a single function, since the surrounding code becomes obtrusive due to peripheral vision. Fold to our rescue -- they basically hide unnecesssary code and can be toggled using `zc` to close and `zr` to open. 

Further, you can also change how folding occurs (i.e. manual vs indented vs syntax based folding). The `set foldmethod=indent` sets the mode to indent. But, this will indent all the code blocks including small `if` statements or loops. So, I have `set foldnestmax=2` which doesn't do more than 2 levels of folding. You can change it per-buffer (eg. for C++ files having classes and methods defined) but globally, I like to keep it this way. 

When you wish to open a new file, either in a split or a buffer or a tab, Tabbed file completions are a necessity and unless you use Ctrl-P Vim (an awesome plugin, more on that later), you're limited by Vim's default tabbed completions. `set wildmenu` shows a new line above the status bar of vim showing all the tab completions when trying to open a file. Moreover, binary and object files are useless while being served for Tab completions, so I use a wild the following `wildignore` options to ignore such files - 

```vim
    set wildignore+=*/tmp/*,*.so,*.swp,*.zip,*.o,*.a        " MacOSX/Linux
    set wildignore+=*\\tmp\\*,*.swp,*.zip,*.exe             " Windows
```

I find the default Vim command history (50 lines) to be very less, so I cranked it up to 1000 lines using `set history=1000`. Also, when re-factoring a piece of code, I find that default undo levels are quite less and I also have it set to 1000 using `set undolevels=1000`.

## What Plugin Manager should I go with?

There's a bunch of Plugin Managers available to be used with Vim - vim-plug, Vundle, Pathogen, dein, but I prefer to use Vundle since it's really easy to setup and works quite well. It can also fetch Plugin sources directly from GitHub, generate HelpTags automatically and clean out unused plugins. 

The Vundle GitHub page ([here](https://github.com/VundleVim/Vundle.vim)).

Please note than the first thing I wish to be doing is to load our Plugin Manager, so I generally keep my `call vundle#begin()` and `call vundle#end()` at the beginning of my .vimrc. Also, take care that all the Plugin paths should go between these two calls, like this 

```vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
...  
*Other Plugins*
...
call vundle#end()
```

## The Plugins - Why and How?

At this point, if I had to change several lines of configuration, I'd be happy given my editor. But our goal here is to have a full fledged IDE-like experience with Vim, and I can't do justice to the readers without introducing them to the various plugins. Each plugin I use serves some purpose for a certain language and you're free to omit them as you wish. 

### Auto-Pairs (jiangmiao/auto-pairs)

From the [GitHub page](https://github.com/jiangmiao/auto-pairs) - Insert or delete brackets, parens, quotes in pair. 

### Supertab ('ervandew/supertab')

When you're in `INSERT` mode and you try to use Omni-Completions, you're offered a list of suggestions to choose from. This plugin enables us to use Tab to cycle through those suggestions. Without this, you'll be inserting a Tab character and will have to cycle through suggestions using Arrow Keys, which violates my philosophy. So, no Arrow Keys. 

### The tpope Family - VimFugitive and VimSurround

[VimFugitive GitHub](https://github.com/tpope/vim-fugitive)

[VimSurround GitHub](https://github.com/tpope/vim-surround)

Firstly, a big shoutout to tpope for his awesome Plugins! vim-fugitive enables git support directly from within vim, so you don't have to press Ctrl-Z, then do some git stuff and be back with `fg`. 

### [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) - The superb and blazingly fast auto-completion engine

Vim by default provides Omni-Completions which provide automatic completions, but on the press of some key. It works okay-ish, but what good is an autocompletion tool if you have to press another set of keys just to have a suggestion window? 

\**Drumroll*\* Enters YCM. This is THE PLUGIN which makes the coding experience in Vim worth every penny. Once you understand how well it works with C Family (and a whole lot of other languages), you'll be amazed at how you were even spending your life without this gem of a plugin. I'll be writing specifically about this in one of the following parts. 

My particular configuration with YCM is as follows. I'm using some language specific options here as well lest it shall become all too scattered. 

```vim
    let g:ycm_gocode_binary_path = "$GOPATH/bin/gocode"
    let g:ycm_godef_binary_path = "$GOPATH/bin/godef"
    let g:ycm_seed_identifiers_with_syntax = 1
    let g:ycm_min_num_identifier_candidate_chars = 5
    let g:ycm_global_ycm_extra_conf = "~/.vim/.ycm_extra_conf.py"
    let g:ycm_python_binary_path = '/usr/bin/python3'
    let g:ycm_confirm_extra_conf = 0
    let g:ycm_always_populate_location_list = 1
    let g:ycm_log_level = "debug"
    let g:ycm_semantic_triggers =  {
    \   'c' : ['->', '.','re![_a-zA-Z0-9]'],
    \   'objc' : ['->', '.', 're!\[[_a-zA-Z]+\w*\s', 're!^\s*[^\W\d]\w*\s',
    \             're!\[.*\]\s'],
    \   'ocaml' : ['.', '#'],
    \   'cpp,objcpp' : ['->', '.', '::', 're![_a-zA-Z0-9]'],
    \   'perl' : ['->'],
    \   'php' : ['->', '::'],
    \   'cs,java,javascript,typescript,d,python,perl6,scala,vb,elixir,go' : ['.'],
    \   'ruby' : ['.', '::'],
    \   'lua' : ['.', ':'],
    \   'erlang' : [':'],
    \ }
    let g:ycm_filetype_blacklist = { 'markdown' : 0 }
    let g:typescript_compiler_binary = 'tsc'
```

The semantic triggers tell the YCM client to check which key press should it use to start predicting text. Like, for C you may want to have suggestions only when you reference some property of a struct with `.` or `->`. I like to keep it on for any key press specifically for C, and the regex next to 'c' in `ycm_semantic_triggers` option does exactly that. 

The `.ycm_extra_conf.py` file tell the YCM engine which compilation flags to use at runtime. More on that later!

### [Vim multiple cursors](https://github.com/terryma/vim-multiple-cursors)

Sublime like multiple cursors. My configuration:

```vim
"Vim-Multiple Cursors Configuration
let g:multi_cursor_next_key='<C-n>'
let g:multi_cursor_prev_key='<C-p>'
let g:multi_cursor_skip_key='<C-x>'
let g:multi_cursor_quit_key='<Esc>'
```

### Vim Airline - A beautiful lightweight status line

Statusline on steroids. Great buffer management owning to the solid angular brackets in the top which makes hopping through buffers extremely easy. You may have to install PowerLine fonts in your distro to view the below characters correctly. They're all UTF-8 encoded chars and may not show in a vanilla distro. 

Unfortunately, this blog is written using markdown and hence doesnt't have support for UTF-8. You can find the configuration in my .vimrc by searching for `vim-airline`. 

## Ending Notes

I may seem fanatic to some since they may question why I have spent so much time perfecting Vim. I say that it actually saves me a lot of time in the long run. I have to type lots and lots of text everyday (and lots of code too, albeit not on a daily basis). I am a touch typist and I type fairly fast and not having to lift my fingers off the keyboard is a boon when you're in the Programming Jedi mode. 

Lastly, comments and critique is welcome! See you in next part. :-)


