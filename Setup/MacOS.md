## Keyboard

Setting up a new keyboard? Need to fix the Ctrl and CMD keys so it's what I'm used to
### OS side

Swap the control and command keys

Settings -> Keyboard -> Keyboard Shortcuts... -> Modifier keys

![[Pasted image 20230921140648.png]]

### Iterm2

Iterm2 -> Settings -> Keys

Follow [this article](https://jonnyhaynes.medium.com/jump-forwards-backwards-and-delete-a-word-in-iterm2-on-mac-os-43821511f0a) to allow the `option` key to: 
- delete a whole word in iterm2
- jump forward over words
- jump backwards over words 

Change to this

![[Pasted image 20230921140840.png]]

Perfection!

## Homebrew

Install brew to install other packages
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```