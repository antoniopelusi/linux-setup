# 🎩 Fedora Post-Install Setup Guide

---

## 1. 📦 Package Setup

```sh
# Enable RPM Fusion Third Party Repositories:
sudo dnf -y install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Enable Flathub Third Party Repositories:
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Update AppStream metadata:
sudo dnf -y group upgrade core

# Update dnf packages:
sudo dnf -y update --refresh

# Update flatpak packages:
flatpak -y update

# Update the Firmware:
sudo fwupdmgr refresh --force
sudo fwupdmgr update

# Install dnf packages:
sudo dnf -y install \
  zsh vim-enhanced gcc-c++ python3-pip fuse-libs pandoc fastfetch 7zip-standalone-all \
  thunderbird chromium transmission inkscape audacity jupyterlab texstudio texlive-scheme-full \
  gnome-extensions-app gnome-tweaks papirus-icon-theme f42-backgrounds-gnome gpaste gnome-shell-extension-gpaste

# Install flatpak packages:
flatpak -y install flathub dev.zed.Zed
```

---

## 2. 📟 Install NVIDIA driver

> [!IMPORTANT]
> Proceed with this section only if your PC has an **NVIDIA GPU** and the **Secure Boot** enabled.
> 
> Check if your PC has an **NVIDIA GPU**:
> ```sh
> lspci | grep -iE "nvidia|vga|3d"
> ```
> 
> Check **Secure Boot** status:
> ```sh
> mokutil --sb-state
> ```

```sh
# Install these requirements:
sudo dnf -y install kmodtool akmods mokutil openssl

# Generate a default key:
sudo kmodgenca -a --force

# Enroll the key in MOK, insert a simple and short password and reboot the system:
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
systemctl reboot
```

On the next boot **MOK Management** is launched:
`Enroll MOK` -> `Continue` -> `Yes` -> Enter the *password* created earlier -> `Reboot`.

```sh
# Install the driver and the CUDA library and enable nvidia-modeset:
sudo dnf -y install akmod-nvidia xorg-x11-drv-nvidia-cuda libva-nvidia-driver
sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"

# Install full ffmpeg and Additional Codecs:
sudo dnf -y swap ffmpeg-free ffmpeg --allowerasing
sudo dnf -y update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin

# Enable OpenH264 for Firefox
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
```

> [!CAUTION]
> Check if the driver is built:
> ```sh
> modinfo -F version nvidia
> ```
> Do not reboot. Keep repeating until the driver version appears.

---

## 3. 🔲 Install CPU Hardware Codecs

> [!IMPORTANT]
> Only install the driver that corresponds to your **CPU manufacturer**.
> Run the following command to check the CPU manufacturer:
> ```sh
> lscpu | grep "Model name"
> ```

- For **Intel CPU**:
  ```sh
  sudo dnf -y swap libva-intel-media-driver intel-media-driver --allowerasing
  sudo dnf -y install libva-intel-driver
  ```
- For **AMD CPU**:
  ```sh
  sudo dnf -y swap mesa-va-drivers mesa-va-drivers-freeworld
  sudo dnf -y swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
  ```

---

## 4. ⚙️ Settings

```sh
# Download the required files:
curl -L \
  https://raw.githubusercontent.com/antoniopelusi/fedora-setup/main/files/AntonioPelusi.png -o ~/Downloads/AntonioPelusi.png \
  https://raw.githubusercontent.com/antoniopelusi/fedora-setup/main/files/firma.png -o ~/Templates/firma.png \
  https://raw.githubusercontent.com/antoniopelusi/fedora-setup/main/files/toolstab.config.json -o ~/Downloads/toolstab.config.json

# Set the Dock Favorite Apps:
gsettings set org.gnome.shell favorite-apps "[]"
gsettings set org.gnome.shell favorite-apps "[\
'org.mozilla.firefox.desktop', \
'net.thunderbird.Thunderbird.desktop', \
'org.gnome.Nautilus.desktop', \
'org.gnome.Ptyxis.desktop', \
'dev.zed.Zed.desktop']"

# Remove folders:
gsettings set org.gnome.desktop.app-folders folder-children "[]"

# Add minimize and maximize buttons to the windows:
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'

# Setup Night Light:
gsettings set org.gnome.settings-daemon.plugins.color night-light-schedule-automatic false
gsettings set org.gnome.settings-daemon.plugins.color night-light-schedule-from 0
gsettings set org.gnome.settings-daemon.plugins.color night-light-schedule-to 0
gsettings set org.gnome.settings-daemon.plugins.color night-light-temperature 4700

# Enable Show Battery Percentage:
gsettings set org.gnome.desktop.interface show-battery-percentage true

# Disable Automatic Screen Brightness, Dim Screen, Automatic Screen Blank and Automatic Suspend:
gsettings set org.gnome.settings-daemon.plugins.power ambient-enabled false
gsettings set org.gnome.settings-daemon.plugins.power idle-dim false
gsettings set org.gnome.desktop.session idle-delay 0
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'

# Set Right Alt as Compose Key:
gsettings set org.gnome.desktop.input-sources xkb-options "['compose:ralt']"

# Enable Automatic Date & Time and Automatic Time Zone:
timedatectl set-ntp true
gsettings set org.gnome.desktop.datetime automatic-timezone true

# Set 24-hour Time Format:
gsettings set org.gnome.desktop.interface clock-format '24h'

# Enable Week Day:
gsettings set org.gnome.desktop.interface clock-show-weekday true

# Set AntonioPelusi.png as Avatar and delete the .png file:
dbus-send --system --print-reply --dest=org.freedesktop.Accounts \
  /org/freedesktop/Accounts/User$(id -u) \
  org.freedesktop.Accounts.User.SetIconFile \
  string:"$HOME/Downloads/AntonioPelusi.png"
rm ~/Downloads/AntonioPelusi.png

# Disable NetworkManager-wait-online.service:
sudo systemctl disable NetworkManager-wait-online.service

# Change the default zoom level:
gsettings set org.gnome.nautilus.icon-view default-zoom-level small

# Create the Projects/ directory, change its icon and add it to the bookmarks:
mkdir ~/Projects
gio set ~/Projects metadata::custom-icon-name "folder-code"
echo "file://$HOME/Projects Projects" >> ~/.config/gtk-3.0/bookmarks

# Set Style to Dark:
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

# Set Accent Color to Slate:
gsettings set org.gnome.desktop.interface accent-color 'slate'

# Set Background to the dynamic version of f42 background:

gsettings set org.gnome.desktop.background picture-uri 'file:///usr/share/backgrounds/f42/default/f42.xml'
gsettings set org.gnome.desktop.background picture-uri-dark 'file:///usr/share/backgrounds/f42/default/f42.xml'

# Change Icon Pack:
gsettings set org.gnome.desktop.interface icon-theme 'Papirus-Dark'

# Install papirus-folders and apply the bluegrey folder color to Papirus-Dark icon theme:
curl -fsSL https://raw.githubusercontent.com/PapirusDevelopmentTeam/papirus-folders/master/install.sh | sh
papirus-folders -C bluegrey --theme Papirus-Dark

# Change DNS:
sudo nmcli con mod "$(nmcli -g NAME con show --active | head -1)" ipv4.dns "1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4"
sudo nmcli con mod "$(nmcli -g NAME con show --active | head -1)" ipv6.dns "2606:4700:4700::1111 2606:4700:4700::1001 2001:4860:4860::8888 2001:4860:4860::8844"
sudo nmcli con mod "$(nmcli -g NAME con show --active | head -1)" ipv4.ignore-auto-dns yes
sudo nmcli con mod "$(nmcli -g NAME con show --active | head -1)" ipv6.ignore-auto-dns yes
sudo nmcli con mod "$(nmcli -g NAME con show --active | head -1)" connection.dns-over-tls opportunistic
sudo nmcli con up "$(nmcli -g NAME con show --active | head -1)"
```

Edit the Device Name (also called hostname):
> [!WARNING]
> Insert a valid `<hostname>`.
```sh
sudo hostnamectl set-hostname "<hostname>"
```

Settings -> Display: manually adjust `Resolution` and `Refresh Rate`.

Settings -> Online Accounts: manually add `Google` Account.

## 5. 📱 Apps configuration

### 5.1 🦊 Firefox

Login into Firefox to automatically sync and install the following extensions:
- **GNOME Shell Integration**
- **Bitwarden Password Manager**
- **Dark Reader**
- **ToolsTab**
- **Adaptive Tab Bar Color**

Disable `Always offer to translate`.

Set `Bookmarks Toolbar` to `Always Show`.

Remove default bookmarks from the `Bookmarks Toolbar`.

Open all the bookmarks to initialize the favicons.

Login into **Bitwarden Password Manager** to get access to the other passwords.

In the **ToolsTab** startpage, press `Alt`+`Left Click` on the date and time tab to import `toolstab.config.json`.

Delete `toolstab.config.json`:
```sh
rm ~/Downloads/toolstab.config.json
```

In the **Dark Reader** settings, go to `Configure website toggling` and disable `Enabled by Default`.

Enable the **OpenH264** Plugin in Firefox's [**about:addons**](about:addons): -> **Plugins** -> `···` -> `Always Activate`.

### 5.2 📧 Thunderbird

Open **Thunderbird** and add the email accounts.

### 5.3 🐙 Configure Git + GitHub

```sh
# Configure Git:
git config --global user.name "Antonio Pelusi"
git config --global user.email "antoniopelusi2000@gmail.com"
git config --global core.editor vim

# Generate key for Github:
ssh-keygen -t ed25519 -C "antoniopelusi2000@gmail.com"
cat ~/.ssh/id_ed25519.pub
```

Copy the SSH key and [add it to GitHub](https://github.com/settings/ssh/new).

### 5.4 🐚 Configure Zsh

```sh
# Install Oh My Zsh:
CHSH=no RUNZSH=no sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" -- --unattended
chsh -s $(which zsh)
zsh

# install .zsh_functions:
git clone git@github.com:antoniopelusi/.zsh_functions.git ~/.zsh_functions
cd ~/.zsh_functions
make install
source ~/.zshrc
cd -
```
> [!TIP]
> Close and re-open the terminal to apply the shell change.

### 5.5 🧩 Gnome Shell Extensions

Install the following extensions from [extensions.gnome.org](https://extensions.gnome.org):
1. [**Alphabetical App Grid**](https://extensions.gnome.org/extension/4269/alphabetical-app-grid/)
2. [**Vitals**](https://extensions.gnome.org/extension/1460/vitals/)

```sh
# Disable Background Logo
gnome-extensions disable background-logo@fedorahosted.org

# Enable GPaste:
gnome-extensions enable GPaste@gnome-shell-extensions.gnome.org

# Configure Vitals:
dconf load /org/gnome/shell/extensions/vitals/ << EOF
[/]
hot-sensors=['_processor_usage_', '_memory_usage_', '_gpu#1_utilization_', '__network-rx_max__', '__temperature_avg__']
show-battery=true
show-gpu=true
EOF
gnome-extensions disable Vitals@CoreCoding.com
gnome-extensions enable Vitals@CoreCoding.com
```

### 5.6 📝 Zed

Press `Ctrl`+`Alt`+`,` and replace the content with the following settings:
```json
{
  // --- AI ---
  "edit_predictions": {
    "provider": "none",
  },

  // --- Git ---
  "git": {
    "inline_blame": {
      "enabled": false,
    },
  },
  "git_panel": { "button": false },

  // --- Toolbar ---
  "toolbar": {
    "selections_menu": false,
  },
  "gutter": {
    "runnables": false,
  },

  // --- Tabs ---
  "tabs": {
    "git_status": true,
    "file_icons": true,
  },

  // --- Panels ---
  "outline_panel": { "button": false },
  "collaboration_panel": { "button": false },
  "search": { "button": false },
  "diagnostics": { "button": false },
  "debugger": { "button": false },
  "notification_panel": { "button": false },
  "global_lsp_settings": { "button": false },

  // --- Privacy ---
  "telemetry": {
    "diagnostics": false,
    "metrics": false,
  },

  // --- Session ---
  "session": {
    "trust_all_worktrees": true,
  },
}
```

Login using the **Github Account**.

Press `Ctrl`+`Alt`+`B`, then `Ctrl`+`Alt`+`C` and configure **Github Copilot Chat**.

### 5.7 📄 TeXstudio

Options -> Configure TeXstudio -> General: set `Style` to `Adwaita Dark (txs)`

Options -> Configure TeXstudio -> Build: set `Default Bibliography Tool` to `Biber`

### 5.8 📊 JupyterLab

Enable `Dark Reader` Extension for JupyterLab.

---

## 9. 🔁 Cleanup and Final Reboot

```sh
# Cleanup unused dependencies:
sudo dnf -y autoremove
sudo dnf -y clean all
flatpak -y uninstall --unused

# Reboot the system:
systemctl reboot
```

---

## ✅ Done!
