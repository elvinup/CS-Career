## RTX

Install `rtx` to install most tools

```bash
curl https://rtx.pub/install.sh | sh

echo "eval \"\$(/home/elvin/.local/share/rtx/bin/rtx activate bash)\"" >> ~/.bashrc # or .zshrc if you're using zsh
```

## Trick out Terminal with Zsh and Custom Themes

https://medium.com/@satriajanaka09/setup-zsh-oh-my-zsh-powerlevel10k-on-ubuntu-20-04-c4a4052508fd

Make sure to set 

```
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=5'
```

To make autosuggestions more visible if they're hard to see