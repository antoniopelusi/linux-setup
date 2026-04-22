# 🍃 Mint Post-Install Setup Guide

---

## 1. 📦 Package Setup

```sh
# Update apt packages:
sudo apt update
sudo apt -y upgrade
sudo apt -y dist-upgrade

# Install dnf packages:
sudo apt -y install vim zsh git
```

---

## 2. ⚙️ Settings

```sh
# Set background:
gsettings set org.cinnamon.desktop.background picture-options 'none'
gsettings set org.cinnamon.desktop.background color-shading-type 'solid'
gsettings set org.cinnamon.desktop.background primary-color '#2f2f2f'

# Set natural scroll:
gsettings set org.cinnamon.desktop.peripherals.touchpad natural-scroll true

```

### 4. 🦊 Configure Firefox

Install the following extension:
- [**uBlock Origin**](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)

### 3. 🐚 Configure Zsh

```sh
# Install Oh My Zsh:
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
> [!TIP]
> Close and re-open the terminal to apply the shell change.

---

## 4. 🔁 Cleanup and Final Reboot

```sh
# Cleanup unused dependencies:
sudo apt -y autoremove

# Reboot the system:
systemctl reboot
```

---

## ✅ Done!
