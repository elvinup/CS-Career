
On a Mac? Configure your [[MacOS]] keyboard settings first!

## Iterm2

Install this terminal emulator first
## Zsh

Install [[Zsh]]
## Vim

Install and setup .vimrc with [[Vim]]

## Tmux

[[Tmux]] config for multiple windows in terminal
## Mise

Install [[Mise]] to install most other tools
## Docker

#### [Stoken Container](https://github.com/jriddle-sf/stoken-in-docker)

Useful for grabbing the centro token

## Raycast

For automating actions on a macbook. Follow [[Raycast]]

## Maven

Follow [[Maven]] installation instructions

## SSH

- Setup SSH Config with ~/.ssh
	- Git SOMA (id_rsa)
	- Regular Github_sfemu (id_ed22519_work)

## GPG key setup if required

```
brew install gnupg

gpg --gen-key # no pw required
gpg --list-secret-keys --keyid-format=long 

# sec ed25519/<key> 202403-05
gpg --armor --export <key> | pbcopy # Paste this into Github GPG Key
```

GPG Git Signing

```
git config user.signingkey <key>
git config --global commit.gpgsign true
```

