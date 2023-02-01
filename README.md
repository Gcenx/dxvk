# DXVK

A Vulkan-based translation layer for Direct3D 9/10/11 which allows running 3D applications on macOS using Wine.

Release builds can be found [here](https://github.com/doitsujin/dxvk/releases).

## How to use
In order to install a DXVK package obtained from the [release](https://github.com/Gcenx/DXVK-macOS/releases) page into a given wine prefix, copy or symlink the DLLs into the following directories as follows, then open `winecfg` and manually add DLL overrides for `d3d11`, `d3d10core`, `dxgi`, and `d3d9`:
```
WINEPREFIX=/path/to/wineprefix
cp x64/*.dll $WINEPREFIX/drive_c/windows/system32
cp x32/*.dll $WINEPREFIX/drive_c/windows/syswow64
winecfg
```

Verify that your application uses DXVK instead of wined3d by checking for the presence of the log file `d3d9.log` or `d3d11.log` in the application's directory, or by enabling the HUD (see notes below).

In order to remove DXVK from a prefix, remove the DLLs and DLL overrides, and run `wineboot -u` to restore the original DLL files.

## Build instructions

### Requirements:
- [Meson](https://mesonbuild.com/) build system (at least version 0.49)
- [Mingw-w64](https://www.mingw-w64.org) compiler and headers (at least version 8.0)
- [glslang](https://github.com/KhronosGroup/glslang) compiler

### Building DLLs

#### The simple way
Inside the DXVK directory, run:
```
./package-release.sh master /your/target/directory --no-package
```

This will create a folder `dxvk-master` in `/your/target/directory`, which contains both 32-bit and 64-bit versions of DXVK, which can be set up in the same way as the release versions as noted above.

In order to preserve the build directories for development, pass `--dev-build` to the script. This option implies `--no-package`. After making changes to the source code, you can then do the following to rebuild DXVK:
```
# change to build.32 for 32-bit
cd /your/target/directory/build.64
ninja install
```

#### Compiling manually
```
# 64-bit build. For 32-bit builds, replace
# build-win64.txt with build-win32.txt
meson setup --cross-file build-win64.txt --buildtype release --prefix /your/dxvk/directory build.w64
cd build.w64
ninja install
```

The D3D9, D3D10, D3D11 and DXGI DLLs will be located in `/your/dxvk/directory/bin`. Setup has to be done manually in this case.

### Notes on Vulkan drivers
Before reporting an issue, please check the [Wiki](https://github.com/doitsujin/dxvk/wiki/Driver-support) page on the current driver status and make sure you run a recent enough driver version for your hardware.

### Online multi-player games
Manipulation of Direct3D libraries in multi-player games may be considered cheating and can get your account **banned**. This may also apply to single-player games with an embedded or dedicated multiplayer portion. **Use at your own risk.**

### HUD
The `DXVK_HUD` environment variable controls a HUD which can display the framerate and some stat counters. It accepts a comma-separated list of the following options:
- `devinfo`: Displays the name of the GPU and the driver version.
- `fps`: Shows the current frame rate.
- `frametimes`: Shows a frame time graph.
- `submissions`: Shows the number of command buffers submitted per frame.
- `drawcalls`: Shows the number of draw calls and render passes per frame.
- `pipelines`: Shows the total number of graphics and compute pipelines.
- `memory`: Shows the amount of device memory allocated and used.
- `gpuload`: Shows estimated GPU load. May be inaccurate.
- `version`: Shows DXVK version.
- `api`: Shows the D3D feature level used by the application.
- `cs`: Shows worker thread statistics.
- `compiler`: Shows shader compiler activity
- `samplers`: Shows the current number of sampler pairs used *[D3D9 Only]*
- `scale=x`: Scales the HUD by a factor of `x` (e.g. `1.5`)

Additionally, `DXVK_HUD=1` has the same effect as `DXVK_HUD=devinfo,fps`, and `DXVK_HUD=full` enables all available HUD elements.

### Frame rate limit
The `DXVK_FRAME_RATE` environment variable can be used to limit the frame rate. A value of `0` uncaps the frame rate, while any positive value will limit rendering to the given number of frames per second. Alternatively, the configuration file can be used.

### Device filter
Some applications do not provide a method to select a different GPU. In that case, DXVK can be forced to use a given device:
- `DXVK_FILTER_DEVICE_NAME="Device Name"` Selects devices with a matching Vulkan device name, which can be retrieved with tools such as `vulkaninfo`. Matches on substrings, so "VEGA" or "AMD RADV VEGA10" is supported if the full device name is "AMD RADV VEGA10 (LLVM 9.0.0)", for example. If the substring matches more than one device, the first device matched will be used.

**Note:** If the device filter is configured incorrectly, it may filter out all devices and applications will be unable to create a D3D device.

### State cache
DXVK caches pipeline state by default, so that shaders can be recompiled ahead of time on subsequent runs of an application, even if the driver's own shader cache got invalidated in the meantime. This cache is enabled by default, and generally reduces stuttering.

The following environment variables can be used to control the cache:
- `DXVK_STATE_CACHE=0` Disables the state cache.
- `DXVK_STATE_CACHE_PATH=/some/directory` Specifies a directory where to put the cache files. Defaults to the current working directory of the application.

### Debugging
The following environment variables can be used for **debugging** purposes.
- `VK_INSTANCE_LAYERS=VK_LAYER_KHRONOS_validation` Enables Vulkan debug layers. Highly recommended for troubleshooting rendering issues and driver crashes. Requires the Vulkan SDK to be installed on the host system.
- `DXVK_LOG_LEVEL=none|error|warn|info|debug` Controls message logging.
- `DXVK_LOG_PATH=/some/directory` Changes path where log files are stored. Set to `none` to disable log file creation entirely, without disabling logging.
- `DXVK_CONFIG_FILE=/xxx/dxvk.conf` Sets path to the configuration file.
- `DXVK_PERF_EVENTS=1` Enables use of the VK_EXT_debug_utils extension for translating performance event markers.
