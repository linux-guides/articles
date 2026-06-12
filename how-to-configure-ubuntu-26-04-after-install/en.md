Recently, Ubuntu 26.04 was released. This is a Long Term Support (LTS) version of the distribution that will be the main one for the next several years. There weren't many changes here. The interface was slightly modified, the GNOME version was updated, several default applications were replaced, the firmware distribution format was changed, and the Software &amp; Updates utility was removed.

In this article, we will look at how to configure Ubuntu 26.04 after installation. We will cover how to configure new system components, add Flatpak support, install commonly used applications, restore Software &amp; Updates, and configure the GNOME environment.

## Setting Up Ubuntu 26.04 After Installation

Where possible, there will be not only instructions on how to perform an action in the interface, but also commands, so that everything can be done conveniently and quickly, or even to write an automation script. I will be describing the setup of Ubuntu on a desktop PC for everyday use. We will also skip the section about installing drivers.

### Step 1. Configuring Desktop Background

Let's start by changing the desktop background to something more beautiful than the default image. To do this, open the context menu on the desktop and select **Change Background**:

![](configure-ubuntu-2604.png)

In the window that opens, scroll down a bit and select an image you like in the Background section. For example, this one:

![](configure-ubuntu-2604-1.png)

To select the same image in the terminal, run the following commands:

```
gsettings set org.gnome.desktop.background picture-uri 'file:///usr/share/backgrounds/mendhak-Red_Acer.jpg'
```

```
gsettings set org.gnome.desktop.background picture-uri-dark 'file:///usr/share/backgrounds/mendhak-Red_Acer.jpg'
```

```
gsettings set org.gnome.desktop.screensaver picture-uri 'file:///usr/share/backgrounds/mendhak-Red_Acer.jpg'
```

### Step 2. Enable Dark Theme

In this same window, you can enable dark style and choose an accent color. These settings are located in the upper part of the window:

![](configure-ubuntu-2604-2.png)

To enable dark style using the terminal, execute the following commands:

```
gsettings set org.gnome.desktop.interface color-scheme prefer-dark
```

```
gsettings set org.gnome.desktop.interface gtk-theme Yaru-dark
```

```
gsettings set org.gnome.desktop.interface icon-theme Yaru-dark
```

### Step 3. System Update

During the time that has passed since the release of the image you used for installation, new versions of packages with bug fixes may have been released, so before installing anything, it is recommended to update the system. To do this, first update the package list:

```
sudo apt update
```

![](configure-ubuntu-2604-3.png)

Then update the packages themselves to the new version:

```
sudo apt upgrade
```

To update snap packages, use the following command:

```
sudo snap refresh
```

The following command will reboot the computer if necessary:

```
[ -f /var/run/reboot-required ] &amp;&amp; sudo reboot
```

### Step 4. Installing Software & Updates

The Software & Updates utility, which was previously used in Ubuntu for managing repositories, is no longer shipped by default. However, the universe, multiverse, and restricted repositories are already enabled by default, even if you didn't choose to install proprietary software during distribution installation. If you still want to install it, run the following command:

```
sudo apt install software-properties-gtk
```

After that, the utility can be found in the main menu.

### Step 5. Switching to amd64v3 Packages

In the Ubuntu 26.04 repositories, there are now package versions for the amd64v3 architecture. This is the same x86_64, but it uses new instruction sets that have appeared in modern processors. This is unlikely to provide a significant performance boost, but it will be useful for tasks that require intensive computations. First, check if your processor supports this architecture. The following command should show x86-64-v3 with a supported note:

```
/lib64/ld-linux-x86-64.so.2 --help | grep  x86-64
```

![](configure-ubuntu-2604-4.png)

To enable downloading packages with the new architecture, run the following command:

```
echo 'APT::Architecture-Variants "amd64v3";' | sudo tee /etc/apt/apt.conf.d/99amd64v3
```

![](configure-ubuntu-2604-5.png)

This command will create a 99amd64v3 file in the /etc/apt/apt.conf.d/ directory. Now you need to update the system once again:

```
sudo apt update
```

```
sudo apt upgrade
```

![](configure-ubuntu-2604-6.png)

If something goes wrong and you want to switch back to the default version, delete the /etc/apt/apt.conf.d/99amd64v3 file and repeat the update:

```
sudo rm /etc/apt/apt.conf.d/99amd64v3
```

### Step 6. Removing Unnecessary Firmware

Since Canonical developers have split the firmware package into 17 different packages by vendors, you can now remove the firmware you don't need. You can view the list of all packages with the command:

```
apt list --installed | grep firmware
```

![](configure-ubuntu-2604-7.png)

Now you can remove packages for manufacturers whose hardware you don't use. Here's a command that will remove firmware for manufacturers that are rarely found on home computers. But before executing it, check the list to make sure you don't remove something you need:

```
sudo apt purge linux-firmware-qualcomm-graphics linux-firmware-qualcomm-misc linux-firmware-qualcomm-wireless linux-firmware-mediatek linux-firmware-marvell-wireless linux-firmware-marvell-prestera linux-firmware-mellanox-spectrum linux-firmware-netronome
```

### Step 7. Default Editor

In Linux, there is an editor command that opens the default text editor in the terminal. This command is used in visudo and similar commands. In Ubuntu, nano is used by default. To change the editor to Vi, execute:

```
sudo update-alternatives --config editor
```

Here, select the desired editor by pressing the corresponding number.

### Step 8. Configuring sudo-rs

Starting from version 25.10, Ubuntu uses sudo-rs. This is an alternative implementation of sudo, written in Rust. The previous version of sudo is also present in the system, and it is called sudo.ws. You can verify that the Rust version is being used by running:

```
sudo --version
```

![](configure-ubuntu-2604-8.png)

In this version, the developers have slightly changed the default behavior. Now sudo will display asterisks when entering a password. If you are accustomed to the old style, when password entry is completely hidden, you need to add the following line at the end of the /etc/sudoers file or in a separate file in /etc/sudoers.d. For example:

```
sudo visudo -f /etc/sudoers.d/pwfeedback
```

```
Defaults !pwfeedback
```

![](configure-ubuntu-2604-9.png)

To save, you need to use **:w!**, since this file is marked as Read Only. After this, asterisks will no longer be displayed. If you want to switch back to GNU sudo, you can use the update-alternatives script for this:

```
sudo update-alternatives --config sudo
```

After this, you will need to select the appropriate number:

![](configure-ubuntu-2604-12.png)

### Step 9. Configuring Coreutils

In Ubuntu 26.04, for some utilities from coreutils, alternatives written in Rust from the uutils package are used. Only cp, mv, and rm will be used from the coreutils suite, because their Rust alternatives still have several unresolved issues. You can verify this by running the following commands:

```
ls --version
```

```
cp --version
```

![](configure-ubuntu-2604-10.png)

But the versions of utilities from the coreutils package are still present in the system. In 95% of cases, you won't notice the difference. However, if you still need to use the GNU version for any of the utilities, they are all located in the /usr/bin directory with the gnu prefix:

![](configure-ubuntu-2604-11.png)

In fact, there are currently several packages with coreutils:

- **coreutils-from-uutils** - uutils, contains binary files like /usr/bin/ls
- **gnu-coreutils** - original coreutils, contains binary files with the gnu prefix, for example: /usr/bin/gnuls
- **coreutils-from-gnu** - original coreutils, contains binary files without a prefix, for example /usr/bin/ls

If you want to use coreutils from GNU as it was before, you need to install the coreutils-from-gnu package. The update-alternatives option won't work here. To do this, you should first configure a lower priority for the uutils package:

```
sudo vi /etc/apt/preferences.d/uutils
```

```
Package: coreutils-from-uutils
Pin: release a=*
Pin-Priority: -10
```

And then install coreutils from GNU:

```
sudo apt install coreutils-from-gnu coreutils-from-uutils- --allow-remove-essential
```

Do this only if you need it. In most cases, it's not worth replacing uutils back.

### Step 10. Configuring Active Corners

In GNOME, you can open the Overview screen by moving the cursor to the top-left corner. To enable this, turn on the **Hot Corner** option on the **Multitasking** tab:

![](configure-ubuntu-2604-13.png)

Or run the following command:

```
gsettings set org.gnome.desktop.interface enable-hot-corners true
```

There is also an **Active Screen Edges** option here, but you don't need to enable it because Ubuntu already uses its own Tiling Assistant extension for this purpose.

### Step 11. Configuring Window Switching with Alt+Tab

By default, when using Alt+Tab, applications will switch between all workspaces. To make them switch within a single workspace, select the **Include apps from current workspace only** value for the **App Switching** parameter on the **Multitasking** tab:

![](configure-ubuntu-2604-14.png)

Or execute the commands:

```
gsettings set org.gnome.shell.extensions.dash-to-dock isolate-workspaces true
```

```
gsettings set org.gnome.shell.extensions.tiling-assistant tiling-popup-all-workspace false
```

### Step 12. Hiding the Home Folder Shortcut

You can hide the home folder shortcut on the **Ubuntu Desktop** tab. To do this, set the **Show Desktop Folder** switch to the disabled position:

![](configure-ubuntu-2604-15.png)

You can also do this by running the command:

```
gsettings set org.gnome.shell.extensions.ding show-home false
```

### Step 13. Configuring Nautilus

To be able to create new files in the Nautilus context menu, you need to create template files in the Templates directory. For example, for the most common file types:

```
touch ~/Templates/new_file.md
```

```
touch ~/Templates/new_file.txt
```

```
echo -n '#!/bin/bash' &gt; ~/Templates/new_script.sh
```

The result will look like this:

![](configure-ubuntu-2604-16.png)

To make Nautilus sort by type by default, with folders at the top, run the following command:

```
gsettings set org.gnome.nautilus.preferences default-sort-order 'type'
```

### Step 14. Disabling Sleep Mode

You can disable the computer from entering sleep mode after a certain period of inactivity on the **Power** tab using the **Automatic Suspend** option. The same can be done using the command:

```
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
```

Since this setting only applies to the current user, you need to do the same for gdm so that the computer doesn't enter sleep mode on the login screen:

```
sudo -u gdm dbus-run-session gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-timeout 0
```

### Step 15. Disabling Screen Lock

Screen lock is disabled in the settings, on the **Privacy &amp; Security** tab, in the **Screen Lock** section, using the **Automatic Screen Lock** option:

![](configure-ubuntu-2604-17.png)

The same can be done by executing:

```
gsettings set org.gnome.desktop.screensaver lock-enabled false
```

### Step 16. Automatic Cart Cleanup

To automatically clean up the trash by deleting files that have been there for more than 30 days, go to the **Privacy &amp; Security** tab, select **File History &amp; Trash**, and in the **Trash &amp; Temporary Files** section, enable the **Automatically Delete Trash Content** option:

![](configure-ubuntu-2604-18.png)

Or run the following command:

```
gsettings set org.gnome.desktop.privacy remove-old-trash-files true
```

### Step 17. Configuring Search Sources

To remove unnecessary items from the search located on the "Overview" screen, open the **Search** tab in settings and disable all providers you don't need in the **Search Results** list:

![](configure-ubuntu-2604-19.png)

In gsettings, all providers are enabled by default, and to disable them, you need to add them to exclusions:

```
gsettings set org.gnome.desktop.search-providers disabled " ['org.gnome.seahorse.Application.desktop', 'org.gnome.clocks.desktop', 'org.gnome.Characters.desktop', 'org.gnome.Calculator.desktop']"
```

The result will look like this:

![](configure-ubuntu-2604-20.png)

### Step 18. Installing Extensions

To install GNOME extensions, it's convenient to use GNOME Extension Manager. To install it, execute the following command:

```
sudo apt install gnome-shell-extension-manager
```

The application has two tabs. On the first one, you can view and configure installed extensions, and on the second one, you can browse the catalog of extensions available for installation:

![](configure-ubuntu-2604-21.png)

To install an extension, type its name in the search, and then click the **Install** button next to its name:

![](configure-ubuntu-2604-22.png)

Here are a few extensions that make working in GNOME more convenient:

- **Logo Menu** - adds a menu with the distribution logo in addition to the main GNOME menu. A useful feature - there is an ability to force quit an application, as you could previously do with xkill.
- **Caffeine** - prevents the screen from turning off when you're watching a video.
- **Hide Cursor** - hides the cursor after a few seconds of inactivity.
- **Just Perfection** - contains many settings for the desktop environment.
- **Copyous** - a clipboard manager that can open a dialog next to the cursor.
- **RX Layout Switcher** - enables keyboard layout switching using Alt+Shift.

Since Ubuntu 26.04 has just been released, developers may not have released updates for some extensions, although they are fully compatible. To install incompatible extensions, you need to disable compatibility checking using the following command:

```
gsettings set org.gnome.shell disable-extension-version-validation true
```

Be careful with this step. Do not install extensions that haven't been updated for many years, as this may break your desktop environment. This feature should be used temporarily, only for extensions that are highly likely to work.

### Step 19. Configuring Just Perfection

Using the Just Perfection extension, you can replace a number of different extensions for customizing the desktop appearance. Here you can hide interface elements, move their placement, change their behavior and size. The extension settings can be opened in **Extension Manager**.

![](configure-ubuntu-2604-24.png)

There are several profiles with default parameters, as well as the ability to configure everything independently in the Custom profile. Let's look at how to configure several parameters in the terminal. To immediately open the desktop instead of the "Overview" screen, execute the following command:

```
gsettings --schemadir ~/.local/share/gnome-shell/extensions/just-perfection-desktop\@just-perfection/schemas/ set org.gnome.shell.extensions.just-perfection startup-status 0
```

This command will move the panel to the bottom of the screen:

```
gsettings --schemadir ~/.local/share/gnome-shell/extensions/just-perfection-desktop\@just-perfection/schemas/ set org.gnome.shell.extensions.just-perfection top-panel-position 1
```

And this one will make notifications display at the bottom and centered:

```
gsettings --schemadir ~/.local/share/gnome-shell/extensions/just-perfection-desktop\@just-perfection/schemas/ set org.gnome.shell.extensions.just-perfection notification-banner-position 4
```

### Step 20. Installing Flatpak and Bazaar

By default, Ubuntu only supports Snap packages. To add Flatpak support, run the following command:

```
sudo apt install flatpak
```

After that, add the Flathub repository:

```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

After that, you should restart your computer, and you can install Flatpak packages in the terminal. For managing such packages in a graphical interface, it's better to use the Bazaar software center. To install it, run:

```
flatpak install flathub io.github.kolunmi.Bazaar
```

The issue with [fusermount and AppArmor](https://bugs.launchpad.net/ubuntu/+source/apparmor/+bug/2130388), which prevented the program from working in Ubuntu 25.10, has not been fixed yet, so to make everything work, you need to disable the AppArmor profile for fusermount:

```
sudo ln -s /etc/apparmor.d/fusermount3 /etc/apparmor.d/disable/
```

```
sudo apparmor_parser -R /etc/apparmor.d/fusermount3
```

The program looks like this:

![](configure-ubuntu-2604-23.png)

You should also install Flatseal for managing permissions for Flatpak packages:

```
flatpak install flathub com.github.tchx84.Flatseal
```

### Step 21. Installing Refine

To configure additional GNOME settings, you can use the Refine utility. It can be installed using Bazaar or by running the following command:

```
flatpak install flathub page.tesk.Refine
```

Using the program, you can restore support for pasting text by clicking the mouse wheel on the **Mouse &amp; Touchpad** tab:

![](configure-ubuntu-2604-25.png)

This feature was [disabled](https://gitlab.gnome.org/GNOME/gsettings-desktop-schemas/-/merge_requests/119?ref=itsfoss.com) by default in GNOME 50. This can also be done with the command:

```
gsettings set org.gnome.desktop.interface gtk-enable-primary-paste true
```

Additionally, on the Power tab, you can enable the display of the power button on the lock screen:

![](configure-ubuntu-2604-26.png)

This can also be done with the command:

```
gsettings set org.gnome.desktop.screensaver restart-enabled true
```

### Step 22. Configuring AppImage

To run old AppImage images that require fuse2 to work, you need to install this library:

```
sudo apt install libfuse2t64
```

For centralized management of AppImage applications and creating shortcuts for them in the menu, you can use the GearLever application.

```
flatpak install flathub it.mijorus.gearlever
```

The application stores all your AppImage applications in the ~/Appimages directory. After you add an AppImage file to the directory, you will be able to open it from the application's main menu or find its shortcut in the GNOME menu.

![](configure-ubuntu-2604-27.png)

### Step 23. Click to Minimize in Dash

In the Unity shell, there was previously the ability to minimize windows by clicking on the application icon in the Dash panel. To enable this feature now, execute the following command:

```
gsettings set org.gnome.shell.extensions.dash-to-dock click-action minimize
```

### Step 24. Installing Multimedia Codecs

If during Ubuntu installation you did not check that multimedia codecs need to be installed, and now you need them, they can be installed with the command:

```
sudo apt install ubuntu-restricted-extras
```

Also, Ubuntu does not ship by default with a plugin for opening images in HEIC format (photos taken on iPhone). If you need to open such images, run the following command to install the package:

```
sudo apt install libheif-plugin-libde265
```

### Step 25. Switching Keyboard Layout With Alt+Shift

### Step 25. Switching Keyboard Layout With Alt+Shift

For quite a long time now, Ubuntu and GNOME have been using Super + Space to switch keyboard layouts. If you want to switch layouts using Alt+Shift, you need to use the RX Layout Switcher extension. At the time of writing this article, it doesn't officially support GNOME 50, but it works. Most likely, within a few weeks, a version with updated metadata will be released and support will appear. In the meantime, you should disable extension compatibility checking, as shown in step 18. After that, the extension will work:

![](configure-ubuntu-2604-28.png)

If you need to add keyboard layouts, this can be done in the settings, on the **Keyboard** tab, in the **Input Sources** section:

![](configure-ubuntu-2604-29.png)

Since the extension for switching layout on the login screen doesn't work, you may need the [Primary Input on LockScreen](https://extensions.gnome.org/extension/4727/primary-input-on-lockscreen/) extension here, which will change the keyboard layout on the login screen to the primary one.

### Step 26. Installing a Browser

By default, Ubuntu comes with Firefox. If you're used to Chrome-based browsers, you can install Brave or Chromium. Execute the following commands to install Brave from the developers' repository:

```
sudo apt install curl
```

```
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
```

```
sudo curl -fsSLo /etc/apt/sources.list.d/brave-browser-release.sources https://brave-browser-apt-release.s3.brave.com/brave-browser.sources
```

```
sudo apt update
```

```
sudo apt install brave-browser
```

If you want to use Chromium, you can install it from a snap package or use the PPA repository from xtradeb:

```
sudo add-apt-repository ppa:xtradeb/apps
```

```
sudo apt update
```

```
sudo apt install chromium
```

Chromium recently added support for vertical tabs, so if that's what you needed, now it's available:

![](configure-ubuntu-2604-30.png)

### Step 27. Installing Docker

For setting up development environments or running applications with a web interface, Docker is often used. To install it on Ubuntu, first add the developers' repository:

```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

```
sudo tee /etc/apt/sources.list.d/docker.sources &lt;&lt;EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release &amp;&amp; echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Then update the package list and install the application components:

```
sudo apt update
```

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Next, all that remains is to add the user to the docker group and add the docker service to autostart:

```
sudo usermod -aG docker $USER
```

```
sudo systemctl enable --now docker
```

After this, you should log into the system again. We used the developers' repository so that you can receive application updates as soon as they are released.

### Step 28. Installing Necessary Programs

Right after installation, Ubuntu doesn't have many programs that might be needed. Here are some programs that can be installed:

- **VLC** - one of the best video players, better to install from Flathub, since the program already includes all codecs there
- **Gapless** - audio player
- **Obsidian** - note-taking app in Markdown
- **GIMP** - image editor
- **Transmission** - torrent client
- **KeePassXC** - password manager
- **LibreOffice** - office suite

Here are the commands for installation:

```
flatpak install flathub org.videolan.VLC
```

```
flatpak install flathub com.github.neithern.g4music
```

```
flatpak install flathub md.obsidian.Obsidian
```

```
sudo apt install gimp transmission libreoffice keepassxc
```

You can also install some utilities that will be useful to you:

- **curl** - downloading files in terminal
- **vim** - improved version of vi text editor
- **ncdu** - allows you to find the largest files in the file system and free up disk space
- **git** - version control system
- **rg** - search by file contents instead of grep
- **htop** - task manager in terminal
- **unrar** - utility for unpacking rar
- **p7zip** - utility for unpacking p7zip
- **btop** - view list of running processes in terminal
- **fdfind** - fast search by file names
- **fzf** - fuzzy live search by file names

```
sudo apt install curl vim ncdu git ripgrep htop unrar p7zip btop fd-find fzf
```

## Wrapping Up

In this article, we looked at how to configure Ubuntu 26.04 after installation. Many steps are optional and only show how to restore what was in Ubuntu before, or how to improve the GNOME interface. What do you do to configure your system? Let's collect other useful commands and settings that didn't make it into the article in the comments.


