---
layout: post
title:  "A 2022 Guide to Installing Emacs on Windows"
date:   2022-04-16 16:00:00
categories: emacs
---
<!--more-->
{% newthought "I have spent" %} the last few years running Linux, but I got tired of fighting Nvidia drivers and weird GUI bugs. So I've made the switch to Windows 11{% sidenote "one" "An encrypted disk issue ended in me losing my entire drive and rage installing windows. Also, now I can play Elden Ring."%} which has let me try out some new, different, and exciting GUI bugs and Nvidia fights.

## WSL

{% newthought "If you're" %} in the Windows Insider Program, then the obvious choice is to run emacs in WSL. You can get the GUI version going by using [WSLg](https://github.com/microsoft/wslg) and this lets you leave all your configs alone and not have to do the dance of getting all the various bash utils and what not running on windows.

The downside is that you have have to fiddle about with your files in wsl (not a huge deal) and it won't behave as nicely as a native window.
{% maincolumn "assets/emacs-install/wslg-gui.png" "Not only is the window border ugly, but it also doesn't play nice with window snapping which is a dealbreaker for me."  %}
There are alternative window forwarding options for wsl but I decided to go native. How hard can it be?

## Native
{% newthought "After deciding" %} to run emacs on Windows{% sidenote "two" "I'm on some sort of FOSS hitlist now, aren't I?" %} I tried out a most of the commonly recommended options before settling on using [msys2](https://www.msys2.org/). There are pros and cons to all the options but this approach had the advantage of not having to install a bunch of seperate programs to get everything working.

To start, follow the [msys2 installation guide](https://www.msys2.org/#installation). After you've done that you should have access to the msys shell{% sidenote "three" "Because windows obviously didn't have enough shells already." %}. The nice thing is everything we do is in here from now on and you can use it to keep everything up to date from now on. Fun fact, msys2 uses the `pacman` package manager from Arch; so now you can tell people: "I use Arch, btw"{% sidenote "four" "I am not liable for any physical harm that befalls you if you do this." %}

Next up we want to install the [mingw-w64-x86_64-emacs package](https://packages.msys2.org/package/mingw-w64-x86_64-emacs?repo=mingw64), the command will be something like
```shell
pacman -S mingw-w64-x86_64-emacs
```
this will get you emacs itself. The `.emacs.d` folder ends up at `c:/Users/<YOU>/AppData/Roaming`. The emacs install itself will be at `C:/msys64/mingw64/bin/emacs.exe` if you want to make a shortcut to `runemacs`.

If you're the type to symlink your config you can use the `mklink` command in the old school windows terminal (not powershell {% sidenote "five" "Not confusing at all having all these terminals." %})

{% newthought "But you're not out the woods yet"%}, emacs is gonna be missing a bunch of bash utils and other Linux-y goodness that it's expecting to find and won't be able to. 

First up, make sure that `C:\msys64\mingw64\bin` is added to your `PATH` env variable.

Then use msys to install [binutils](https://packages.msys2.org/package/mingw-w64-x86_64-binutils?repo=mingw64), this will fix the `critical error cannot find "as"` from showing up all the time.

### Spacemacs Specific Stuff

Depending on your setup you might not need most of this, I'm running Spacemacs and these were the main things I needed to setup to get things running smoothly

{% newthought "Hunspell or Ispell" %} spell checking won't working out the box. Install [hunspell](https://packages.msys2.org/package/mingw-w64-x86_64-hunspell) and [hunspell-en](https://packages.msys2.org/package/mingw-w64-x86_64-hunspell-en?repo=mingw64) and add something like the following to your config:
```elisp
(setq ispell-program-name "C:/msys64/mingw64/bin/hunspell.exe")
(setenv "LANG" "en_GB")
(with-eval-after-load "ispell"
(setq ispell-really-hunspell t)
(setq ispell-local-dictionary "en_GB"))
)
```
{% newthought "I use Org-download" %} to insert screenshots{%sidenote "six" "The windows screenshot on SUPER-Shift-S is actually super great." %} and images. It won't work with out Imagemagick, so [install](https://packages.msys2.org/package/mingw-w64-x86_64-imagemagick) that and let your config know where the binary is (note the second line if you have Imagemagick already installed another way):
```elisp
(setq org-download-screenshot-method "C:/msys64/mingw64/bin/convert.exe clipboard: %s")
;; (setq org-download-screenshot-method "\"c:/Program Files/ImageMagick-7.1.0-Q16-HDRI/convert.exe\" clipboard: %s")
```

{%newthought "Diff" %} is the last missing piece, grab [diffutils](https://packages.msys2.org/package/diffutils?repo=msys&variant=x86_64) and you should be good to go (assuming you have git installed).

## Bonus: Native Installation on M1 Macs

Use `brew` and the [railwaycat](https://github.com/railwaycat/homebrew-emacsmacport) version of emacsmacport.
```shell
brew tap railwaycat/emacsmacport
brew install emacs-mac
```