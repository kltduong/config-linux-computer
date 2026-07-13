# Linux Setup Notes

This file collects the setup commands and notes for a fresh Linux/Kubuntu/Ubuntu environment.

## 0. Optional: Mount SSD

Use this only when you need to mount an NTFS SSD manually.

```bash
sudo mount -t ntfs-3g /dev/sda1 /mnt
```

## 1. Update and Upgrade System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

## 2. Install Basic Tools

### Install curl

`curl` is a command-line tool for transferring data.

```bash
sudo apt install -y curl
```

### Install Terminator

Terminator is a terminal emulator.

```bash
sudo apt install -y terminator
```

### Install Git

```bash
sudo apt install -y git-all
```

## 3. Copy Vim Configuration

Copy `.vimrc` to the home directory.

Run this from the directory that contains `.vimrc`.

```bash
cp .vimrc ~/
```

## 4. Install Zsh and Oh My Zsh

### Install Zsh

```bash
sudo apt install -y zsh
```

### Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### Change Oh My Zsh Theme

Manually edit `~/.zshrc` and change the theme line.

Example:

```bash
nano ~/.zshrc
```

Find this line:

```bash
ZSH_THEME="robbyrussell"
```

Change it to your preferred theme, then reload Zsh:

```bash
source ~/.zshrc
```

## 5. Install Docker Desktop

Reference:

```text
https://docs.docker.com/desktop/setup/install/linux/ubuntu/
```

Recommended approach:

1. Set up Docker's package repository.
2. Download the latest `.deb` package.
3. Install the downloaded package with `apt`.

Example:

```bash
sudo apt-get update
sudo apt install ./docker-desktop-amd64.deb
```

Docker's documentation notes that this message can be ignored at the end of installation:

```text
N: Download is performed unsandboxed as root, as file '/home/user/Downloads/docker-desktop.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
```

## 6. Install fcitx5-unikey

Reference: [https://github.com/fcitx/fcitx5-unikey](https://github.com/fcitx/fcitx5-unikey)
`fcitx5-unikey` is the Vietnamese Unikey input method engine for Fcitx5.

### 6.1 Install packages

```bash
sudo apt install -y \
  fcitx5 \
  fcitx5-config-qt \
  fcitx5-frontend-gtk3 \
  fcitx5-frontend-qt5 \
  fcitx5-frontend-qt6 \
  fcitx5-unikey
```

### 6.2 Set Fcitx5 as input method

```bash
im-config -n fcitx5
```

Set Fcitx5 for KDE Wayland:

```bash
kwriteconfig6 \
  --file kwinrc \
  --group Wayland \
  --key InputMethod \
  /usr/share/applications/fcitx5-wayland-launcher.desktop
```

Reboot:

```bash
reboot
```

### 6.3 Verify after reboot

```bash
pgrep -a -x fcitx5
pgrep -a -f 'fcitx5-wayland-launcher'
```

Expected:

```text
/usr/bin/fcitx5
/usr/libexec/fcitx5-wayland-launcher --reopen
```

### 6.4 Add Unikey

Open config:

```bash
fcitx5-configtool
```

In the GUI:

1. Select group: `Default`
2. Uncheck `Only Show Current Language`
3. Search `unikey`
4. Add `Unikey`
5. Keep `Keyboard - English (US)` first
6. Click `Apply`
7. Click `Close`

Final list:

```text
Keyboard - English (US)
Unikey
```

Switch input method:

```text
Ctrl + Space
```

## 7. Install Applications from `.deb` Files

Install these applications by downloading their `.deb` files and installing them with `apt`:

- Visual Studio Code
- Windsurf
- GitKraken

Example:

```bash
sudo apt install ./some-package.deb
```

## 8. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

After installing, restart the terminal or reload the shell configuration.

## 9. Install Gromit-MPX

Reference:

```text
https://github.com/bk138/gromit-mpx/tree/master
```

## 10. Copy Gromit-MPX Configuration

Copy the Gromit-MPX config file to the correct config location.

Run this from the directory that contains `gromit-mpx.cfg`.

```bash
mkdir -p ~/.config
cp gromit-mpx.cfg ~/.config/gromit-mpx.cfg
```
## 11. VPN config
Follow image in data/setup_vpn.png

Get username/password in this link https://account.network-auth.com/
Preshared Key for L2TP provided by Miyata san on the Teams

## 12. Others

Kiro has key keybinding (Ctrl+L) conflit with others, we need to hadd this (revise this parter later when on it)
```json
{
  "key": "ctrl+l",
  "command": "-expandLineSelection"
},
{
  "key": "ctrl+l",
  "command": "-expandLineSelection",
  "when": "textInputFocus"
},
{
  "key": "ctrl+l",
  "command": "-extension.vim_navigateCtrl",
  "when": "editorTextFocus && vim.active && vim.use<C-l> && !inDebugRepl"
},
{
  "key": "ctrl+l",
  "command": "kiroAgent.focusChatInput",
  "args": {"ensureSession": true}
}
```

Map jk to Esc for vim in Kiro, add this to global settings
```json
    {
      "before": ["j", "j"],
      "after": ["<Esc>"]
    }
```

Install Pylance on Kiro
[[https://github.com/kirodotdev/Kiro/issues/1318
](https://github.com/kirodotdev/Kiro/issues/1318#issuecomment-3665764492)](https://github.com/kirodotdev/Kiro/issues/1318#issuecomment-3665764492)
