---
title: "如何让自己的 Linux/macOS 终端更好用"
slug: "better-terminal"
date: "2020-04-18T17:19:00+0000"
lastmod: "2025-01-17T01:58:41+0000"
draft: false
tags:
  - "Linux"
  - "macOS"
  - "Terminal"
  - "Shell"
visibility: "public"
---
# 0X00 [视频在这里](<https://www.bilibili.com/video/BV1JA411b7dp/>) 下面是配置文件

这篇博客要配合[发在bilibili的视频](<https://www.bilibili.com/video/BV1JA411b7dp/>)来看，这个文件是在`~/.zshrc`的。大家有问题直接在视频下面留言或者直接给我私信好了～

```sh
    # system env
    export ZSH="/Users/shawn/.oh-my-zsh"
    export LANGUAGE=en_US
    export LANG=en_US.UTF-8
    # ZSH_THEME="agnoster"
    ZSH_THEME="powerlevel10k/powerlevel10k"
    EDITOR=/usr/bin/vim
    PATH=$PATH:$HOME/Library/Python/3.7/bin
    PATH=$PATH:$HOME/Library/Python/2.7/bin


    HIST_STAMPS="yyyy-mm-dd"
    HISTFILESIZE=100000
    HISTFILE=~/.zsh_history

    # zsh plugin
    plugins=(
        z
        git
        docker
        fabric
        extract
        thefuck
        fzf-zsh
        git-open
        colored-man-pages
        zsh-autosuggestions
        zsh-syntax-highlighting
    )

    # alias for simple command
    alias py2='/Users/shawn/Library/Python/2.7/bin/ipython2'
    alias py='/Users/shawn/Library/Python/3.7/bin/ipython3'
    alias cat='/usr/local/bin/bat'
    alias down='aria2c -x16 -j4'
    alias me="cd $HOME/Workstadion/ && ls"

    # alias to source command
    alias _cat='/bin/cat'

    # ctrl + n autosuggest
    bindkey '^n' autosuggest-accept

    source $ZSH/oh-my-zsh.sh

    # docker
    attach() {
      docker exec -it `docker ps | grep $* | awk -F ' ' '{print $1}'` bash
    }

    attach_django() {
      docker exec -it `docker ps | grep $* | awk -F ' ' '{print $1}'` python manage.py shell
    }

    git_set_proxy() {
      git config --global http.proxy 'socks5://127.0.0.1:1080'
      git config --global https.proxy 'socks5://127.0.0.1:1080'
    }

    git_unset_proxy() {
      git config --global --unset http.proxy
      git config --global --unset https.proxy
    }

    json() {
        # echo `xclip -o` | jq   # Linux
        echo `pbpaste` | jq    # macOS
    }

    # To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
    [[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```