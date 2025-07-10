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

Creates a file called ```/etc/X11/xorg.conf.d/20-modesetting.conf``` with this content

```
Section "Device"
    Identifier    "My GPU Name"
    Driver        "modesetting"
    Option        "ShadowFB" "false" 
    Option        "Atomic" "true" #only effective on Xlibre, or Xorg-git with a special patch
    Option        "TearFree" "true"
EndSection
```

## 2. DMABUF

Creates a file called ```/etc/X11/xorg.conf.d/15-dmabuf.conf``` with this content

```
Section "ServerFlags"
	Option "Debug" "dmabuf_capable"
EndSection
```

# Konfiguring KDE/KWIN

Now that we have activated TearFree, the X11 server will take care of the tearing, so we can deactivate Kwin's functions in this regard.

Let's create a file ```/home/$USER/.config/plasma-workspace/env/kwin_env.sh```

with this content:

```
export KWIN_PERSISTENT_VBO=1 #???
export KWIN_USE_BUFFER_AGE=1 #default
export KWIN_EXPLICIT_SYNC=0 
export KWIN_X11_NO_SYNC_TO_VBLANK=1
export KWIN_USE_INTEL_SWAP_EVENT=1 #not default, should be relevant only on glx
```

Let's make the file executable with

```chmod +x /home/$USER/.config/plasma-workspace/env/kwin_env.sh```

Now let's disable the synchronisation of Qucik Scene Graph (a function of the QT libraries) but only for Kwin, not for the rest of the system, since for plasmashell and other parts of the system it is useful.


```sudo mkdir /etc/systemd/user/plasma-kwin_x11.service.d/```

Create a file ```/etc/systemd/user/plasma-kwin_x11.service.d/10-kwin_smoother.conf```

With this content

```
[Service]
Environment="QSG_NO_VSINK=1"
```

# Using EGL insted of glx

QT and Kwin work better using the EGL interface instead of the traditional GLX.

Create a file ```/etc/profile.d/egl.sh```

with this content:

```
export QT_XCB_GL_INTEGRATION=xcb_egl
export KWIN_OPENGL_INTERFACE=egl
export GST_GL_PLATFORM=egl #this is not related to kwin, but it's useful
```


However, you may encounter some bugs with Libreoffice's OpenGL alimations. In this case you can modify the Impress launcher to use glx instead of EGL.

In the Envoroment variable: ```QT_XCB_GL_INTEGRATION=xcb_glx```



