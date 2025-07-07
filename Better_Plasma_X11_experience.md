This guide is to improve the KDE Plasma experience on X11. It particularly applies to those using the modesetting driver (mainly Intel graphics cards)

# Install XLibre or xorg-git

This is a crucial step. XLibre and xorg-git contain the "TearFree" feature for modesetting, which is not available in the standard releases. If you use a driver other than modesetting, check that the TearFree option exists for your driver.

Installation depends on the distribution you use. On Arch Linux and derivatives:

**Xorg-git:**

```
sudo yay -S xorg-server-git xorg-server-devel-git xorg-server-common-git xorgproto-git
sudo yay -S xf86-input-libinput-git 
```

**Xlibre:**

see the pinned comment [here](https://aur.archlinux.org/packages/xlibre-server)

# Configuring Xorg

## 1. Modesetting options

Crea un file chiamato ```/etc/X11/xorg.conf.d/20-modesetting.conf``` con questo contenuto

```
Section "Device"
    Identifier  "Intel Graphics"
    Driver      "modesetting"
    Option 	    "ShadowFB" "false"
    # Option      "Atomic" "true" 
    Option      "TearFree" "true"
EndSection
```
