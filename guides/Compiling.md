Compiling LÖVR
===

Compiling the LÖVR code yourself lets you create a custom LÖVR build so you can add your own
features or run it on other operating systems.  Below is a guide for setting up all the dependencies
and compiling the code on various types of systems.

System Requirements
---

LÖVR is currently known to work with (and optimized for)

- CPU: 64 bit, little endian, x86 or ARM
- OS: Windows (10+), macOS (11+), Linux (libc 2.31+), Android (10+)
- GPU: supports Vulkan 1.1

Other systems and hardware may work, but are not frequently tested.

Dependencies
---

LÖVR uses the following libraries.  They are included as submodules in the `deps` directory of the
repository, so make sure you clone with the `--recursive` flag or run
`git submodule update --init --recursive` in an existing repository.

- Lua (5.1+) or LuaJIT (2.1+)
- GLFW (3.3+)
- glslang
- OpenXR
- Vulkan SDK
- msdfgen
- ODE

LÖVR requires a C compiler that supports C11.  GCC 4.9, clang 3.1, and Windows SDK 2104 support C11.

Windows
---

From the lovr folder, run these commands to create a build folder and compile the project using
CMake:

    $ mkdir build
    $ cd build
    $ cmake ..
    $ cmake --build .

The executable will then exist at `/path/to/lovr/build/Debug/lovr.exe`.  A LÖVR project (a folder
containing a `main.lua` script) can then be dropped onto `lovr.exe` to run it, or it can be run
via the command line as `lovr.exe path/to/project`.

macOS
---

Build using CMake, as above:

    $ mkdir build
    $ cd build
    $ cmake ..
    $ cmake --build .

The lovr executable should exist in `lovr/build/bin` now.  It's recommended to set up an alias or
symlink so that this executable can be found in your PATH environment variable.  Once that's done,
you can run a project like this:

    $ lovr /path/to/myGame

:::note
You can set the `LOVR_BUILD_BUNDLE` CMake variable to `ON` to build a .app instead of a plain
executable.
:::

LÖVR requires MoltenVK 1.3.268 or higher.  The easiest way to get MoltenVK is to install the Vulkan
SDK from LunarG.  Be sure to leave the "global install" checkbox enabled while installing so LÖVR is
able to find the Vulkan library.

Prebuilt binaries include MoltenVK in the .app bundle, so installing the Vulkan SDK is optional when
using one of those builds.

Linux
---

Install a C compiler and CMake, then run:

    $ mkdir build
    $ cd build
    $ cmake ..
    $ cmake --build .

LÖVR can use either SteamVR or monado as the VR runtime on Linux.

To use LÖVR with SteamVR on Linux, LÖVR needs to run within the Steam Runtime. To do this, first
[install Steam](https://wiki.archlinux.org/index.php/Steam#Installation).  Next, [install the Steam
udev rules](https://github.com/ValveSoftware/SteamVR-for-Linux#usb-device-requirements).
Then, run LÖVR within the Steam runtime:

    $ ~/.local/share/Steam/ubuntu12_32/steam-runtime/run.sh lovr

If you receive errors related to `libstdc++`, set the `LD_PRELOAD` environment variable when running
the command:

    $ LD_PRELOAD='/usr/$LIB/libstdc++.so.6 /usr/$LIB/libgcc_s.so.1' ~/.steam/steam/ubuntu12_32/steam-runtime/run.sh lovr

Android
---

Here be dragons.

### Setup

You will need to compile on a macOS or Linux system. Compiling LÖVR APKs on Windows does not
currently work (unless all steps are performed in [Windows Subsystem for
Linux](https://learn.microsoft.com/en-us/windows/wsl/install)).

First, make sure the Java JDK is installed (version 17 is known to work).

Then, install the Android SDK (29) and NDK (21.4.7075529).

The Android command line tools can be found on the Android website or installed using a package
manager.  The command line tools contain a tool named `sdkmanager` that can be used to install
various versions of Android, the Android build tools, and the NDK:

    $ cmdline-tools/bin/sdkmanager --sdk_root=/path/to/android/sdk "build-tools;34.0.0" "cmake;3.22.1" "ndk;21.4.7075529" "platform-tools" "platforms;android-29"

The SDK will be installed to the chosen `sdk_root` path.  To reduce the SDK size, the `emulator`
package can be safely uninstalled:

    $ cmdline-tools/bin/sdkmanager --sdk_root=/path/to/android/sdk --uninstall emulator

Android Studio isn't required, but can be used to install the SDK, NDK, and Java as well.

Note where the SDK is installed.  Some paths in the SDK will need to be specified.

Finally, compiling a LÖVR APK requires a copy of the `glslangValidator` tool installed on the
system. Most package managers will offer this as part of a "glslang" or "glslang-tools" package.

### Building

The following CMake variables need to be set, either using the CMake GUI or by using `-D` flags on
the command line:

- Set `CMAKE_TOOLCHAIN_FILE` to the path to `android.toolchain.cmake`.  This is usually at
  `ndk-bundle/build/cmake/android.toolchain.cmake` inside the Android SDK.
- Set `ANDROID_SDK` to the path to the Android SDK.
- Set `ANDROID_ABI` to `arm64-v8a`.
- Set `ANDROID_NATIVE_API_LEVEL` to the Android version to use (e.g. `29`).
- Set `ANDROID_BUILD_TOOLS_VERSION` to one of the versions listed in the `build-tools` folder.
- Set `ANDROID_KEYSTORE` to the path to they keystore file.  See "Creating a Keystore" below.
- Set `ANDROID_KEYSTORE_PASS` to the keystore password.  This can be used in multiple ways,
  described in [Creating a Keystore](#creating-a-keystore) below.
- Optional: Set `ANDROID_MANIFEST` to use a custom Android manifest XML file.
- Optional: Set `ANDROID_PACKAGE` to change the package name from `org.lovr.app`.
- Optional: Set `ANDROID_ASSETS` to include extra assets (e.g. a project folder) in the APK.
- Windows: Make sure you add `-G "Unix Makefiles"` so it doesn't try to use Visual Studio.

The usual CMake incantation with all of the above variables set up should produce `lovr.apk`:

    $ mkdir build
    $ cd build
    $ cmake \
        -D CMAKE_TOOLCHAIN_FILE=/path/to/ndk/build/cmake/android.toolchain.cmake \
        -D ANDROID_SDK=/path/to/android \
        -D ANDROID_ABI=arm64-v8a \
        -D ANDROID_NATIVE_API_LEVEL=29 \
        -D ANDROID_BUILD_TOOLS_VERSION=34.0.0 \
        -D ANDROID_KEYSTORE=/path/to/test.keystore \
        -D ANDROID_KEYSTORE_PASS=pass:hunter2 \
        ..
    $ cmake --build .

### Installing the APK

To install the APK, an Android device needs to be connected.  Run

    $ adb devices

to ensure that a device is connected, then run

    $ adb install lovr.apk

To install the apk.  The `-r` flag can be used to overwrite an existing apk.

### Adding Project Code

To build an APK that runs a LÖVR project instead of the splash screen, add an `ANDROID_ASSETS` CMake
variable with the path to the project folder.  The project will be included in the `assets` folder
of the APK, and LÖVR will run that project when the APK starts.

:::warning
By default the Android packager ignores directories that begin with underscores.
:::

### Using a Custom Android Manifest

Although LÖVR provides a default `AndroidManifest.xml`, you can also use your own by passing its
path as the `ANDROID_MANIFEST` option to CMake.  This can be used to request extra permissions,
change the package ID or app name, etc.

### Creating a Keystore

APKs must be signed to work properly.  First, generate a keystore file using Java's `keytool` tool:

    $ keytool -genkey -keystore <name>.keystore -alias <name> -keyalg RSA -keysize 2048 -validity 10000

The `ANDROID_KEYSTORE` CMake variable should be set to the path to the keystore file, and the
`ANDROID_KEYSTORE_PASS` variable contains the password used when creating the keystore. The password
can take the following forms:

- `pass:<string>` will use `<string>` as the password.
- `env:<var>` will use the value of the `<var>` environment variable.
- `file:<file>` will use the contents of `<file>` as the password.

Troubleshooting
---

- If you get "CMake no `CMAKE_CXX_COMPILER` found" on Windows, then install Visual Studio and create a
  blank C++ project, which will prompt you to install the compilers and tools necessary to compile
  LÖVR.
- If you get the "The code execution cannot proceed because VCRUNTIME140_1.dll was not found" popup
  on Windows, install the Microsoft Visual C++ Redistributable package.
- If you receive an issue about the `<stdalign.h>` header missing on Windows, then the compiler does
  not support C11, and a newer version of the Windows SDK (at least 2104) needs to be installed.
