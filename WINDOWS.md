
# Building on Windows

---

## 🚀 Quick Overview

This guide walks you through the process of building Stremio on Windows. Follow the steps carefully to set up the environment, build dependencies, and compile the project.  

---

## 🛠️ Requirements

Ensure the following are installed on your system:

- **Operating System**: Windows 7 or newer  
- **Utilities**: [7zip](https://www.7-zip.org/) or similar  
- **Tools**:
  - [Git](https://git-scm.com/download/win)  
  - [Microsoft Visual Studio](https://visualstudio.microsoft.com/)  
  - [CMake](https://cmake.org/)  
  - [Qt](https://www.qt.io/)  
  - [OpenSSL](https://slproweb.com/products/Win32OpenSSL.html)  
  - [Node.js](https://nodejs.org/)  
  - [FFmpeg](https://ffmpeg.org/download.html)  
  - [MPV](https://sourceforge.net/projects/mpv-player-windows/)  

---

## 📂 Setup Guide

### 1️⃣ **Install Essential Tools**
- **Git**: [Download](https://git-scm.com/download/win) and install.
- **Visual Studio**: [Download Community 2022](https://visualstudio.microsoft.com/de/downloads/).
- **Node.js**: Get version [v8.17.0](https://nodejs.org/dist/v8.17.0/win-x86/node.exe).
- **FFmpeg**: [Download](https://ffmpeg.zeranoe.com/builds/win32/static/ffmpeg-3.3.4-win32-static.zip).  
  *(Other versions may also work)*.

---

### 2️⃣ **Build Qt (5.15.16)**

1. Create required directories:
   ```cmd
   mkdir C:\bin
   mkdir C:\bin\vcbuildtrees
   cd C:\bin
   ```
2. Clone and bootstrap **vcpkg**:
   ```cmd
   git clone https://github.com/microsoft/vcpkg.git
   cd vcpkg
   bootstrap-vcpkg.bat
   ```
3. Update your environment variables (CMD session):
   ```cmd
   set VCPKG_ROOT="C:\bin\vcpkg"
   set PATH=%VCPKG_ROOT%;%PATH%
   ```
4. Integrate vcpkg with Visual Studio:
   ```cmd
   vcpkg integrate install
   ```
5. Install Qt dependencies:
   ```cmd
   vcpkg install qt5-base --x-buildtrees-root C:\bin\vcbuildtrees
   vcpkg install qt5-webview qt5-websockets qt5-webglplugin qt5-webengine qt5-webchannel qt5-tools qt5-declarative qt5-quickcontrols2 qt5-quickcontrols --x-buildtrees-root C:\bin\vcbuildtrees
   ```
6. Install additional tools:
   ```cmd
   vcpkg install openssl
   vcpkg install angle
   ```

> **⏳ Note**: Building dependencies, especially `qt5-webengine`, may take over an hour depending on your CPU.

---

### 3️⃣ **Prepare the MPV Library**

- Download the MPV library: [MPV libmpv](https://sourceforge.net/projects/mpv-player-windows/files/libmpv/).
- Use the `mpv-dev-i686` version.  
> **⏳ Note:** The submodule https://github.com/Zaarrg/libmpv already includes .lib, just make sure to unzip the actual .dll for x64 systems.
---

### 4️⃣ **Clone and Configure the Repository**

1. Clone the repository:
   ```cmd
   git clone --recursive git@github.com:Stremio/stremio-shell.git
   cd stremio-shell
   ```
2. Update system PATH:
   ```cmd
   set PATH=C:\bin\vcpkg;C:\bin\vcpkg\installed\x64-windows\bin;%PATH%
   ```

   > **⚠️ Note**: Make sure to also include the vcpkg toolchain either as arg or in cmakelists:
   > ```cmd
   > -DCMAKE_TOOLCHAIN_FILE=C:/bin/vcpkg/scripts/buildsystems/vcpkg.cmake
   > OR IN CMAKELISTS
   > add_definitions(-DCMAKE_TOOLCHAIN_FILE=C:/bin/vcpkg/scripts/buildsystems/vcpkg.cmake)
   > ```
   
3. Download the `server.js` file:
   ```cmd
   powershell -Command Start-BitsTransfer -Source "https://s3-eu-west-1.amazonaws.com/stremio-artifacts/four/v%package_version%/server.js" -Destination server.js
   ```
---

### 5️⃣ **Build the Shell**

1. Make sure to run the following in the `Developer Command Prompt for VS 2022`

> **⏳ Note:** With vcpkg add to ur env `set PATH=%PATH%;C:\bin\vcpkg\installed\x64-windows\tools\qt5\bin` as this contains `windeployqt`
> or run `set PATH=%PATH%;C:\bin\vcpkg\installed\x64-windows\tools\qt5\bin` otherwise step 4 will fail

2. Generate the build files:
   ```cmd
   cmake -G"NMake Makefiles" -DCMAKE_BUILD_TYPE=Release ..
   ```
3. Compile:
   ```cmd
   cmake --build .
   ```

4. Build distributable
   ```cmd
   build_windows_vcpkg.bat {cmake-build-folder}
   
   build_windows_vcpkg.bat cmake-build-release
   ```


> **⏳ Note:** This will create `dist-win` with all necessary files like `node.exe`, `ffmpeg.exe`. Also make sure to have `node.exe` and `stremio-runtime.exe` in `windows\` folder
---

## 📦 Installer (Optional)

1. Download and install [NSIS](https://nsis.sourceforge.io/Download).  
   Default path: `C:\Program Files (x86)\NSIS`.

2. Generate the installer:
   ```cmd
   FOR /F "usebackq delims== tokens=2" %i IN (`type stremio.pro ^| find "VERSION=5"`) DO set package_version=%i
   "C:\Program Files (x86)\NSIS\makensis.exe" windows\installer\windows-installer.nsi
   ```
    - Result: `Stremio %package_version%.exe`.

---

## 🔧 Silent Installation

Run the installer with `/S` (silent mode) and configure via these options:

- `/notorrentassoc`: Skip `.torrent` association.
- `/nodesktopicon`: Skip desktop shortcut.

Silent uninstall:
```cmd
"%LOCALAPPDATA%\Programs\LNV\Stremio-4\Uninstall.exe" /S /keepdata
```

---

✨ **Happy Building!**
