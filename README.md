## Setup Zed Camera on Ubuntu 16.04 x64

### Initial Notes

> This has to be done on Ubuntu 16.04 x64 because the CUDA 8 related binaries were made with the gcc5 that is installed with 16.04. I tried it with 16.10, but the gcc is version 6 and everything fails.

### CUDA 8.0 and nVidia Drivers
- [reference](http://www.webupd8.org/2016/06/how-to-install-latest-nvidia-drivers-in.html)
- `sudo add-apt-repository ppa:graphics-drivers/ppa`
- `sudo apt-get update`
- From System Settings or directly from the menu / Dash
  - open Software & Updates
  - click on the "Additional Drivers" tab
  - select the driver 367.57
    - cuda 8 depends on this version for now
  - click "Apply changes"
  - reboot
- browse to https://developer.nvidia.com/cuda-downloads
  - select the following
    - Linux -> x86_64 -> Ubuntu -> 16.04 -> deb (network)
  - `cd ~/Downloads`
  - `sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb`
  - `sudo apt-get update`
  - `sudo apt-get install cuda`

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
  - After running cmake , take a look at the “NVIDIA CUDA” section
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
  - `sudo make install`
  - `sudo ldconfig`

### ZED SDK
- https://www.stereolabs.com/getstarted/





## Other things installed
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
