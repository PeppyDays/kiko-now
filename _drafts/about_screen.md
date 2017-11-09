---
layout: post
title: screen 명령어 알아보기
tags: [linux, screen]
---


```
# ~/.screenrc

# GNU Screen - main configuration file
# All other .screenrc files will source this file to inherit settings.
# Author: Christian Wills - cwills.sys@gmail.com

# Allow bold colors - necessary for some reason
attrcolor b ".I"

# Tell screen how to set colors. AB = background, AF=foreground
termcapinfo xterm 'Co#256:AB=\E[48;5;%dm:AF=\E[38;5;%dm'

# Enables use of shift-PgUp and shift-PgDn
termcapinfo xterm|xterms|xs|rxvt ti@:te@

# Erase background with current bg color
defbce "on"

# Enable 256 color term
term xterm-256color

# Cache 30000 lines for scroll back
defscrollback 30000

# New mail notification
# backtick 101 30 15 $HOME/bin/mailstatus.sh

hardstatus alwayslastline
# Very nice tabbed colored hardstatus line
hardstatus string '%{= Kd} %{= Kd}%-w%{= Kr}[%{= KW}%n %t%{= Kr}]%{= Kd}%+w %-= %{KG} %H%{KW}  %D %M %d %Y%{= Kc} %C%A%{-}'

# change command character from ctrl-a to ctrl-b (emacs users may want this)
#escape ^Bb

# Hide hardstatus: ctrl-a f
bind f eval "hardstatus ignore"

# Show hardstatus: ctrl-a F
bind F eval "hardstatus alwayslastline"

# Remove welcome message
startup_message off
```


```
# ~/.tmux.conf

# Status update interval
set -g status-interval 1

# Basic status bar colors
set -g status-bg black
set -g status-fg cyan

# Left side of status bar
set -g status-left-bg black
set -g status-left-fg green
set -g status-left-length 40
set -g status-left "#S #[fg=white]» #[fg=yellow]#I #[fg=cyan]#P"

# Right side of status bar
set -g status-right-bg black
set -g status-right-fg cyan
set -g status-right-length 40
set -g status-right "#H #[fg=white]« #[fg=yellow]%H:%M:%S #[fg=green]%d-%b-%y"

# Window status
set -g window-status-format " #I:#W#F "
set -g window-status-current-format " #I:#W#F "

# Current window status
set -g window-status-current-bg red
set -g window-status-current-fg black

# Window with activity status
set -g window-status-activity-bg yellow # fg and bg are flipped here due to a
set -g window-status-activity-fg black  # bug in tmux

# Window separator
set -g window-status-separator ""

# Window status alignment
set -g status-justify centre

# Pane border
set -g pane-border-bg default
set -g pane-border-fg default

# Active pane border
set -g pane-active-border-bg default
set -g pane-active-border-fg green

# Pane number indicator
set -g display-panes-colour default
set -g display-panes-active-colour default

# Clock mode
set -g clock-mode-colour red
set -g clock-mode-style 24

# Message
set -g message-bg default
set -g message-fg default

# Command message
set -g message-command-bg default
set -g message-command-fg default

# Mode
set -g mode-bg red
```


```
tt () {
    if [ $# -lt 1 ]
    then
        echo 'usage: tt <hostname...>'
        return 1
    fi
    local tail='echo -n Press enter to finish.; read'
    local session_id=$RANDOM
    tmux new-session -d -s $session_id "$SHELL -ci \"TERM=screen ssh -t -i ~/.ssh/id_rsa_dgate_office dg@dg.daumkakao.io $1; $tail\""
    while [ $# -gt 1 ]
    do
        shift
        tmux split-window "$SHELL -ci \"TERM=screen ssh -t -i ~/.ssh/id_rsa_dgate_office dg@dg.daumkakao.io $1; $tail\""
        tmux select-layout tiled
    done
    tmux set-window-option synchronize-panes on
    tmux attach-session -t $session_id
}
```








