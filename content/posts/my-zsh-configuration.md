---
title: "My ZSH Configuration"
date: 2021-12-22T20:01:20-08:00
tags: ["blog", "cli", "linux", "macos"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---
## ZSH!
Below you can find the setup that I've found very comfortable while still maintaining a relatively minimal terminal experience. Confidential environment variables and aliases have been omitted to maintain basic operational security. This post will be edited every time a major revision is made.

## Customizations of Note
* Plugin Manager: [Antibody](https://getantibody.github.io/)
* `cat` Alternative: [bat](https://github.com/sharkdp/bat)
* Theme: [lambda-gitster](https://github.com/ergenekonyigit/lambda-gitster)

## Current `.zshrc` Configuration

```shell
export ZSH="/Users/michaelpeters/.oh-my-zsh"
export PATH=/usr/local/git/bin:$PATH

ZSH_THEME="lambda-gitster"

DISABLE_UPDATE_PROMPT="true"

COMPLETION_WAITING_DOTS="true"

DISABLE_UNTRACKED_FILES_DIRTY="true"

source $ZSH/oh-my-zsh.sh

source <(antibody init)

antibody bundle < ~/.zshplugins

export GPG_TTY=$(tty)

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

alias cat="bat"

alias speedUpGit="git config --add oh-my-zsh.hide-status 1; git config --add oh-my-zsh.hide-dirty 1"

alias gs="git status"
alias gcl="git clone"
alias gc="git checkout"
# GOAT: https://stackoverflow.com/questions/1057564/pretty-git-branch-graphs
alias gl="git log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
alias gcom="git commit -S -m"
alias ga="git add"
alias gpo="git push origin"
alias gp="git pull"
alias gd="git diff"

alias dcu="docker compose up"
alias dcub="docker compose up --build"
alias dcd="docker compose down"
alias dp="docker push"

alias mk="minikube"

timezsh() {
  shell=${1-$SHELL}
  for i in $(seq 1 10); do /usr/bin/time $shell -i -c exit; done
}

rl() {
  exec zsh
}

zshconfig() {
  nano ~/.zshrc
}

temp() {
  sudo powermetrics --samplers smc | grep -i "CPU die temperature"
}

ghm() {
  cd /Users/michaelpeters/go/src/github.com/michaelpeterswa
}

goland() {
  open -na "GoLand.app" --args "$@"
}

autoload -Uz promptinit; promptinit

# The next line updates PATH for the Google Cloud SDK.
if [ -f '/Users/michaelpeters/Downloads/google-cloud-sdk/path.zsh.inc' ]; then . '/Users/michaelpeters/Downloads/google-cloud-sdk/path.zsh.inc'; fi

# The next line enables shell command completion for gcloud.
if [ -f '/Users/michaelpeters/Downloads/google-cloud-sdk/completion.zsh.inc' ]; then . '/Users/michaelpeters/Downloads/google-cloud-sdk/completion.zsh.inc'; fi

```
