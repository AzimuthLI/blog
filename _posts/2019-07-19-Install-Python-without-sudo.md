---
layout: post
title: Install Python on Ubuntu without sudo
date: 2019-07-17 13:32:20 +0300
description:  # Add post description (optional)
img:  python_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Python, Linux]
---

System: Ubuntu 18.04 LTS

Dependencies 

``` 
$ sudo apt update
$ sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget 
```

Note: the dependencies requires sudo privilege to install, but usually those are already provided. However, if you are working on a machine/server without those dependencies, maybe you should write to your system admin.

## Install Python from source

The trick of installing python without sudo is to install it from source. Here are the steps:

1. Download Python package and extract the source code.

    ```
    $ mkdir ~/.local && cd ~/.local
    $ mkdir src python && cd src
    $ wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tar.xz 
    $ tar -xf Python-3.7.4.tar.xz
    ```

2. Navigate to the source code directory and run the configure script:

    ```
    $ ./configure --prefix=$HOME/.local/python --enable-optimizations 
    ```

    The `--enable-optimizations` option will optimize the Python binary by running multiple tests which will make the build process slower.

3. Install Python (change number of cores `-j ncpu` accroding to your computer specification):

    ```
    $ make -j 8 && make install
    ```

4. Add system environment variables:

    ```
    $ echo "export PATH=$HOME/.local/python/bin:$PATH" >> ~/.bashrc
    $ source ~/.bashrc
    ```

5. Verify installation:

     ```
     $ which python3
     $ <Your home dir>/.local/python/bin
     ```

## Debugging

If you see `ImportError: No module named '_ctypes'` or any error info related to missing module `'_ctypes'`, it is because the `libffi-dev` package can not be correctly located. 

Run the following command:

```
$ pkg-config --libs-only-L libffi
$ pkg-config --cflags libffi
```

If they do not show the libffi `/libs` directory and `/include` directory, then the system libffi package is not properly found by the python installer, thus we need to install libffi package from source too.

## Install libffi and link it to python 

1. Download `libffi-3.2.1` and extract the source code:

    ```
    $ cd ~/.local/src 
    $ wget https://sourceware.org/ftp/libffi/libffi-3.2.1.tar.gz 
    $ tar -xf libffi-3.2.1.tar.gz
    ```

2. Install the package under user directory (again, change number of cores accrodingly):

    ```
    $ mkdir ~/.local/libffi
    $ cd ~/.local/src/libffi-3.2.1
    $ sed -e '/^includesdir/ s/$(libdir).*$/$(includedir)/'     \
    $ -i include/Makefile.in && 
    
    $ sed -e '/^includedir/ s/=.*$/=@includedir@/'     \
    $ -e 's/^Cflags: -I${includedir}/Cflags:/'     \
    $ -i libffi.pc.in        && 
    
    $ ./configure --prefix=/<your home dir>/.local/libffi --disable-static && 
    
    $ make -j 16 $$ make install
    ```

4. Then re-run the two `pkg-config` command and see the output. If it shows:

    ```
    $ pkg-config --cflags libffi
    Package libffi was not found in the pkg-config search path.
    Perhaps you should add the directory containing `libffi.pc'
    to the PKG_CONFIG_PATH environment variable
    No package 'libffi' found
    ```

    Then do the following to add environment variables:

    ```
    $ echo "PATH=$HOME/.local/libffi/bin:$PATH" >> ~/.bashrc
    $ echo "export PKG_CONFIG_PATH=$HOME/.local/libffi/lib/pkgconfig:$PKG_CONFIG_PATH" >> ~/.bashrc
    $ source ~/.bashrc
    ```

5. Re-run the command above and it should show:

    ```
    $ pkg-config --libs-only-l  libffi
    -lffi
    $ pkg-config --cflags libffi
    -I/<your home dir>/.local/libffi/include
    ```
    However, if you don't see the `--cflags` output (which happened to me), then open the file `~/.local/libffi/lib/pkgconfig/libffi.pc` with any editor and add `-I${includedir}` behind `Cflags`.

Then, you can re-do the python installation process (perhaps remove the failed installation and re-install from the very beginning). This time, change the configuration step (step 2) as:

```
$ LDFLAGS=`pkg-config --libs-only-L libffi` ./configure --prefix=$HOME/.local/python --enable-optimizations 
```

Happy coding!

