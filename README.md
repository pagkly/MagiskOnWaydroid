# Gapps + Magisk on Waydroid 

**If you want to skip fork and straight to downloading the modified image, download [latest build with pico OpenGapps and Magisk v24.1](https://github.com/pagkly/MagiskOnWaydroid/suites/5780425514/artifacts/192678597) and follow [installation](#installation-guide) to init modified images.**

Forked [MagiskOnWSA](https://github.com/LSPosed/MagiskOnWSA) and modified to install Magisk and OpenGapps (pico) replacing FOSS apps in [BlissOS/LineageOS 18.1 dev image (Android 11)](https://sourceforge.net/projects/blissos-dev/files/waydroid/lineage/lineage-18.1/)

Most OpenGApps variants works except for aroma (aroma does not support x86_64, please use super instead)

Tested in ArchLinux, Fedora.

More in [Waydroid Telegram](https://t.me/WayDroid)

## Bugs
1. **Zygisk not yet working (No ETA)**

    Currently, this script is using MagiskonWSA method in patching initrc so that it would load magisk su binaries close to the end of initrc, before ui started. However zygote is usually loaded at the start of initrc, meaning it would be too late to replace zygote with zygisk zygote by the time su from magisk is loaded. 
    In [LSPosed's MagiskonWSA](https://github.com/LSPosed/MagiskonWSA) implementation, there is a [patched kernel with su binaries](https://github.com/LSPosed/WSA-Kernel-SU). Waydroid, on the other hand, uses lxc containers utilizing linux host kernel without su binaries needed to patch zygote.
    Though, there might be a way to load su binaries as kernel module when lxc session is starting. Anybody who is well-versed in lxc can contact me/create issue to explain to me how to make it works.
    
    Modules requiring zygote/zygisk like [Riru](https://github.com/RikkaApps/Riru), [LSPosed](https://github.com/LSPosed/LSPosed) (pre-zygisk) or Shamiko (module to hide magisk root utilizing zygisk) unfortunately wont work for now unless someone figure out a workaround.
    
    In the meantime, here is some examples of modules that work: 
    - [Busybox NDK](https://github.com/Magisk-Modules-Repo/busybox-ndk)
    - [Magisk Hide Prop](https://github.com/Magisk-Modules-Repo/MagiskHidePropsConf)
    - [Detach](https://github.com/Magisk-Modules-Repo/Detach) (detach app from play store)

   **Not all Magisk (> 24102 builds) guaranteed to work. **
    
    This problem is related to the su patched kernel implementation.
    The latest that works is stable v24.1 up to canary 24102.
    This is due to a recent change in how Magisk load the binaries during booting somewhere in commit between build v24102 to v24103 and present in v24.2, 2420x builds.
    I might add several links of magisk version to choose from in workflow so that new version from 2430x onwards can be tested. However, default would still be v24.1 and thus, do not update magisk apk.

2. **Restart waydroid container twice after additional setup in Magisk.**
   
   First restart usually have a bug where the ethernet connection would fail to connect to the internet. Second restart should fix them.
   
   See [Installation Guide](#installation-guide)
    


## Installation Guide
1. Install waydroid on your Linux Distro.
    
   - e.g. In Arch Linux with AUR + yay 

      ```shell
      # install one of the kernel that supports ashmem,binder
      # sudo pacman -S linux-mainline-anbox #(5.17-rc)
      # sudo pacman -S linux-zen #(currently 5.16)
      sudo pacman -Syuu
      yay -S waydroid-image-dev
      ```

1. Go to the **Action** tab in your forked repo
    ![Action Tab](https://docs.github.com/assets/images/help/repository/actions-tab.png)
1. In the left sidebar, click the **Build WSA** workflow.
    ![Workflow](https://docs.github.com/assets/images/actions-select-workflow.png)
1. Above the list of workflow runs, select **Run workflow**
    ![Run Workflow](https://docs.github.com/assets/images/actions-workflow-dispatch.png)
1. Input the download link of Magisk and select the [OpenGApps variant](https://github.com/opengapps/opengapps/wiki#variants) (none is no OpenGApps) you like, select the root solution (none means no root) and click **Run workflow**
    ![Run Workflow](https://docs.github.com/assets/images/actions-manually-run-workflow.png)
1. Wait for the action to complete and download the artifact
    ![Download](https://docs.github.com/assets/images/help/repository/artifact-drop-down-updated.png)
1. Unzip the artifact
    - The size shown in the webpage is uncompressed size and the zip you download will be compressed. So the size of the zip will be much less than the size shown in the webpage.
1. Copy system.img and vendor.img to /usr/share/waydroid-extras/images and init the new img.
    ```shell
    # Create dir if not exists, copy files
    sudo mkdir -p /user/share/waydroid-extras/images
    sudo cp system.img /user/share/waydroid-extras/images/
    sudo cp vendor.img /user/share/waydroid-extras/images/
    # Initialize new images
    sudo waydroid init -f
    # Restart waydroid lxc container
    sudo systemctl start waydroid-container.service
    waydroid session start &
    # Start Waydroid UI
    waydroid show-full-ui
    ```
1. After additional setup in magisk, reboot and restart container twice (refer to [Bugs](#bugs) as to why)
    ```shell
    waydroid stop session
    
    # First container restart
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid stop session
    
    # Second container restart
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid show-full-ui
    ```
    
    
## Additional FAQ

- Why the size of the zip does not match the one shown?

   The zip you downloaded is compressed and Github is showing the uncompressed size.
- How can I update System.img to new version?

    Rerun the Github action, download the new artifact, replace the content of your previous installation.
- How can I update Magisk to new version?

    Do the same as updating System.img
- How to pass safetynet?

    Like all the other emulators, no way.
- Magisk online module list is empty?

    Install manually by 
   
    ```shell
    adb push module.zip /data/local/tmp
    adb shell su -c magisk --install-module /data/local/tmp/module.zip
    ```
- Can I use Magisk 23.0 stable or lower version?

    No. Magisk has bugs preventing itself running on WSA. Magisk 24+ has fixed them. So you must use Magisk 24 or higher version.

- Github Action script is updated, how can I synchronize it?

    1. In your fork repository, click `fetch upstream`
        ![fetch](https://docs.github.com/assets/cb-33284/images/help/repository/fetch-upstream-drop-down.png)
    1. Then and click `fetch and merge`
        ![merge](https://docs.github.com/assets/cb-128489/images/help/repository/fetch-and-merge-button.png)

## Credits
- [Waydroid](https://github.com/waydroid/waydroid)
- [Magisk](https://github.com/topjohnwu/Magisk)
- [The Open GApps Project](https://opengapps.org)
- [WSA-Kernel-SU](https://github.com/LSPosed/WSA-Kernel-SU) and [kernel-assisted-superuser](https://git.zx2c4.com/kernel-assisted-superuser/)
- [WSAGAScript](https://github.com/ADeltaX/WSAGAScript)
- [MagiskonWSA](https://github.com/LSPosed/MagiskonWSA)
- [MagiskOnEmulator](https://github.com/shakalaca/MagiskOnEmulator)
- [MagiskOnEmu](https://github.com/HuskyDG/MagiskOnEmu)
- [Project Kokoro](https://github.com/supremegamers/kokoro)
