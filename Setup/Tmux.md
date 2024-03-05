Tmux allows you to have multiple windows in the same terminal window for max productivity


## Install

```
brew install tmux
```
## Copy Paste
### Ubuntu 

- **Hold Shift** before and while highlighting text in a tmux window
- After highlighting, press CTRL + SHIFT + c and it'll copy into your clipboard
### Mac

#### Copy Paste

- **Hold shift** until finished highlighting text in a tmux window

## Config

```
vi ~/.tmux.conf
```

```bash
# 0 is too far from ` ;)
set -g base-index 1

# Automatically set window title
set-window-option -g automatic-rename on
set-option -g set-titles on

#set -g default-terminal screen-256color
set -g status-keys vi
set -g history-limit 10000

setw -g mode-keys vi
set -g mouse on
setw -g monitor-activity on

bind-key | split-window -h
bind-key - split-window -v

bind-key J resize-pane -D 5
bind-key K resize-pane -U 5
bind-key H resize-pane -L 5
bind-key L resize-pane -R 5

bind-key M-j resize-pane -D
bind-key M-k resize-pane -U
bind-key M-h resize-pane -L
bind-key M-l resize-pane -R

# Vim style pane selection
bind h select-pane -L
bind j select-pane -D 
bind k select-pane -U
bind l select-pane -R

# Use Alt-vim keys without prefix key to switch panes
bind -n M-h select-pane -L
bind -n M-j select-pane -D 
bind -n M-k select-pane -U
bind -n M-l select-pane -R

# Use Alt-arrow keys without prefix key to switch panes
bind -n S-Left select-pane -L
bind -n S-Right select-pane -R
bind -n S-Up select-pane -U
bind -n S-Down select-pane -D

# Shift arrow to switch windows
bind-key Left  previous-window
bind-key Right next-window

# No delay for escape key press
set -sg escape-time 0

# Reload tmux config
bind r source-file ~/.tmux.conf

# THEME
set -g status-bg black
set -g status-fg white
#set -g window-status-current-bg white
#set -g window-status-current-fg black
#set -g window-status-current-attr bold
#set -g status-interval 60
#set -g status-left-length 30
set -g status-left '#[fg=green](#S) #(whoami)'
set -g status-right '#[fg=yellow]#(cut -d " " -f 1-3 /proc/loadavg)#[default] #[fg=white]%H:%M#[default]'

# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-yank'

run '~/.tmux/plugins/tpm/tpm'
```
