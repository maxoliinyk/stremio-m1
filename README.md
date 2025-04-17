# Building Stremio Shell from Source on Apple Silicon

> **Tested on:** M1, macOS Sequoia  
> Thanks to everyone in [this Reddit thread](https://www.reddit.com/r/Stremio/comments/15gijoq/stremio_for_apple_silicon_m1_m2_etc/) and [zavan](https://github.com/zavan)

## Prerequisites

- [git](https://git-scm.com/)
- [Homebrew](https://brew.sh/)
- Xcode Command Line Tools

---

## 1. Install and Link Dependencies

```sh
brew install mpv ffmpeg node cmake qt@5 openssl
brew install --cask qt-creator
brew link openssl --force
brew link qt5 --force
```

---

## 2. Clone the Repository

```sh
git clone --recursive https://github.com/Stremio/stremio-shell.git
cd stremio-shell
```

---

## 3. Make Required Changes

Edit `./CMakeLists.txt` and `./mac/finalize.sh` as follows:

<details>
<summary><strong>CMakeLists.txt</strong></summary>

```diff
@@ -44,10 +44,10 @@ endif()
 if(APPLE)
   list(APPEND SOURCES images/stremio.icns)
   set_source_files_properties(images/stremio.icns PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
-  set(MPV_LIBRARY_mpv ${CMAKE_CURRENT_SOURCE_DIR}/deps/lib/libmpv.dylib)
+  set(MPV_LIBRARY_mpv /opt/homebrew/opt/mpv/lib/libmpv.dylib)
   set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath,@executable_path/../Frameworks")
   add_definitions("-pipe")
-  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -arch x86_64")
+  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -arch arm64")
   set(ENV{OPENSSL_ROOT_DIR} $ENV{OPENSSL_BIN_PATH})
 endif()
```
</details>

<details>
<summary><strong>mac/finalize.sh</strong></summary>

```diff
@@ -6,20 +6,14 @@ TAG=${1:-master}
 
 DEST_DIR=./stremio.app/Contents/MacOS
 
-cp ./mac/ffmpeg $DEST_DIR/
-cp ./mac/ffprobe $DEST_DIR/
-cp $(which node) $DEST_DIR/
+cp -f $(which ffmpeg) $DEST_DIR/
+cp -f $(which ffprobe) $DEST_DIR/
+cp -f $(which node) $DEST_DIR/
 chmod +w $DEST_DIR/ffmpeg
 chmod +w $DEST_DIR/ffprobe
 chmod +w $DEST_DIR/node
 
 mkdir -p ./stremio.app/Contents/Frameworks
-cp ./deps/libmpv/mac/lib/*.dylib ./stremio.app/Contents/Frameworks/
-
-# https://bugreports.qt.io/browse/QTBUG-57265
-# you don't want to be using always-overwrite in any version until Qt 5.11.3
-macdeployqt ./stremio.app -executable=./stremio.app/Contents/MacOS/ffmpeg -executable=./stremio.app/Contents/MacOS/ffprobe -executable=./stremio.app/Contents/MacOS/node
 
 SHELL_VERSION=$(git grep -hoP '^\s*VERSION\s*=\s*\K.*$' HEAD -- stremio.pro)
 curl $(cat ./server-url.txt) > $DEST_DIR/server.js
-# ./mac/fix_osx_deps.sh "./stremio.app/Contents/Frameworks" "@executable_path/../Frameworks"
```
</details>

---

## 4. Configure, Build, and Pack

```sh
qmake CONFIG+=sdk_no_version_check
cmake -DCMAKE_PREFIX_PATH=/opt/homebrew/opt/qt@5 .
cmake --build .
chmod +x ./mac/finalize.sh && ./mac/finalize.sh
chmod +x ./mac/pack.sh && ./mac/pack.sh
```

---

## 5. Finish

- The `.dmg` will be created (e.g., `Stremio 4.4.168.dmg`)
- Run it to install.
- **Don’t forget to enable hardware acceleration in Stremio settings!**

---
