# Razer Blade 15: Arch Linux changelog

I bought RB15 notebook to replace my outdated MacBook Air 2012 with an idea in my mind
to use it for deep learning purposes. Originally RB15 came with preinstalled Windows 10,
but since I started to use MacOS I couldn't bear Windows no more, so I decided to install
Arch Linux with Gnome desktop environment.

Here're some changes I had to do to Arch Linux to run things smoothly.

## 1. Set Intel integrated graphics for display and NVIDIA GPU for deep learning

### Problem

By default NVIDIA driver used GeForce GTX 1070 Max-Q as primary GPU. The problem was that:
- it reduced available GPU RAM
- it bogged down the whole system during a neural network training
- it increased power consumption (= less battery life)

Related problem: [How to configure iGPU for xserver and nvidia GPU for CUDA work](https://askubuntu.com/questions/1061551/how-to-configure-igpu-for-xserver-and-nvidia-gpu-for-cuda-work)

### Solution

I ran `nvidia-smi` after startup. I expected to see `No running processes found`, but I saw a list of `Xorg`
processes that used my NVIDIA GPU. That means Gnome Display Manager used Xorg session with NVIDIA GPU as
primary GPU.

I examined `/var/log/Xorg.0.log`:


```
(II) xfree86: Adding drm device (/dev/dri/card1)
(II) systemd-logind: got fd for /dev/dri/card1 226:1 fd 11 paused 0
(II) xfree86: Adding drm device (/dev/dri/card0)
(II) systemd-logind: got fd for /dev/dri/card0 226:0 fd 12 paused 0
(**) OutputClass "nvidia" ModulePath extended to "/usr/lib/nvidia/xorg,/usr/lib/xorg/modules,/usr/lib/xorg/modules"
(**) OutputClass "nvidia" setting /dev/dri/card1 as PrimaryGPU
```

`(**)` means that the setting had been read from config file! I found out that the config file was
`/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf`. I changed the config file to set
Intel integrated graphics adapter as primary GPU:


```python
Section "OutputClass"
    Identifier "intel"
    MatchDriver "i915"
    Driver "modesetting"
    Option "PrimaryGPU" "yes"                   # <<<<<< add this string
EndSection

Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
#   Option "PrimaryGPU" "yes"                   # <<<<<< comment this string
    ModulePath "/usr/lib/nvidia/xorg"
    ModulePath "/usr/lib/xorg/modules"
EndSection
```
