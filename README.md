# kwin_x11 with ports from kwin-wayland, bug fixes, and maybe other improvements

Patches, PKGBUILD and binary.

# Why still on X11?

With each release of a new version of Plasma I try to make the switch to Wayland, only to quickly revert back to X11. 

Here are the reasons:

1. The first is perhaps personal and concerns the Locally Integrated Menu. I use the [material decoration](https://github.com/guiodic/material-decoration) that implements it and it doesn't work on Wayland. I saw that KDE devs are working on an upstream implementation, but it won't work for GTK apps. They say that a special Wayland protocol is needed, as usual. So in all likelihood it will never be solved, unless KDE rewrites the GTK plugin to use KDE's (private) protocol.

2. The second concerns the lack of inertial scrolling. Xorg allows this at the server level, with Synaptics driver, while in Wayland each application has to do it on its own. Again, this will take many years. It will probably never happen for applications under Wine, which I depend on for work, as Linux has no decent PDF editor. But apart from that, using Okular without inertial scrolling is painful. Recently, KDE developers said they had implemented inertial scrolling for QtQuick apps, but it does not actually work.

3. Libreoffice+QT (or KF5/6) on Wayland has had a bug for years that makes scrolling slow and jerky to an intolerable level.

4. Chromium still has numerous problems under Wayland such as drag&drop not always working.

5. Of course, one cannot forget the problem of restoring windows in the position in which they were closed the last time, especially between different sessions, which will probably take many more years to solve. A very basic feature still lacking in Wayland.

6. After that there are minor annoyances (unreliable thumbnails in Plasma, among them), but which still make the overall experience disappointing.

These, and other minor other deficiencies, severely impact my work.

A few of these problems are solved by launching apps with Xwayland. At this point, you might as well use X11.

Unfortunately, however, the KDE developers are abandoning X11. Improvements are rejected and bugs are not fixed. 

This is why I found myself forced to open this repository that is a patchset for kwin_x11. If you want to contribute, prepare an MR or write a bug report. Thank you in advance!

If you want an improved experience with kwin_x11, I also recommend [this guide](https://gist.github.com/guiodic/2bcc8f2f126d14b1f8a439f644fdc2c9) I wrote.

For more on Wayland's problems [see also this](https://gist.github.com/probonopd/9feb7c20257af5dd915e3a9f2d1f2277=).


