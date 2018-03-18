---
layout: default
title: zsh terminal install
categories: [zsh, oh-my-zsh, plugins, themes]
category: linux
date:   2018-03-17 16:16:01 -0600
---
# Installation procedure

## Fonts install

```sh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit

yum install fontconfig
fc-cache -vf /usr/share/fonts/
mkdir /usr/share/fonts/
mv PowerlineSymbols.otf /usr/share/fonts/
mv 10-powerline-symbols.conf /etc/fonts/conf.d/
cd ..
rm -rf fonts

# Debian or Ubuntu based Linux distribution
sudo apt-get install fonts-powerline
```

## zsh install

```sh
sudo apt install zsh
which zsh
```

## install oh-my-zsh

```sh
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh
```

## Change default themes

```sh
nano ~/.zshrc
ZSH_THEME="agnoster"
```

## Enable Oh-my-zsh plugins

```sh
cd .oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting

nano ~/.zshrc
plugins=(git extract web-search yum git-extras docker vagrant helm docker-compose zsh-syntax-highlighting zsh-autosuggestions)
chsh -s $(which zsh)
```

[custom plugins]:https://medium.com/wearetheledger/oh-my-zsh-made-for-cli-lovers-installation-guide-3131ca5491fb