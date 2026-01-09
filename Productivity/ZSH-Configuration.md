<div align="center">

# ⚡ ZSH Configuration Guide

**Supercharge Your Terminal with Zsh and Oh My Zsh**

[![ZSH](https://img.shields.io/badge/ZSH-Shell-purple)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-green)]()

*Transform your terminal into a powerful, beautiful, and efficient development environment*

---

</div>

## 📋 Overview

This guide will help you:

- ✅ Install and configure Zsh (Z Shell)
- ✅ Set up Oh My Zsh with powerful plugins and themes
- ✅ Create custom aliases and functions
- ✅ Configure advanced features like autocompletion
- ✅ Customize your terminal prompt and appearance

> 💡 **Note:** Zsh is a powerful shell that offers advanced features like better tab completion, path expansion, and extensive customization options. Oh My Zsh is a popular framework that makes Zsh configuration easy and fun.

---

## 🚀 Step-by-Step Guide

### Step 1: Install Zsh

#### 1.1 Check if Zsh is Installed

```bash
zsh --version
```

#### 1.2 Install Zsh

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install zsh -y
```

**CentOS/RHEL/Fedora:**
```bash
sudo yum install zsh -y
# Or for newer versions
sudo dnf install zsh -y
```

**macOS:**
```bash
# Zsh is pre-installed on macOS Catalina and later
# For older versions or to update
brew install zsh
```

#### 1.3 Verify Installation

```bash
which zsh
zsh --version
```

---

### Step 2: Install Oh My Zsh

Oh My Zsh is a community-driven framework for managing Zsh configuration.

#### 2.1 Install via Official Script

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Or using wget:
```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### 2.2 Set Zsh as Default Shell

The installer will ask if you want to change your default shell. Choose `Y` (yes).

Alternatively, manually set it:
```bash
chsh -s $(which zsh)
```

**Log out and log back in** for the changes to take effect.

#### 2.3 Verify Installation

```bash
echo $ZSH
# Should output: /home/username/.oh-my-zsh
```

---

### Step 3: Configure Oh My Zsh

#### 3.1 Access Configuration File

```bash
vi ~/.zshrc
# Or
nano ~/.zshrc
```

#### 3.2 Essential Configuration

Edit your `~/.zshrc` file:

```bash
# Path to your oh-my-zsh installation
export ZSH="$HOME/.oh-my-zsh"

# Set name of the theme to load
ZSH_THEME="robbyrussell"  # Default theme

# Set list of themes to pick from when loading at random
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Which plugins would you like to load?
plugins=(
  git
  docker
  docker-compose
  kubectl
  npm
  node
  python
  pip
  sudo
  z
  zsh-autosuggestions
  zsh-syntax-highlighting
)

# Load Oh My Zsh
source $ZSH/oh-my-zsh.sh

# User configuration
export PATH="$HOME/bin:$PATH"

# Aliases (add your custom aliases here)
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
```

#### 3.3 Reload Configuration

```bash
source ~/.zshrc
# Or simply restart your terminal
```

---

### Step 4: Popular Themes

#### 4.1 Powerlevel10k (Recommended)

Powerlevel10k is a fast and highly customizable theme.

**Install:**
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

**Configure:**
```bash
# Edit ~/.zshrc
ZSH_THEME="powerlevel10k/powerlevel10k"

# Reload and run configuration wizard
source ~/.zshrc
p10k configure
```

#### 4.2 Spaceship

Modern theme with comprehensive information:

```bash
git clone https://github.com/spaceship-prompt/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"

# Edit ~/.zshrc
ZSH_THEME="spaceship"
```

#### 4.3 Agnoster

Clean and minimal theme:

```bash
# Already included in Oh My Zsh
ZSH_THEME="agnoster"
```

#### 4.4 Other Popular Themes

```bash
# Add to ~/.zshrc
ZSH_THEME="robbyrussell"      # Default, simple
ZSH_THEME="bureau"            # Minimal
ZSH_THEME="pure"              # Minimal, fast
ZSH_THEME="afowler"           # Simple with git info
ZSH_THEME="eastwood"          # Classic Unix style
```

---

### Step 5: Essential Plugins

#### 5.1 Built-in Plugins

Oh My Zsh comes with many built-in plugins. Add them to your `plugins` array:

```bash
plugins=(
  # System
  git              # Git aliases and functions
  docker           # Docker autocompletion
  docker-compose   # Docker Compose support
  kubectl          # Kubernetes commands
  
  # Development
  node             # Node.js shortcuts
  npm              # npm shortcuts
  python           # Python aliases
  pip              # pip shortcuts
  rust             # Rust support
  golang           # Go support
  
  # Utilities
  sudo             # Double ESC to add sudo
  z                # Jump to recent directories
  colored-man-pages # Colorized man pages
  extract          # Extract any archive type
  web-search       # Search web from terminal
  
  # Productivity
  history          # Better history search
  copyfile         # Copy file contents to clipboard
  copydir          # Copy directory path
)
```

#### 5.2 External Plugins

**zsh-autosuggestions** - Suggests commands as you type:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Add to plugins array in ~/.zshrc
plugins=(... zsh-autosuggestions)
```

**zsh-syntax-highlighting** - Syntax highlighting for commands:

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Add to plugins array (MUST be last)
plugins=(... zsh-syntax-highlighting)
```

**zsh-completions** - Additional completion definitions:

```bash
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

# Add to plugins array
plugins=(... zsh-completions)
```

**zsh-history-substring-search** - Better history search:

```bash
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search

# Add to plugins array
plugins=(... zsh-history-substring-search)
```

**After installing external plugins, add to ~/.zshrc:**
```bash
# Load zsh-completions
autoload -U compinit && compinit
```

---

### Step 6: Custom Aliases and Functions

#### 6.1 Useful Aliases

Add these to your `~/.zshrc`:

```bash
# Directory navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias ~='cd ~'
alias -- -='cd -'

# Listing
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ls='ls --color=auto'

# System
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias ps='ps aux'
alias top='htop'  # If htop is installed

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline --graph --decorate'
alias gd='git diff'
alias gco='git checkout'
alias gb='git branch'

# Docker shortcuts
alias d='docker'
alias dc='docker-compose'
alias dcu='docker-compose up'
alias dcd='docker-compose down'
alias dcp='docker-compose ps'
alias dcl='docker-compose logs -f'

# Network
alias ping='ping -c 5'
alias ports='netstat -tulanp'
alias ip='ip -c'

# Editing
alias zshconfig='vi ~/.zshrc'
alias zshreload='source ~/.zshrc'
alias vimconfig='vi ~/.vimrc'

# Utility
alias c='clear'
alias h='history'
alias path='echo $PATH | tr ":" "\n"'
alias now='date +"%T"'
alias nowtime=now
alias nowdate='date +"%d-%m-%Y"'
```

#### 6.2 Custom Functions

Add these functions to your `~/.zshrc`:

```bash
# Create directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Extract any archive
extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.bz2)   tar xjf "$1"     ;;
            *.tar.gz)    tar xzf "$1"     ;;
            *.bz2)       bunzip2 "$1"     ;;
            *.rar)       unrar e "$1"     ;;
            *.gz)        gunzip "$1"      ;;
            *.tar)       tar xf "$1"      ;;
            *.tbz2)      tar xjf "$1"     ;;
            *.tgz)       tar xzf "$1"     ;;
            *.zip)       unzip "$1"       ;;
            *.Z)         uncompress "$1"  ;;
            *.7z)        7z x "$1"        ;;
            *)           echo "'$1' cannot be extracted via extract()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}

# Quick search in history
h() {
    if [ -z "$*" ]; then
        history 1 | awk '!a[$0]++' | head -20
    else
        history 1 | grep "$@" | awk '!a[$0]++' | head -20
    fi
}

# Create a new directory and enter it
mcd() {
    mkdir -p "$1" && cd "$1"
}

# Find files by name
ff() {
    find . -type f -iname "*$1*"
}

# Find directories by name
fd() {
    find . -type d -iname "*$1*"
}

# Quick server (Python)
server() {
    local port="${1:-8000}"
    python3 -m http.server "$port"
}

# Copy file contents to clipboard
copyfile() {
    cat "$1" | xclip -selection clipboard  # Linux
    # cat "$1" | pbcopy  # macOS
}
```

---

### Step 7: History Configuration

Improve your command history:

```bash
# Add to ~/.zshrc

# History file location
HISTFILE=~/.zsh_history

# Maximum number of commands in history
HISTSIZE=10000

# Maximum number of commands in history file
SAVEHIST=10000

# Append to history file, don't overwrite
setopt APPEND_HISTORY

# Share history between sessions
setopt SHARE_HISTORY

# Don't add duplicate commands
setopt HIST_IGNORE_DUPS

# Remove duplicates when writing history
setopt HIST_IGNORE_ALL_DUPS

# Don't save commands starting with space
setopt HIST_IGNORE_SPACE

# Don't save the `history` command itself
setopt HIST_SAVE_NO_DUPS

# Add timestamp to history
setopt EXTENDED_HISTORY
```

---

### Step 8: Advanced Features

#### 8.1 Autocompletion

Zsh has powerful autocompletion. Enable enhanced completion:

```bash
# Add to ~/.zshrc

# Enable completion system
autoload -Uz compinit
compinit

# Case insensitive completion
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'

# Menu selection for completion
zstyle ':completion:*' menu select

# Colors in completion
zstyle ':completion:*' list-colors ''

# Complete aliases
setopt COMPLETE_ALIASES

# Completion cache
zstyle ':completion::complete:*' use-cache 1
```

#### 8.2 Directory Navigation

```bash
# Add to ~/.zshrc

# Auto cd (just type directory name)
setopt AUTO_CD

# Pushd configuration
setopt AUTO_PUSHD
setopt PUSHD_IGNORE_DUPS
setopt PUSHD_SILENT

# Use z plugin for smart directory jumping
# Already included if z plugin is enabled
```

#### 8.3 Key Bindings

```bash
# Add to ~/.zshrc

# Vim key bindings (if you prefer vim mode)
# bindkey -v

# Emacs key bindings (default)
bindkey -e

# Better history search
autoload -U history-search-end
zle -N history-beginning-search-backward-end history-search-end
zle -N history-beginning-search-forward-end history-search-end
bindkey "^[[A" history-beginning-search-backward-end
bindkey "^[[B" history-beginning-search-forward-end
```

---

## 📝 Complete ~/.zshrc Example

Here's a comprehensive example configuration:

```bash
# Path to oh-my-zsh installation
export ZSH="$HOME/.oh-my-zsh"

# Theme
ZSH_THEME="powerlevel10k/powerlevel10k"

# Plugins
plugins=(
    git
    docker
    docker-compose
    kubectl
    node
    npm
    python
    pip
    sudo
    z
    zsh-autosuggestions
    zsh-syntax-highlighting
    zsh-completions
    history-substring-search
)

# Load Oh My Zsh
source $ZSH/oh-my-zsh.sh

# History configuration
HISTFILE=~/.zsh_history
HISTSIZE=10000
SAVEHIST=10000
setopt APPEND_HISTORY
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_IGNORE_SPACE
setopt EXTENDED_HISTORY

# Directory navigation
setopt AUTO_CD
setopt AUTO_PUSHD
setopt PUSHD_IGNORE_DUPS

# Aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias d='docker'
alias dc='docker-compose'
alias zshconfig='vi ~/.zshrc'
alias zshreload='source ~/.zshrc'

# Custom functions
mkcd() { mkdir -p "$1" && cd "$1" }
server() { python3 -m http.server "${1:-8000}" }

# Completion
autoload -Uz compinit
compinit
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' menu select

# Path
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"
```

---

## ✅ Best Practices

1. ✅ **Start with defaults** - Use Oh My Zsh defaults and customize gradually
2. ✅ **Backup your config** - Keep your `.zshrc` in version control
3. ✅ **Use plugins wisely** - Too many plugins can slow down startup
4. ✅ **Organize aliases** - Group related aliases together
5. ✅ **Test changes** - Reload config after each change
6. ✅ **Use theme preview** - Preview themes before committing
7. ✅ **Keep it fast** - Monitor startup time and optimize
8. ✅ **Document custom code** - Comment your custom functions

---

## 🔧 Troubleshooting

### Zsh Not Starting

```bash
# Check if zsh is installed
which zsh

# Verify default shell
echo $SHELL

# Manually start zsh
zsh
```

### Oh My Zsh Not Loading

```bash
# Check if Oh My Zsh is installed
ls -la ~/.oh-my-zsh

# Verify ~/.zshrc exists and sources Oh My Zsh
cat ~/.zshrc | grep oh-my-zsh
```

### Slow Startup

```bash
# Profile startup time
zsh -i -c exit

# Or use zprof
# Add to top of ~/.zshrc
zmodload zsh/zprof

# Add to end of ~/.zshrc
zprof
```

### Plugin Issues

```bash
# Reinstall plugin
cd ~/.oh-my-zsh/custom/plugins/plugin-name
git pull

# Check plugin syntax
zsh -n ~/.oh-my-zsh/custom/plugins/plugin-name/plugin-name.plugin.zsh
```

---

## 📚 Additional Resources

- [Oh My Zsh GitHub](https://github.com/ohmyzsh/ohmyzsh)
- [Zsh Manual](https://zsh.sourceforge.io/Doc/Release/index.html)
- [Powerlevel10k](https://github.com/romkatv/powerlevel10k)
- [Awesome Zsh Plugins](https://github.com/unixorn/awesome-zsh-plugins)

---

<div align="center">

**⚡ Your terminal is now supercharged!**

[← Back to Handbook](../README.md)

</div>

