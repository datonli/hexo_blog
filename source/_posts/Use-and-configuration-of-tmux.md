---
title: tmux的使用和配置
date: 2018-08-18 01:07:56
tags: linux
---
之前因为觉得tmux比较复杂，一直没有去尝试使用，今天在一同事的怂恿下开始尝试使用，发现非常好，比screen还是强大很多。记录下简单的使用方法和相应的配置文件。

```
创建一新的session或指定名字为xxx的session
tmux [new -s xxx]

打开唯一的session或指定名字为xxx的session
tmux a [-t xxx]
```
通过后面贴出的配置文件，把原本别扭的键位改为跟screen一致的键位，记录下这些键位的简单使用方法。
```
prefix键：Ctrl+a

create : prefix + c
detach : prefix + d
exit : prefix + x
选择特定的window : prefix + 数字键
windows间切换 : F11(prev) 和 F12(next)
```

配置文件如下，可以直接使用：
```
# .tmux.conf
set -g default-terminal "screen-256color" #Terminal setting
set -g display-time 3000                  #Time(ms) to show the message bar
set -g escape-time 200
set-window-option -g automatic-rename off #disable window title auto-rename
set-option -g buffer-limit 16             #Number of copy buffers.
set -g history-limit 65535                #History

setw -g mode-keys vi                      #Use Vi mode
set -g status-keys vi                     #Use Vi mode

#set -g mouse-select-pane on
#set -g mouse-resize-pane on              #resize panel with mouse
#set -g mouse-select-window on            #select window with mouse
#setw -g mode-mouse on                    #Make mouse useful in copy mode
#set -g mouse on

#}

#-------[ Window/Pane ]----------------------------------------# {
set -g base-index 1  # Panel, window 1 base
set -g pane-base-index 1 #Panel 1 base
#set -g message-bg "default"   # Color of the message bar.
set -g message-attr "bold"   # Style attributes for status line messages.
set -g display-panes-active-colour blue #highlight active panel border with blue
set -g display-panes-colour colour250 #orange
# pane border
set -g pane-border-fg colour235 #base02
set -g pane-active-border-fg colour240 #base01
#}

#-------[ Key Binding ]----------------------------------------# {

unbind C-b
set -g prefix C-a # change prefix key to Ctrl-a, same as gnu screen
bind a send-prefix #send ^A

unbind ,
bind A command-prompt "rename-window '%%'" #rename window by A

unbind [
bind Escape copy-mode  # Copy mode

#window navigation
bind -n 'F11' prev
bind -n 'F12' next

# More straight forward key bindings for splitting
unbind %
bind | split-window -h

unbind '-'
bind - split-window -v

#switch panels
bind k selectp -U # switch to panel Up
bind j selectp -D # switch to panel Down
bind h selectp -L # switch to panel Left
bind l selectp -R # switch to panel Right

bind q killp #kill panel
bind-key -t vi-copy 'v' begin-selection  # Begin selection in copy mode.
bind-key -t vi-copy 'C-v' rectangle-toggle # Begin selection in copy mode.
bind-key -t vi-copy 'y' copy-selection  # Yank selection in copy mode.
bind-key <      swap-window -t :-
bind-key >      swap-window -t :+
#}

#-------[ Status Bar and colors ]----------------------------------------# {
set -g status-utf8 on
set -g status-bg black
set -g status-fg blue

set -g status-left "#[fg=colour250,bold,bg=colour235][#S]#[default]"

setw -g clock-mode-colour green
setw -g clock-mode-style 24
setw -g window-status-current-format '#[fg=black,bg=colour167]❰#[bold,fg=black, bg=colour74] #I #W '
setw -g window-status-separator ""
setw -g window-status-format "#[fg=colour243,bg=colour237,bold]❰#[fg=colour250,bg=colour240] #I #W "

set -g status-right-length 50
#cpu load
set -g status-right "#[fg=black,bg=colour72,bold]❰#[fg=black,bg=colour109,bold]"
set -ga status-right " U:#(uptime|sed 's/.*:.//'|sed 's/,//g') "
#ram usage
set -ga status-right "#[fg=black,bg=colour72,bold]❰#[fg=black,bg=colour109,bold]"
set -ga status-right " M:#(free|awk 'NR==2{printf \"%.2f\", 100*$3/$2}')% "
set -ga status-right "#[fg=black,bg=colour72,bold]❰#[fg=black,bg=colour109,bold] %Y-%m-%d %H:%M#[default]"

set -g status-interval 5
set -g message-style "fg=black,bg=colour109,bold"
set -g message-command-style "fg=black,bg=colour109,bold"


#setw -g window-status-activity-bg colour23
#setw -g window-status-activity-fg colour239
set -g visual-activity on
setw -g monitor-activity on
#}

#-------[ Commands ]----------------------------------------# {

# open a panel for man page
bind m command-prompt "splitw -h 'exec man %%'"
bind '~' splitw htop
#reload config
bind r source-file ~/.tmux.conf \; display "Reloaded ~/.tmux.conf"
#}
# vim: fdm=marker foldmarker={,}]}
```
