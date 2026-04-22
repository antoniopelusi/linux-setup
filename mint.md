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
gsettings set org.cinnamon.desktop.background primary-color '#2a2a2e'

# Disable lock screen
gsettings set org.cinnamon.desktop.lockdown disable-lock-screen true
gsettings set org.cinnamon.desktop.screensaver lock-enabled false
gsettings set org.cinnamon.desktop.screensaver lock-on-suspend false

# Disable suspension
gsettings set org.cinnamon.settings-daemon.plugins.power sleep-display-ac 0
gsettings set org.cinnamon.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
gsettings set org.cinnamon.settings-daemon.plugins.power sleep-display-battery 0
gsettings set org.cinnamon.settings-daemon.plugins.power sleep-inactive-battery-timeout
0
gsettings set org.cinnamon.desktop.session idle-delay 0
```

Set the following pinned app order:
1. Firefox
2. Files
3. Terminal

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
