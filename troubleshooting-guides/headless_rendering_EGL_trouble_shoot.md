# Headless Rendering EGL Trouble Shoot

One of the problems I encountered when installing and running a 3D rendering simulator on some GPU nodes on the EIDF server was the following error: 

I encountered this error message in a selected number of GPU nodes: 

```
Platform::WindowlessEglApplication::tryCreateContext(): 
unable to find EGL device for CUDA device 0
WindowlessContext: Unable to create windowless context
```

The error happens because the GPU node Nvidia driver does not come with libEGL_nvidia.so. To resolve this issue, we will apply the following patch (a more systematic way is to ask the admin team to update the GL support) to install the missing libraries.

### Nvidia driver temporary patch (Local Nvidia driver library install)

* Go into the root directory `cd /root`
* Find the nvidia driver version: `nvidia-smi` (It was 515.105.01 in my case so will use this for this example).
* Find the matching nvidia driver and download:
    * `wget https://download.nvidia.com/XFree86/Linux-x86_64/515.105.01/NVIDIA-Linux-x86_64-515.105.01.run `
* Extract it:  `sh NVIDIA-Linux-x86_64-515.105.01.run --extract-only`
    * The usual place to find the nvidia libraries: `/usr/local/lib/x86_64-linux-gnu/` or `/usr/lib/x86_64-linux-gnu/ `
    * And it should have contained: `libEGL_nvidia.so.0`, which is not there with the native driver.
* Make sure you can find the file `/usr/share/glvnd/egl_vendor.d/10_nvidia.json`
    * The file looks like this:
        * {
                "file_format_version" : "1.0.0",
                "ICD" : {
                    "library_path" : "libEGL_nvidia.so.0"
                }
            }
    * [If not found] It may also appear in `/usr/local/share/glvnd/egl_vendor.d/`, if so symlink it.
        * `cd /usr/share && mkdir glvnd && cd glvnd && mkdir egl_vendor.d`
        * `ln -s /usr/local/share/glvnd/egl_vendor.d/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json`
* Create the following symbolic links: 
    * Go to the Nvidia folder: `cd /root/NVIDIA-Linux-x86_64-515.105.01`
    * ln -s ./libEGL_nvidia.so.515.105.01 libEGL_nvidia.so.0
        ln -s ./libEGL.so.515.105.01 libEGL.so.1
        ln -s ./libGL.so.1.7.0 libGL.so.1
* Export `LD_LIBRARY_PATH`: 
    * Note that this may change when activate/deactivate a conda environment.
    * `export LD_LIBRARY_PATH=/root/NVIDIA-Linux-x86_64-515.105.01:$LD_LIBRARY_PATH`

### Reference

https://github.com/facebookresearch/habitat-sim/issues/1511#issuecomment-1537221109
