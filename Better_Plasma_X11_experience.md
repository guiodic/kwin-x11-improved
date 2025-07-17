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

see [here](https://github.com/X11Libre/pkgbuilds-arch-based))

# Configuring Xorg

## 1. Modesetting options

TearFree should be on by default, but you can configure things bettere

Creates a file called ```/etc/X11/xorg.conf.d/20-modesetting.conf``` with this content

```
Section "Device"
    Identifier    "My GPU Name"
    Driver        "modesetting"
    Option        "ShadowFB" "false" # you don't need on recent hardware
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
export KWIN_PERSISTENT_VBO=1 #default?
export KWIN_USE_BUFFER_AGE=1 #default
export KWIN_EXPLICIT_SYNC=0 #Xorg takes care of it
export KWIN_X11_NO_SYNC_TO_VBLANK=1 #Xorg takes care of it
export KWIN_USE_INTEL_SWAP_EVENT=1 #not default, should be relevant only on glx, but we are going to use EGL
```

Let's make the file executable with

```chmod +x /home/$USER/.config/plasma-workspace/env/kwin_env.sh```

Now let's disable the synchronisation of Quick Scene Graph (a function of the QT libraries) but only for Kwin, not for the rest of the system, since for plasmashell and other parts of the system it is useful.


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

make it executable 

```sudo chmod +x /etc/profile.d/egl.sh```

However, you may encounter some bugs with Libreoffice's OpenGL alimations. In this case you can modify the Impress launcher in menu, to use glx instead of EGL.

In the Envoroment variable: ```QT_XCB_GL_INTEGRATION=xcb_glx```

# Suggested installations

```sudo yay -S material-kwin-decoration-git # for Locally integrated menu```

kwin-x11 with backports: https://github.com/guiodic/kwin-x11-improved

# Other stuff

## My /etc/modprobe.d/i915.conf

```
options i915 enable_guc=3 enable_fbc=1 nuclear_pageflip=1 enable_dmc_wl=1
```

## My $HOME/.drirc

```
<driconf>
  <device>
    <application name="Default">
      <option name="mesa_glthread" value="true" />
      <option name="force_glsl_extensions_warn" value="false" />
      <option name="mesa_no_error" value="true" />
      <option name="mesa_glthread_app_profile" value="1" />
      <option name="mesa_glthread_driver" value="true" />
    </application>
    <application name="glxgears" executable="glxgears">
     <option name="mesa_glthread" value="true" />
      <option name="force_glsl_extensions_warn" value="false" />
      <option name="mesa_no_error" value="true" />
      <option name="vblank_mode" value="0"      />
      <option name="mesa_glthread_app_profile" value="1" />
      <option name="mesa_glthread_driver" value="true" />
    </application>  
  </device>
</driconf>
```

## Enable vulkan video decoding/encoding on Intel

/etc/profile.d/VULKAN_VIDEO_ON.sh (must be executable)

```
export ANV_DEBUG=video-decode,video-encode
```

## MESA and OPENCL tricks

/etc/profile.d/mesa.sh (must be executable)

```
export INTEL_COMPUTE_CLASS=1
export VAAPI_MPEG4_ENABLED=1
export mesa_glthread=true
#export INTEL_SIMD_DEBUG=fs2x8 # (uncomment for Intel Xe2 or newer)
```

## my chromium flags

$HOME/.config/chromium-flags.conf

```
#--use-gl=angle 
#--use-angle=vulkan
#--use-vulkan
--use-cmd-decoder=passthrough
--enable-features=VaapiIgnoreDriverChecks,VaapiLowPowerEncoderGen9x,Vp9kSVCHWDecoding,VaapiVp9kSVCHWEncoding,DesktopScreenshots,SharingDesktopScreenshotsEdit,WebUIDarkMode,PlatformHEVCDecoderSupport,PlatformHEVCEncoderSupport,PartialRaster,CanvasOopRasterization,AcceleratedVideoDecoder,AcceleratedVideoEncoder,AcceleratedVideoDecodeLinuxGL,AcceleratedVideoDecodeLinuxZeroCopyGL,UseMultiPlaneFormatForHardwareVideo,Vulkan,DefaultANGLEVulkan,VulkanFromANGLE
--ignore-gpu-blocklist 
--disable-gpu-driver-bug-workaround
#
#--enable-skia-graphite
#--skia-graphite-backend="Dawn"
#
--disk-cache-dir=/run/user/1000/chromium-cache
--disk-cache-size=419430400
--enable-chrome-browser-cloud-management
--show-component-extension-options
--enable-gpu-rasterization
--media-router=0
--enable-remote-extensions
--video-capture-use-gpu-memory-buffer 
--enable-video-player-chromecast-support
--enable-pixel-canvas-recording
--enable-native-gpu-memory-buffers
--enable-zero-copy
--enable-partial-raster
--enable-parallel-downloading
--enable-quic
--enable-lazy-image-loading
--enable-lcd-text
--enable-oop-rasterization
--canvas-oop-rasterization
--enable-gpu-compositing
--enable-accelerated-mjpeg-decode
--enable-font-antialiasing
--enable-accelerated-2d-canvas
--enable-gpu-memory-buffer-video-frames
--enable-gpu-memory-buffer-compositor-resources
--enable-hardware-overlays

#FOR DEBUG
#--enable-features=VaapiVideoDecodeLinuxGL
#--vmodule=*/ozone/*=1,*/vaapi/*=4,*/media/*=4,*/shared_image/*=1
#--enable-logging=stderr --v=0 

```










