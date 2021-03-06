My Model: 20KHCTO1WW

# Setting up Arch Linux on a Lenovo X1 Carbon (6th Gen 2018)

Why? https://medium.com/@shazow/my-computer-is-my-home-5a587dcc1d76

## Additional resources
* https://github.com/ejmg/an-idiots-guide-to-installing-arch-on-a-lenovo-carbon-x1-gen-6
* https://kozikow.com/2016/06/03/installing-and-configuring-arch-linux-on-thinkpad-x1-carbon/
* Not x1 carbon: https://ticki.github.io/blog/setting-up-archlinux-on-a-lenovo-yoga/
* Privacy guide: https://theprivacyguide1.github.io/linux_hardening_guide.html

## List of Issues

* Resolved: With the 20KG (NFC edition), the touchpad and trackpoint do not work together - fixed with Linux kernel 4.17.
* Resolved: Si03 vs S3 Sleep power drain - fixed with BIOS v1.30.
    * Lenovo implemented S0i3 sleep instead of the normal S3 sleep (suspend to RAM): https://news.ycombinator.com/item?id=17551286
    * This is for "Windows Modern Standby", which allows the device to wake-up to perform background activities.
* Thermal throttling issues with BIOS v1.25-v1.30(+?). (Workaround tool available.)
    * https://www.notebookcheck.net/Lenovo-admits-ThinkPad-CPU-throttling-problem-when-running-Linux-fix-in-development.435549.0.html
* With the 4G-WAN edition, the mobile modem does not work with Linux. This is unlikely to be resolved at all.



## Upgrade BIOS

### From Windows

For the first one-time set up.


### From Linux

See Arch Linux Wiki on instructions on how to create a USB disk image. Note that this *created* disk image has a FLASH folder which can set a custom logo.

* Download page: https://pcsupport.lenovo.com/gb/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-6th-gen-type-20kh-20kg/downloads/ds502281
* If you are using fwupdmgr and it says "https://github.com/fwupd/fwupd/issues/1454"
Both options under "Security -> UEFI BIOS Update Option" must be enabled:
    * Flash BIOS Updating by End-Users
    * Windows UEFI Firmware Update
    * https://github.com/fwupd/fwupd/issues/1454
* If you are following fwupmgr instructions, then distinguish between cab files:
https://old.reddit.com/r/thinkpad/comments/ejgb6a/howto_flash_the_nonabsolute_persistence_module/
* https://200ok.ch/posts/2018-09-26_X1_carbon_6th_gen_about_50_percent_slower_on_Linux.html
* `sudo dmidecode -t bios | grep Version` to get BIOS version



### Secure Boot - a few options

Basically, just set an Admin and HDD password.

* SSD SED
    * "No" access to data from rest
* + Secure Boot
    * Prevent others from running tools without CMOS battery clear
* OR + Admin password
    * Prevent others from running tools without CMOS battery clear

There is no value to Secure Boot.

* LUKS?
    * If you don't trust hardware. But this needs Secure Boot to prevent Evil Maid.

* TPM:
    * PIN or no password on boot. Not sure if works with SED.
    * Needs Secure Boot.

## Power

TEST: Card Reader.
TEST: Thunderbolt BIOS assist mode.
TEST: Undervolt CPU - with thinkpad-power-fix(name?) (GitHub)

s-tui for cur power consumption?

* TODO: Disable card reader due to bug?
* TODO: Thunderbolt BIOS assist mode save power (while in sleep)?
* Suspend Issues?
* Undervolt CPU - stress test performance.
* Which tool to set battery limits to preserve battery life. How-to automatically turn this on and off?
* Power management throttling issues?
* Throttling issues?
* TLP to tweak, and `powertop` to monitor
    * https://github.com/linrunner/TLP/blob/master/default
    * https://linrunner.de/en/tlp/docs/tlp-linux-advanced-power-management.html
    * tlp-stat
    * tpacpi-bat (optional dependency of tlp)
    * Use 50-60% thresholds
        * 65-75% is optimum: https://www.youtube.com/watch?v=AF2O4l1JprI
        * thinkpad says 40-50: https://support.lenovo.com/us/en/solutions/ht078208
    * tlp fullcharge for one-off max-charge
    * https://linrunner.de/en/tlp/docs/
* https://github.com/teleshoes/tpacpi-bat

* TODO: Turn off Power LED:

echo "0" | sudo tee "/sys/class/leds/tpacpi::power/brightness"

* TODO: Systemd suspend-then-hibernate. Enable Swap only when hibernating.
* Alternative: Hibernate on low battery: /etc/udev/rules.d/
* Look through: https://wiki.archlinux.org/index.php/Power_management

* enabling panel self-refresh with i915 module option enable_psr=1, to save a bit of power. May be problematic on older hardware/kernels. Use 'sudo systool -vm i915' to check this option value in case your distro already has it enabled

* https://www.reddit.com/r/thinkpad/comments/alol03/tips_on_decreasing_power_consumption_under_linux/
    * 3W difference between lowest and highest brightness

## Arch Linux Install

* UEFI - for faster boot.
* No disk encryption - using hardware disk encryption.

I document the following differences to the installation guide:

* When setting the vconsole font, use this temporary HiDPI font
```
setfont latarcyrheb-sun32
```

* When installing ext4
```
# https://wiki.archlinux.org/index.php/ext4#Bytes-per-inode_ratio
# Based on existing usage (default is -i 16384)
# https://wiki.archlinux.org/index.php/ext4#Reserved_blocks
# This is *not* reduced as SSDs do not like to be nearly full (i.e. use default 5%)
mkfs.ext4 -i 65536 -L archroot /dev/nvme0n1p2
```

* When doing the pacstrap
```
pacstrap base base-devel linux linux-lts linux-firmware vim dhcpcd
```
(note: base-devel untested)

After the `arch-chroot`:

* Install beautiful vconsole HiDPI font
    ```
    export LANG=en_GB.UTF-8
    pacman -Syu --needed terminus-font
    ```
    Set vconsole font in `etc/vconsole.conf`
        ```
        KEYMAP=uk
        FONT=ter-i32b
        ```

* Modify `/etc/fstab`
    * Replace relatime with `noatime` for (slightly) improved performance when reading files.
        * Not set the last access time
    * Add `lazytime` for (slightly) improved performance
        * Batch all file time updates to every minute

* Install `efistub` as the "bootloader"
    * Install the microcode first.
    * `pacman -S efibootmgr`.
    * Create a script `/root/addefi.sh`.
        ```
        efibootmgr --disk /dev/nvme0n1 \
            --part 1 \
            --create \
            --label "Arch Linux" \
            --loader '\vmlinuz-linux' \
            --unicode 'root=PARTUUID=XXX rw nowatchdog initrd=\intel-ucode.img initrd=\initramfs-linux.img' \
            --verbose
        ```
        (Get PARTUUID from `blkid`)
    * TODO: Add similar for recovery, linux-lts and linux-lts recovery.
    * TODO: "quiet splash" boot

    * `nowatchdog` - https://wiki.archlinux.org/index.php/Improving_performance#Watchdogs
      Improve performance?

## System Install

* TODO: zram (1G of Swap might use 333MB ram)
* TODO: sudo systemctl restart iwd - on unplug Ethernet adapter
* TODO: Shutdown on failed login: https://cowboyprogrammer.org/2016/09/reboot_machine_on_wrong_password/
* TODO: Randomise MAC addresses:
    * https://wiki.archlinux.org/index.php/MAC_address_spoofing
    * Requires randomising hostname?
    * Additional reading: https://insanity.industries/post/random-mac-addresses-for-privacy/
* TODO: Use more secure DNS client


* Pacmatic: `pacmatic python-html2text`
* Man pages `man`
    * Run `mandb` one-off to build the search for `man -k`
* `htop lsof`
* Use `iwd` for WiFi.
    * Gives commands `iwctl` and `iwmon`, with systemd service `iwd`
    * Install `crda` and restart.
        * `iw reg set GB` (temporary)
        * Edit `/etc/conf.d/wireless-regdom` for a permanent solution.
    *`systemctl enable --now iwd`.
    * Use `iwctl` to manage connections.
    * Note: iwd solves persistence problem with wpa_supplicant (i.e. without additional software `wpa_supplicant` forgets your network), and is simpler to use.
    * Note: Do not use at the same time as NetworkManager service as they conflict.
* Use `rfkill` to manage network connections
* Edit `/etc/pacman.conf`
    * Uncomment `Color` and `TotalDownload`
* Install `pacman-contrib`
    * Use `checkupdates` to do exactly that
    * Use `rankmirrors` to sort mirrors automatically
    * Use `pacsearch` to search
    * Enables `pacdiff` to be used when updating files after an upgrade
* Install the auto clean scripts
    * Install reflector and the TODO: the script from archwiki mirrorlist-upgrade.hook
    * Remove cache of uninstalled packages
    `sudoedit /etc/pacman.d/hooks/paccache-remove.hook`
    ```
    [Trigger]
    Operation = Remove
    Type = Package
    Target = *

    [Action]
    Description = Removing package cache for uninstalled packages...
    When = PostTransaction
    Exec = /usr/bin/paccache -ruk0
    ```
    * Remove cache of old packages
    `sudoedit /etc/pacman.d/hooks/paccache-upgrade.hook`
    ```
    [Trigger]
    Operation = Upgrade
    Type = Package
    Target = *

    [Action]
    Description = Removing old cached packages...
    When = PostTransaction
    Exec = /usr/bin/paccache -rk3
    ```
* Enable the weekly SSD trim:
    * `sudo systemctl enable --now fstrim.timer`
* Install `tlp`
    * https://wiki.archlinux.org/index.php/TLP
    * Install `acpi_call ethtool smartmontools x86_energy_perf_policy lsb-release`
    * `systemctl enable --now tlp`
* Install drivers: `mesa vulkan-intel libva-intel-driver`
    * (`libva-intel-driver` recommended for X1C6 rather than `intel-media-bridge` - check wiki)
    * Install `libva-utils` and check with `vainfo` (basically check output is several lines)
    * Configure Intel modprobes
        * Edit `/etc/modprobe.d/i915.conf`
        ```
        # Load updated firmware
        options i915 enable_guc=2

        # Compress framebuffer
        options i915 enable_fbc=1

        # Show boot logo
        options i915 fastboot=1
        ```
        * https://wiki.archlinux.org/index.php/Intel_graphics#Module-based_options
* Add sudo (should be more limited but...)
    * sudo visudo -f /etc/sudoers.d/adm
    ```
    %adm ALL=(ALL:ALL) ALL
    ```
* `sudoedit /etc/profile`, and alter the default umask `umask 027`
* Install firmware updater `sudo pacmatic -S --needed fwupd`
    * Check with `sudo fwupdmgr get-updates`
    * Update with `sudo fwupdmgr update`
        * Note: there is a Thunderbolt security update - follow instructions!!!
        * TODO: There is also an NVMe update
* Install throttling fix.
    * `sudo pacmatic -S --needed throttled`
    * `sudo systemctl enable --now lenovo_fix.service`
    * TODO: undervolting.

* TODO: Look into https://old.reddit.com/r/linux/comments/e6qupr/new_linux_vulnerability_lets_attackers_hijack_vpn/?st=k3w8hhwm&sh=dcf5e072
* TODO: Look into bonding ethernet and wifi

* Note: Reboot to UEFI `systemctl reboot --firmware-setup`

### Deprecated

* Grub faster & boot menu: https://wiki.archlinux.org/index.php/GRUB#Dual-booting
    * Press <Esc> to bring up Grub menu
    * /etc/default/grub
        * GRUB_DEFAULT=saved  (default entry number 0-index)
        * GRUB_TIMEOUT=0  (time to boot default, try 5.)
        * GRUB_HIDDEN_TIMEOUT=1
        * GRUB_HIDDEN_TIMEOUT_QUIET=true
        * GRUB_DISABLE_SUBMENU=y
    * /etc/grub.d/40_custom
        ```
        menuentry "Shutdown Laptop" {
             echo "Shutting down..."
             halt
        }

        menuentry "Restart Laptop" {
             echo "Restarting..."
             reboot
        }

        menuentry "Reboot into UEFI" {
            echo "Rebooting into UEFI..."
            fwsetup
        }
        ```
    * sudo grub-mkconfig -o /boot/grub/grub.cfg
    * Clean/Smooth boot: https://wiki.archlinux.org/index.php/silent_boot
        * https://www.reddit.com/r/thinkpad/comments/aoh4s3/some_clean_booting_action_with_t470_and_archlinux/
    * TODO: Extra boot options (Shutdown/Restart/UEFI)
    * TODO: LTS kernel option.
* Filesystems
    * `sudo pacmatic --needed exfat-utils`
* Fonts:
    * TODO: Look at presets
        * 70-no-bitmaps.conf
        * 10-sub-pixel-rgb
        * 11-lcdfilter-default.conf
    * Look at reddit "Make your Arch fonts beautiful easily!"
    * dejavu
    * Firacode
    * ttf-ubuntu-font-family?
    * noto-fonts, noto-fonts-extra, noto-fonts-cjk, noto-fonts-emoji
    * ttf-liberation (some Windows fonts)

## Desktop Install

Getting key combos: evtest, xev, showkey

* Create your user: `useradd --create-home --groups adm qasim`
    * `adm` groups gives convenient read-access to system log files

* Create an SSH Key `ssh-keygen -t ecdsa -C User@Device#Realm`
* Install `hunspell-en_GB`
* Install `firefox-i18n-en-gb keepassxc openssh`
* Install `git` and `https://github.com/QasimK/dotfiles/`.

* Sway
    * `man 5 sway`
    * TODO: `dbus-run-session sway` for multi-login
        * TODO: sway start up script with wayland env variables
        * TODO: `exec sway` to auto log-out when sway closes
    * TODO: bemenu rather than dmenu
    * `sway xorg-server-xwayland`
    * `cp /etc/sway/config` to `~/.config/way/config` temporarily
    * Start Sway, Firefox and go through the dotfiles setup.
    * TODO: Clamshell mode: https://old.reddit.com/r/swaywm/comments/eim1k1/conditional_clamshell_mode/

* SwayWM:
    * TODO: Use Kanshi for portable output https://github.com/emersion/kanshi
    * Supports 2x Scaling... but extremely application-specific.
        * e.g. Firefox is just blurry.
    * Instead: Recommend 1x, and scale individual applications
        * Sublime Text 3, KeePass, Firefox, Terminals all support individual scaling
    * i3blocks (cp /etc/i3blocks.conf -> ~/.config/i3blocks/config)
    * libinput config: https://wayland.freedesktop.org/libinput/doc/latest/index.html
    * **sway config**:
    # cwd.bash
    ```
    #!/usr/bin/env bas
hm
    terminal=${1:xterm}
    pid=$(swaymsg -t get_tree | jq '.. | select(.type?) | select(.type=="con") | select(.focused==true).pid')
    pname=$(ps -p ${pid} -o comm= | sed 's/-$//')

    if [[ $pname == $terminal ]]
    then
        ppid=$(pgrep --newest --parent ${pid})
        readlink /proc/${ppid}/cwd || echo $HOME
    else
        echo $HOME
    fi
    ```
    # github.com/gumieri/dotfiles/.../sway/focused-cwd
      set $Alt Mod1
      set $Super Mod4
      font pango: DejaVu sans Mono, 8
      # output e-DP-1 scale 2
      set $term termite -e /usr/bin/fish
      set $term-cwd $term -d "$(~/.local/bin/cwd.bash $term)"
      # start a terminal
      bindsym $mod+Return exec $term-cwd
      bindsym $mod+$Alt+Return exec $term

    * Couple of features
      workspace_auto_back_and_forth yes
      force_display_urgency_hint 1000 ms


    * Menu Launcher
      # set $menu dmenu_run
      set $menu "rofi -combi-modi drun,run -show combi -modi combi,ssh,keys -show-icons -matching fuzzy -sort -sorting-method fzf -drun-show-actions"

    * # Media keys
      bindsym XF86AudioLowerVolume exec pactl -- set-sink-volume @DEFAULT_SINK@ -5%
      bindsym XF86AudioRaiseVolume exec pactl -- set-sink-volume @DEFAULT_SINK@ +5%
      bindsym Shift+XF86AudioLowerVolume exec pactl -- set-sink-volume @DEFAULT_SINK@ -25%
      bindsym Shift+XF86AudioRaiseVolume exec pactl -- set-sink-volume @DEFAULT_SINK@ +25%
      bindsym XF86AudioMute exec pactl -- set-sink-mute @DEFAULT_SINK@ toggle
      bindsym XF86AudioMicMute exec pactl -- set-source-mute @DEFAULT_SOURCE@ toggle

      bindsym XF86AudioPlay exec playerctl play-pause
      bindsym XF86AudioNext exec playerctl next
      bindsym XF86AudioPrev exec playerctl previous

      bindsym $Alt+XF86AudioLowerVolume exec pactl -- set-source-volume @DEFAULT_SOURCE@ -5%
      bindsym $Alt+XF86AudioRaiseVolume exec pactl -- set-source-volume @DEFAULT_SOURCE@ +5%

      # Super-Alt-L because Super is used
      bindsym $Super+$Alt+l exec swaylock
      bindsym $Super+c exec $term -e "python3 -BiqsS -c 'from math import *'" # Calc
      bindsym $Super+Shift+bracketleft move workspace to output left
      bindsym $Super+Shift+bracketright move workspace to output right
      bindsym $Super+Tab workspace next
      bindsym $Super+Shift+Tab workspace prev
      bindsym $Super+grave workspace back_and_forth


    * Install acpilight and usermod -a -G video <user>
        bindsym XF86MonBrightnessUp exec xbacklight +5
        bindsym XF86MonBrightnessDown exec xbacklight -5
        # These don't work
        bindsym Shift+XF86MonBrightnessUp exec xbacklight +20
        bindsym Shift+XF86MonBrightnessDown exec xbacklight -20
    * Screenshots
        # grimshot contrib - https://raw.githubusercontent.com/swaywm/sway/master/contrib/grimshot
        # https://gitlab.com/gamma-neodots/neodots.guibin/blob/master/grim-wrapper.sh
        # Screenshot (Selection, Window, Display, Everything)
        # Note grim supports scaled screenshots
        # bindsym Print exec grim -g "$(slurp -d)" - | wl-copy
        bindsym Print exec IMG=~/Pictures/Screenshots/$(date +%Y-%m-%d_%H-%m-%S).png && grim -g "$(slurp -d)" $IMG && wl-copy < $IMG
        bindsym Ctrl+Print exec "/home/test/.local/bin/grim-wrapper.bash -w"
        bindsym $Alt+Print exec grim -o "$(swaymsg -t get_outputs | jq -r '.[] | select(.focused) | .name')" - | wl-copy
        bindsym Shift+Print exec grim - | wl-copy
        # bindsym print exec slurp | grim -g - $(xdg-user-dir PICTURES)/$(date +'screenshot_%Y-%m-%d-%H%M%S.png')
    * Screen Recording
        * wf-recorder
        * Workaround hack for screen-sharing:

        Install https://aur.archlinux.org/v4l2loopback-dkms.git
        sudo modprobe v4l2loopback
        sudo v4l2loopback-ctl set-caps any /dev/video2
        wf-recorder --muxer=v4l2 --codec=rawvideo --pixel-format=yuv422p --file=/dev/video2
        Then select this device as a webcam.
    * Lock Screen
        # Notification service
        exec mako
        bindsym $Super+n exec makoctl dismiss

        # Lock Screen
        set $swaylock_command swaylock --daemonize --ignore-empty-password --show-failed-attempts
        exec swayidle -w \
            timeout 300 "$swaylock_command" \
            timeout 315 'swaymsg "output * dpms off"' \
            resume 'swaymsg "output * dpms on"' \
            before-sleep "$swaylock_command"
        # Wait for next Sway release
        # Note: Waybar has a toggle (e.g. for presentations)
        #inhibit_idle fullscreen
    * TODO: Warbar config and style, and helper scripts?

    * Screen recording
        # wf-recorder
    * NOTE:
      (pactl list short sinks/sources)
    * Note alacritty should be released with scrolling soon, but termite is pretty great.
    * urxvt -fn "xft:Deja Vu Sans Mono:pixelsize=24"
    * ~/.config/termite/config (copy default file first.)
        [options]
        font =  Deja Vu Sans Mono 16
        scrollback_lines = 100000
    * `sudo pacmatic -S --needed waybar otf-font-awesome`
    * Further tips: https://samsaffron.com/archive/2019/04/09/my-i3-window-manager-setup

* firefox:
    * Smooth touchpad scrolling (x-org): env MOZ_USE_XINPUT2=1 firefox
* fish:
    * function keepassxc
        env QT_SCALE_FACTOR=2 keepassxc
      end
      funcsave keepassxc
      # Note dmenu does not support shell aliases!
* Redshift should work with Sway/Wayland.
* TODO: Trackpoint - change sensitivity.
* Be sure to conform to XDG Base Directory Spec (2003), annoying.
* Intel GPU Usage: https://medium.com/@niklaszantner/check-your-intel-gpu-usage-via-commandline-11196a7ee827
    * `intel-gpu-tools`
    * `sudo intel_gpu_top`
* Suspend & Resume processes (Unix)
    * SIGSTOP & SIGCONT
* Time Sync
    * Use Chrony (OPTIONAL)
    /etc/chrony.conf
    server 0.pool.ntp.org iburst
    server 1.pool.ntp.org iburst
    server 2.pool.ntp.org iburst
    server 3.pool.ntp.org iburst
    Note that: uk.pool.ntp.org is not used because ntp.org will try to redirect you to the closest servers,
    and you may not be in the UK all the time. Not sure if the trade-off is worth it.
    They also suggest that the country-specific pools may not have enough servers, but UK does.
* Avahi-browser Windows NetBIOS names: insert wins before mdns_minimal.
    * => hostname.local
* Utils:
    * PAGER=/usr/bin/most for better man pages
    * deepin-screenshot - NOT wayland-compatible. Flameshot does not work either.
    * albert - Does not work with wayland properly
    * fd (find); rg (grep); fzf (fzf fish bindings)
    * Dbeaver - SQL database browser
    * convert - file type converter
    * Pdftk - terminal pdf slicer and dice
    * lsix - terminal image viewer (TODO)
    * ranger - terminal file browser
    * nnn - super simple, efficient file browser (note: fish shell integration)
        * mediainfo
        * atool
    * atool - terminal archive files
    * trash-cli - delete files with `trash`

### TODO

* SSH temporary access:
    * https://linux-audit.com/granting-temporary-access-to-servers-using-signed-ssh-keys/
    * authorized_keys owned by root
    * Storage quota
    * Allow only file copy with rssh shell: https://serverfault.com/a/83857
        * Prefer Scponly?
    * Use chroot to stop them from leaving home folder
    * https://nurdletech.com/linux-notes/ssh/hidden-service.html


NOTE NOTE NOTE
PavuControl set "Analog Stereo Duplex."
How to do this with pactl

Compose Keys - 1- Shift + AltGr, 2- then key combo

1- /usr/share/X11/xkb/rules/base.lst
2- /usr/share/X11/locale/en_US.UTF-8/Compose

### Apps

Zoom Screen Sharing:
https://old.reddit.com/r/swaywm/comments/cfrnz1/has_anyone_managed_to_get_screen_sharing_on_zoom/?st=jz1p1ovy&sh=9dd3acf2

### Custom Repository

https://disconnected.systems/blog/archlinux-repo-in-aws-bucket/

    mkdir -p ./chroots
    mkarchroot -C /etc/pacman.conf ./chroots/root base-devel
    makechrootpkg -cur ./chroots


## Configure Keyboard + Touchpad + Trackpoint

https://faq.i3wm.org/question/3747/enabling-multimedia-keys.1.html
(xorg-xbacklight will not work on Wayland. Try acpilight https://wiki.archlinux.org/index.php/Backlight#Backlight_utilities)

The backlight (Fn + space bar) works fine. Bluetooth and WiFi (F10, F8) are fine.
Capslock+light works fine.

* Manual: F1,2,3 (Audio volume) - light works
* Manual: F4 (Microphone) - light works
* TODO: F7 (Screen Displays). [XF86Display]
* TODO: F9 (Settings). [XF86Tools]
* TODO: F11 (Keyboard settings??)
* TODO: F12 (Whatever function)
* TODO: PrtSc -> make it Super?

Note: Fn+B = Break key
    Fn+K = Scroll Lock
    Fn + P = Pause
    Fn + S = SysRq
    Fn + 4 = Sleep
    Fn + PrintSc = Snipping Tool Program (...) [KEY_SYSRQ]
    Fn + Left = Home key
    Fn + Right = End key

Bottom-right tiny corner area is right-click, the rest is left-click. Two-tap = right-click.

sudo sh -c "echo 1060 > /sys/class/backlight/intel_backlight/brightness"
/sys/class/drm/card0-HDMI-A-1/enabled
card0-e-DP-1 is the main display

Tip: Scrolling with the trackpoint is possible using the middle trackpad button.

## Applications

* Document editing
    * `libreoffice-fresh-en-gb`

* PDF Viewer
    * `aspell-en evince`
    * `aspell-en` ensures English is installed for the `aspell` dependency
    * `envince` is gnome, but does not have any major dependencies
    * ALTERNATIVE: `zathura` with its optional dependencies
* Mail
    * `thunderbird-i18n-en-gb`
    * `javascript.enabled=false` in advanced config editor
    * `mailnews.default_view_flags=1`
    * `mailnews.default_sort_order=2`
* File sync - Resilio
    * Follow the user-installation instructions after installing the aur package, set:
        * `device_name`
        * `storage_path=~/.local/share/rslsync`
        * `pid_path=~/.cache/rslsync.pid`
        * `webui.listen` to localhost
        * `dir_whitelist=~` to make the UI simpler
        * (~ = should be full path)
    * I would love to use syncthing, but it does not supported remote encrypted folders, and iOS support is limited.
* Virtualbox
    * (btrfs: chattr +C "~/Virtualbox VMs")
    * lsattr
    * Do it before creating any files in there.

* ICC
    TODO: colord, colormgr
    * https://wiki.archlinux.org/index.php/ICC_profiles#Loading_ICC_profiles
    (X can specify system-wide ICC for applications that support it. Not currently possible with Wayland.)
    There is a *huge* difference using an ICC profile.
    I tested against BT.709 and BT.2020 Youtube Videos and iPhone 6S Plus (which does sRGB very accurately.)
    And also against my supposedly calibrated Dell 2713HM display.
    This is also a good website: https://chromachecker.com/info/en/page/webbrowser

    TODO: Arch Wiki Page Link.
    With MPV, use instructions below.
    With Gimp: Add it in preferences.
    With Firefox: See wiki page.
        gfx.color_management.mode;1
        gfx.color_management.display_profile;/etc/icc_profile.icm

    * x1c6: https://www.notebookcheck.net/Lenovo-ThinkPad-X1-Carbon-2018-WQHD-HDR-i7-Laptop-review.284682.0.html
        * https://www.notebookcheck.net/uploads/tx_nbc2/B140QAN02_0.icm

* Torrent - Transmission
    * vpnboxe .desktop file, noshow normal one.

* mpv + youtube-dl

    alias yy="mpv --really-quiet --volume=50 --autofit=30% --geometry=-10-15 --ytdl --ytdl-format='mp4[height<=?720]' -ytdl-raw-options=playlist-start=1"

    alias dl-a='youtube-dl -x -f bestaudio  --add-metadata --embed-thumbnail --download-archive --prefer-free-formats -i --output "%(title)s.%(ext)s"'

    # From reddit
    ```
        ;# SINGLE AUDIO
        alias yta='youtube-dl -4icvwxo "%(title)s.%(ext)s" --audio-format mp3 --audio-quality 0 --netrc "$(xclip -selection clipboard -o)"'

        ;# MULTIPLE AUDIOS (playlist)

        alias ytam='youtube-dl -4icvwxo "%(playlist_index)s.%(title)s.%(ext)s" --playlist-reverse --audio-format mp3 --audio-quality 0 --netrc "$(xclip -selection clipboard -o)"'

        ;# SINGLE AUDIO AND KEEP VIDEO

        alias ytak='youtube-dl -4ickvwxo "%(title)s.%(ext)s" --audio-format mp3 --audio-quality 0 --netrc "$(xclip -selection clipboard -o)"'

        ;# SINGLE VIDEO

        alias ytv='youtube-dl -4icvwo "%(title)s.%(ext)s" --netrc "$(xclip -selection clipboard -o)"'

        ;# MULTIPLE VIDEOS (channel or playlist)

        alias ytvm='youtube-dl -4icvwo "%(playlist_index)s.%(title)s.%(ext)s" --playlist-reverse --netrc "$(xclip -selection clipboard -o)"'

    If you only want audio (with meta tags and thumbnail embedded):

        youtube-dl.exe -i --extract-audio --audio-quality 256K --audio-format mp3 --embed-thumbnail --add-metadata <URL_HERE>

    For full video (best quality, with meta tags and thumbnail embedded):

        youtube-dl.exe -i --all-subs --embed-subs --embed-thumbnail --add-metadata --merge-output-format mp4 --format bestvideo[ext=mp4]+bestaudio[ext=m4a] <URL_HERE>

    Another:
        youtube-dl.exe -i --all-subs --embed-subs --embed-thumbnail --add-metadata --merge-output-format mp4 --format bestvideo[ext=mp4]+bestaudio[ext=m4a] <URL_HERE>
        youtube-dl.exe -i --all-subs --embed-subs --embed-thumbnail --add-metadata --merge-output-format mkv --format bestvideo+bestaudio [url]


    ```

    i3wm: for_window [class="(?i)mpv"] floating enable

    YT: pop-up mode: https://www.youtube.com/watch_popup?v=CDsNZJTWw0w

    # We want to force hardware video acceleration on Sway/Wayland
    ```
    hwdec=vaapi
    gpu-context=wayland
    ```

    Setup ICC Profile
    ```
    icc-profile=/etc/icc_profile.icm
    # Set below to correct value if MPV complains about contrast in ICC profile
    # icc-contrast=1500
    ```

    Possibly use --target-trc --target-prim if no ICC profile?

    YTDL Options example:
    ```
    ytdl-format='(bestvideo[height<=?1440][fps<=?60]/bestvideo)+bestaudio[acodec=opus]/bestaudio[acodec=vorbis]/bestaudi[acodec=aac]/bestaudio)'
    ytdl-raw-options=playlist-start=1
    ```

    Higher quality:
    ```
    profile=gpu-hq
    ```
    Consider looking into (scale=ewa_lanczossharp, dscale=ewa_lanczossharp, cscale=ewa_lanczossharp)
    And:
    interpolation=yes
    blend-subtitles=yes
    video-sync=display-resample
    tscale=oversample

    (Note we can copy over all the config files, but this is optional)
    $ cp -r /usr/share/doc/mpv ~/.config
    # rm useless files
    $ vim ~/.config/mpv/mpv.conf

    * https://github.com/TheAMM/mpv_thumbnail_script

* imv - image viewer

* xdg-mime default org.gnome.Evince.desktop application/pdf
* xdg-mime query default application/pdf
> org.gnome.Evince.desktop

* More: https://terminalsare.sexy/

## Backup

* Full-disk backup (dd requires encryption (boot partition). Is backup consistent?)
* BTRFS Backups
    * Snapshots (can even mount and login into snapshot)
    * "Send" to external BTRFS-formatted drive.
* Partial, incremental disk backup (borg-backup).
    * vs DejaDup (GUI), duplicity (CLI)?
* TBD: Cloud backup. Spideroak? rclone?

This gives:

* Full disk image, but does it work? Stored where?
* Local BtrFS backups to restore single files, or revert an update.
* External BtrFS backups in case device fails.
* External Borg simple backup in case the others fail to restore.
* Cloud backup...?

## Future

* The Dolby Vision (88% Adobe RGB) display. Video players. Videos that make use of it. What apps support it?
* HDR???
* IR eye-tracker? Does not come with IR.
* Fingerprint reader? What can we use it for? How do we integrate it? Not currently supported.

## Other hardware

* Got: RJ-45 to proprietary port adapter works fine.
* HDMI <-> HDMI. (Works fine.)
* USB(?)/Thunderbolt Type-C (!) <-> DisplayPort - works fine.
* Need update: OS/Arch Linux Backup USB - essential.
* 2.5" backup hard drive?
* Waiting: Ultimate Hacking Keyboard.
