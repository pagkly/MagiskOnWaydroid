# Gapps + Magisk on Waydroid BlissOS/LineageOS 18.1 Dev Image (Android 11)

## Bugs
1. **Zygisk not yet working (No ETA)**
    
    This means modules requiring zygote/zygisk like riru, lsposed (pre-zygisk) or shamiko (with zygisk) wont work. Example of modules working: Busybox NDK, Magisk Hide Prop, Detach (detach app from play store)
    
3. Restart waydroid container twice after additional setup in Magisk. First restart usually have a bug where the ethernet connection would fail to connect to the internet. Second restart should fix them.

    ``` 
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid stop session
    sudo systemctl restart waydroid-container.service
    waydroid start session #(wait until successfully booted)
    waydroid show-full-ui
    ```

## Features
- Forked [MagiskOnWSA](https://github.com/LSPosed/MagiskOnWSA) and modified to install Magisk and Pico OpenGapps in Waydroid 11 system.img.
- Support all OpenGApps variants except for aroma (aroma does not support x86_64, please use super instead)


## Text Guide

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
1. Copy system.img and vendor.img to /usr/share/waydroid-extras/images
    ```shell
    # Initialize new images
    sudo waydroid init -f
    # Restart waydroid lxc container
    sudo systemctl start waydroid-container.service
    waydroid session start
    ```

    - Open new terminal and type `waydroid show-full-ui`

## FAQ

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
   
    `adb push module.zip /data/local/tmp`
    
    `adb shell su -c magisk --install-module /data/local/tmp/module.zip`.
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
- [Project Kokoro](https://github.com/supremegamers/kokoro)
- [MagiskOnEmulator](https://github.com/shakalaca/MagiskOnEmulator)
