## Setup Zed Camera on Ubuntu 16.04 x64

### Initial Notes

> This has to be done on Ubuntu 16.04 x64 because the CUDA 8 related binaries were made with the gcc5 that is installed with 16.04. I tried it with 16.10, but the gcc is version 6 and everything fails.

### Install Ubuntu alongside Windows 10
- [reference](http://www.tecmint.com/install-ubuntu-16-04-alongside-with-windows-10-or-8-in-dual-boot/) link for dual booting ubuntu with win10
- [download](http://releases.ubuntu.com/16.04/ubuntu-16.04.1-desktop-amd64.iso) ubuntu 16.04
- [download](https://etcher.io/) etcher for creating a usb install drive
- prepare windows for dual boot
  - reboot computer and enter the uefi bios
    - this usually requires pressing <del> or <F2>
  - in the bios
    - disable fastboot
    - disable secure boot
      - usually means changing it from `Windows` to `Other OS`
    - save and reboot
  - Start Menu -> Command Prompt (Admin)
  - `diskmgmt.msc`
  - right click on C:\
  - select `shrink`
  - give at least 40 GB
    - Once the space has been resized you will see a new unallocated space on the hard drive.
    - Leave it as default
  - reboot the computer
- burn the ubuntu iso to the usb drive
  - insert the usb drive
  - start etcher
  - select the ubuntu 16.04 iso
  - select the usb drive
  - start the flash
  - notes:
    - you may have to format the usb drive prior to flashing
      - fat32 with mbr is fine
  - when flash is complete
    - keep usb plugged in
    - reboot computer
- install ubuntu
  - enter the uefi bios as before
    - usually by pressing <del> or <F2>
  - go to the boot tab
    - there will be windows, uefi for usb drive, and just the usb drive, maybe some others like your hard drive
  - select option to boot from uefi associated with the usb
  - a menu shows up that looks like this:
  ```
  Try Ubuntu without installing
  Install Ubuntu
  OEM Install (for manufacturers)
  Check disk for defects
  ```
  - cursor down to highlight `Install Ubuntu` but **Don't Press Enter**
    - we are going to temporarily edit the boot configuration
    - if we don't do this, we will have graphics problems (black screen) since there are no nVidia drivers installed yet
  - press the `e` on your keyboard
  - there is a line that says something like
    - `linux /boot/zImage root=UUID=904bf39-9234 ro quiet splash`
  - replace `quiet splash` with `nomodeset`
    - `nomodeset` tells the kernel to not start video drivers until the system is up and running
  - press `F10` to boot with this configuration
  - now you are ready to perform the installation as usual
  - be sure to leave the following **Unchecked**:
    - [ ] Download updates while installing Ubuntu
    - [ ] Install third-party yadda yadda
  - Now it’s time to select an Installation Type.
    - You can choose to Install Ubuntu alongside Windows Boot Manager, option that will automatically take care of all the partition steps.
    - The option `Erase disk and install Ubuntu` should be **avoided** on dual-boot because is potentially dangerous and will wipe out your disk
    - Otherwise choose `Something else` and lay out the partitions as you see fit
      - select the unallocated space
      - click the plus sign
        - set the mount point to be /
        - set format to `ext4`
        - adjust the size to leave room for swap space (like 4GB or whatever)
        - click ok
      - select the remaining unallocated space
      - click plus sign again
        - set format type to `swap space`
        - click ok
      - note:
        - the section `Device for bootloader installation` doesn't matter since the computer uses UEFI, so this gets ignored. It's a hold-over from old BIOS days
      - click `Install Now`
  - The rest of the install should go pretty smooth
    - set your timezone, username, password, etc
    - grab a coffee
  - The installer will tell you to reboot
    - at some point the process will pause and tell you to remove the media and press enter
  - the system will reboot into Grub
    - you'll be greeted with the boot options for Ubuntu or Windows
  - highlight `Ubuntu` but **Don't Press Enter**
    - we have to do the `nomodeset` thing one last time
  - press the `e` on your keyboard
    - this will look familiar but there may be quite a bit more stuff
  - replace `quiet splash` with `nomodeset`
  - press `F10` to boot with this configuration
- install nVidia Driver
  - [reference](http://www.webupd8.org/2016/06/how-to-install-latest-nvidia-drivers-in.html)
  - pull up the terminal
    - click the ubuntu icon in the top left and type `terminal` then click it
    - right click on the icon in the task bar and select `Lock to Launcher`
    - makes it easier to get to the terminal later
  - `sudo add-apt-repository ppa:graphics-drivers/ppa`
  - `sudo apt-get update`
  - From System Settings (the gear icon on the left)
    - open Software & Updates
    - click on the "Additional Drivers" tab
    - select the driver 367.57
      - cuda 8 depends on this version for now
    - click "Apply changes"
    - reboot

> That’s it! In case you need to switch back to Windows, just reboot the computer and select Windows from the Grub menu.

### CUDA 8.0 and nVidia Drivers

- browse to https://developer.nvidia.com/cuda-downloads
  - select the following
    - Linux -> x86_64 -> Ubuntu -> 16.04 -> deb (network)
  - `cd ~/Downloads`
  - `sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb`
  - `sudo apt-get update`
  - `sudo apt-get install cuda`
  - grab coffee

### OpenCV 3.1 with CUDA 8.0 Support

> there are additional things we can build opencv with, like qt5 and eigen3 support. that will be left for further exploration after we get this stuff built.

- [reference](http://www.pyimagesearch.com/2016/07/11/compiling-opencv-with-cuda-support/) for building opencv with cuda 8 support
- install dependencies
  - `sudo apt-get install build-essential cmake libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev libgtk2.0-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libatlas-base-dev gfortran libhdf5-serial-dev python2.7-dev`
- install and configure python virtual environment
  - `wget https://bootstrap.pypa.io/get-pip.py`
  - `sudo python get-pip.py`
  - `sudo pip install virtualenv virtualenvwrapper`
  - `sudo rm -rf get-pip.py ~/.cache/pip`
  - `nano ~/.bashrc`
    - at the bottom of the file add the following:
    ```bash
    # virtualenv and virtualenvwrapper
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh
    ```
  - create the cv working environment
    - `source ~/.bashrc`
    - `mkvirtualenv cv`
    - `pip install numpy`
- download and build OpenCV
  - `mkdir opencv_compile`
  - `cd opencv_compile`
  - `git clone --depth=1 https://github.com/opencv/opencv.git`
  - `git clone --depth=1 https://github.com/opencv/opencv_contrib.git`
  - `cd opencv && mkdir build && cd build`
  ```bash
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D WITH_CUDA=ON \
    -D ENABLE_FAST_MATH=1 \
    -D CUDA_FAST_MATH=1 \
    -D WITH_CUBLAS=1 \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules \
    -D BUILD_EXAMPLES=ON ..
  ```
  - After running cmake , take a look at the “NVIDIA CUDA” section and verify it looks like this:
    ```bash
    --   NVIDIA CUDA
    --     Use CUFFT:                   YES
    --     Use CUBLAS:                  YES
    --     USE NVCUVID:                 NO
    --     NVIDIA GPU arch:             20 21 30 35
    --     NVIDIA PTX archs:            30
    --     Use fast math:               YES
    ```
  - `make -j8`
  - grab coffee
  - `sudo make install`
  - `sudo ldconfig`

### ZED SDK
- https://www.stereolabs.com/getstarted/





## Temporary Dev Notes Scratch Pad
> this is just a section for dumping stuff as it's added or pulled out in case it is useful later while building this document

- `sudo apt install git vim synaptic indicator-multiload`
- we have to clone opencv from git and apply a patch because of a [bug](https://github.com/opencv/opencv/issues/6677)
  - `git clone https://github.com/opencv/opencv.git`
  - `cd opencv`
  - `git checkout 3.1.0`
  - `git format-patch -1 10896129b39655e19e4e7c529153cb5c2191a1db`
  - `git am < 0001-GraphCut-deprecated-in-CUDA-7.5-and-removed-in-8.0.patch`
  - `git format-patch -1 d766663add331bbec49f6dfbd5dee45966bbc34b`
  - `git am < 0002-Fix-public-hdf-headers.patch`
  - `cd ..`
