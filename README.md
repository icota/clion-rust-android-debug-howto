# How to debug Rust apps on Android with CLion

Say you've built and deployed a Rust library on Android following [these steps](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html). In order to debug via rust-gdb or graphically with CLion do the following:

- Stop gradle from stripping your Rust .so files of debug symbols by adding `packagingOptions.doNotStrip "**/*.so"` to `app/build.gradle`

- Push the gdbserver binary from Android NDK's `prebuild` directory to the device:
```
~/Android/Sdk/ndk-bundle/prebuilt/android-x86_64/gdbserver$ ~/Android/Sdk/platform-tools/adb push gdbserver /data/local/tmp
~/Android/Sdk/platform-tools/adb shell "chmod 777 /data/local/tmp/gdbserver"
```

- Restart adbd as root:
```
~/Android/Sdk/platform-tools/adb root
```

- Forward a port that you will use for gdbserver:
```
~/Android/Sdk/platform-tools/adb forward tcp:1337 tcp:1337
```

- Attach gdbserver to the process you wish to debug:
```
~/Android/Sdk/platform-tools/adb shell
generic_x86_64:/ # su
generic_x86_64:/ # set enforce 0
generic_x86_64:/ # /data/local/tmp/gdbserver :1337 --attach $(ps -A | grep your.android.application.id | awk '{print $2}')
```

- Either run `rust-gdb` or set up CLion's "Remote GDB" configuration with:
```
target remote tcp:localhost:1337
```

- Enjoy!
