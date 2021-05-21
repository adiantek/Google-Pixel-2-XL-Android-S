# Google Pixel 2 XL - Android S (AOSP)

## Download the repository
I added some repositories needed for build taimen. Building from android-s-beta-1 doesn't work for me:
```
error: packages/modules/NeuralNetworks/shim_and_sl/Android.bp:96:16: unrecognized property "llndk_stubs"
error: packages/modules/NeuralNetworks/shim_and_sl/Android.bp:133:1: unrecognized module type "llndk_library"
error: packages/modules/NeuralNetworks/runtime/Android.bp:163:16: unrecognized property "llndk_stubs"
error: packages/modules/NeuralNetworks/runtime/Android.bp:247:1: unrecognized module type "llndk_library"
```

So, I built everything frmo `master`:
```diff
ubuntu@android:~/android/$ repo init -u https://android.googlesource.com/platform/manifest
ubuntu@android:~/android/$ repo sync
ubuntu@android:~/android/$ # Add now these lines:
ubuntu@android:~/android/.repo/manifests$ git diff
diff --git a/default.xml b/default.xml
index 35e0e3e59..93e489620 100644
--- a/default.xml
+++ b/default.xml
@@ -88,8 +88,11 @@
   <project path="device/google/cuttlefish" name="device/google/cuttlefish" groups="device,pdk" />
   <project path="device/google/cuttlefish_prebuilts" name="device/google/cuttlefish_prebuilts" groups="device,pdk" clone-depth="1" />
   <project path="device/google/fuchsia" name="device/google/fuchsia" groups="device" />
+  <project path="device/google/taimen" name="device/google/taimen" groups="device,taimen" />
   <project path="device/google/trout" name="device/google/trout" groups="device,trout,gull" />
   <project path="device/google/vrservices" name="device/google/vrservices" groups="pdk" />
+  <project path="device/google/wahoo" name="device/google/wahoo" groups="device,generic_fs,wahoo" />
+  <project path="device/google/wahoo-kernel" name="device/google/wahoo-kernel" groups="device,generic_fs,wahoo" clone-depth="1" />
   <project path="device/google_car" name="device/google_car" groups="pdk" />
   <project path="device/linaro/dragonboard" name="device/linaro/dragonboard" groups="device,dragonboard,pdk" />
   <project path="device/linaro/dragonboard-kernel" name="device/linaro/dragonboard-kernel" groups="device,dragonboard,pdk" clone-depth="1" />
@@ -717,6 +720,7 @@
   <project path="hardware/qcom/msm8960" name="platform/hardware/qcom/msm8960" groups="qcom_msm8960,pdk-qcom" />
   <project path="hardware/qcom/msm8994" name="platform/hardware/qcom/msm8994" groups="qcom_msm8994,pdk-qcom" />
   <project path="hardware/qcom/msm8996" name="platform/hardware/qcom/msm8996" groups="qcom_msm8996,pdk-qcom" />
+  <project path="hardware/qcom/msm8998" name="platform/hardware/qcom/msm8998" groups="qcom_msm8998,pdk-qcom" />
   <project path="hardware/qcom/msm8x09" name="platform/hardware/qcom/msm8x09" groups="qcom_msm8x09" />
   <project path="hardware/qcom/msm8x26" name="platform/hardware/qcom/msm8x26" groups="qcom_msm8x26,pdk-qcom" />
   <project path="hardware/qcom/msm8x27" name="platform/hardware/qcom/msm8x27" groups="qcom_msm8x27,pdk-qcom" />
ubuntu@android:~/android/$ repo sync device/google/taimen device/google/wahoo device/google/wahoo-kernel hardware/qcom/msm8998
```
## Download vendor binaries
I downloaded binaries from https://developers.google.com/android/drivers#taimenrp1a.201005.004.a1 - I know that it marked for Android 11, but it works with Android 12 (SIM card doesn't work, there is missing `com.qti.vzw.ims.internal`). Unpack in here:
```
ubuntu@android:~/android$ ./extract-google_devices-taimen.sh && ./extract-qcom-taimen.sh
# "I ACCEPT" twice
Files extracted successfully.
```
## Set up the environment and choose a target
Type `source build/envsetup.sh` and  `lunch aosp_taimen-userdebug`.

## hal_oemlock_default
hal_oemlock_default.te causes problems with parsing configuration (Duplicate declaration of type), so I commented everything in `device/google/wahoo/sepolicy/vendor/hal_oemlock_default.te` and copied this to `system/sepolicy/vendor/hal_oemlock_default.te`:
```diff
ubuntu@android:~/android/device/google/wahoo$ git diff
diff --git a/sepolicy/vendor/hal_oemlock_default.te b/sepolicy/vendor/hal_oemlock_default.te
index bc8ee58a..34c485a4 100644
--- a/sepolicy/vendor/hal_oemlock_default.te
+++ b/sepolicy/vendor/hal_oemlock_default.te
@@ -1,8 +1,8 @@
-type hal_oemlock_default, domain;
-hal_server_domain(hal_oemlock_default, hal_oemlock)
+# type hal_oemlock_default, domain;
+# hal_server_domain(hal_oemlock_default, hal_oemlock)

-allow hal_oemlock_default hal_bootctl_socket:sock_file write;
-allow hal_oemlock_default hal_bootctl:unix_stream_socket connectto;
+# allow hal_oemlock_default hal_bootctl_socket:sock_file write;
+# allow hal_oemlock_default hal_bootctl:unix_stream_socket connectto;

-type hal_oemlock_default_exec, exec_type, vendor_file_type, file_type;
-init_daemon_domain(hal_oemlock_default)
+# type hal_oemlock_default_exec, exec_type, vendor_file_type, file_type;
+# init_daemon_domain(hal_oemlock_default)
```

```diff
ubuntu@android:~/android/system/sepolicy$ git diff
diff --git a/vendor/hal_oemlock_default.te b/vendor/hal_oemlock_default.te
index 8597f2c6f..bc8ee58aa 100644
--- a/vendor/hal_oemlock_default.te
+++ b/vendor/hal_oemlock_default.te
@@ -1,5 +1,8 @@
 type hal_oemlock_default, domain;
 hal_server_domain(hal_oemlock_default, hal_oemlock)

+allow hal_oemlock_default hal_bootctl_socket:sock_file write;
+allow hal_oemlock_default hal_bootctl:unix_stream_socket connectto;
+
 type hal_oemlock_default_exec, exec_type, vendor_file_type, file_type;
 init_daemon_domain(hal_oemlock_default)
```
## /sys/fs/cgroup/
https://android.googlesource.com/platform/system/core/+/2dc28bcfc4edeaef1b5cc1293ffceca5857b47d8/libprocessgroup/processgroup.cpp#421 - never return an error:
```diff
ubuntu@android:~/android/system/core$ git diff
diff --git a/libprocessgroup/processgroup.cpp b/libprocessgroup/processgroup.cpp
index 815d2bb78..8818e9a0a 100644
--- a/libprocessgroup/processgroup.cpp
+++ b/libprocessgroup/processgroup.cpp
@@ -436,21 +436,21 @@ static int createProcessGroupInternal(uid_t uid, int initialPid, std::string cgr

     if (!MkdirAndChown(uid_path, cgroup_mode, cgroup_uid, cgroup_gid)) {
         PLOG(ERROR) << "Failed to make and chown " << uid_path;
-        return -errno;
+        // return -errno;
     }

     auto uid_pid_path = ConvertUidPidToPath(cgroup.c_str(), uid, initialPid);

     if (!MkdirAndChown(uid_pid_path, cgroup_mode, cgroup_uid, cgroup_gid)) {
         PLOG(ERROR) << "Failed to make and chown " << uid_pid_path;
-        return -errno;
+        // return -errno;
     }

     auto uid_pid_procs_file = uid_pid_path + PROCESSGROUP_CGROUP_PROCS_FILE;

     int ret = 0;
     if (!WriteStringToFile(std::to_string(initialPid), uid_pid_procs_file)) {
-        ret = -errno;
+        // ret = -errno;
         PLOG(ERROR) << "Failed to write '" << initialPid << "' to " << uid_pid_procs_file;
     }
```

Cgroups help with isolating resources, disabling it can be a **security risk**, so it should be fixed. There is a problem with mounting in libprocessgroup:
```
E         : c5      1 libprocessgroup: Failed to mount blkio cgroup: No such file or directory
W         : c5      1 libprocessgroup: Failed to setup blkio cgroup
E         : c5      1 libprocessgroup: lchown() failed for /sys/fs/cgroup/.: Operation not permitted
E         : c5      1 libprocessgroup: Failed to create directory for cgroup2 cgroup
E         : c5      1 libprocessgroup: Failed to mount cgroup2 cgroup: Operation not permitted
W         : c5      1 libprocessgroup: Failed to setup cgroup2 cgroup
E         : c5      1 libprocessgroup: lchown() failed for /sys/fs/cgroup/./.: Operation not permitted
E         : c5      1 libprocessgroup: change of ownership or mode failed for /sys/fs/cgroup/.: Operation not permitted
E         : c5      1 libprocessgroup: Failed to create directory for freezer cgroup
W         : c5      1 libprocessgroup: Failed to setup freezer cgroup
E         : c5      1 libprocessgroup: Failed to mount memory cgroup: No such file or directory
W         : c5      1 libprocessgroup: Failed to setup memory cgroup
```
PS. I reported it: https://issuetracker.google.com/issues/188695839
## bpfloader
When I flashed I saw in logcat:
```
E bpfloader: Failed to load object: /apex/com.android.tethering/etc/bpf/offload.o, ret: Function not implemented
E bpfloader: === CRITICAL FAILURE LOADING BPF PROGRAMS FROM /apex/com.android.tethering/etc/bpf/ ===
E bpfloader: If this triggers reliably, you're probably missing kernel options or patches.
E bpfloader: If this triggers randomly, you might be hitting some memory allocation problems or startup script race.
E bpfloader: --- DO NOT EXPECT SYSTEM TO BOOT SUCCESSFULLY ---
```
Unfortunately, netd can't start if bpf.progs_loaded will not set. I removed sleep and return from: https://android.googlesource.com/platform/system/bpf/+/67fa2073ffbeb0baa6ad20cc9f309853efb1b536/bpfloader/BpfLoader.cpp#124
```diff
ubuntu@android:~/android/system/bpf$ git diff
diff --git a/bpfloader/BpfLoader.cpp b/bpfloader/BpfLoader.cpp
index 7a68894..0ad8f2a 100644
--- a/bpfloader/BpfLoader.cpp
+++ b/bpfloader/BpfLoader.cpp
@@ -121,8 +121,6 @@ int main() {
             ALOGE("If this triggers randomly, you might be hitting some memory allocation "
                   "problems or startup script race.");
             ALOGE("--- DO NOT EXPECT SYSTEM TO BOOT SUCCESSFULLY ---");
-            sleep(20);
-            return 2;
         }
     }
```
If you want fix it properly I give you a hint - there is a loader: https://android.googlesource.com/platform/system/bpf/+/67fa2073ffbeb0baa6ad20cc9f309853efb1b536/libbpf_android/Loader.cpp#688

## Build it!
Type `RELAX_USES_LIBRARY_CHECK=true m` and wait. We must use RELAX_USES_LIBRARY_CHECK because vendor/qcom/taimen/proprietary/ims.apk requires com.qti.vzw.ims.internal.
```
============================================
PLATFORM_VERSION_CODENAME=S
PLATFORM_VERSION=S
TARGET_PRODUCT=aosp_taimen
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-a
TARGET_CPU_VARIANT=cortex-a73
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv8-a
TARGET_2ND_CPU_VARIANT=cortex-a73
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-5.8.0-1027-kvm-x86_64-Ubuntu-20.10
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=AOSP.MASTER
OUT_DIR=out
PRODUCT_SOONG_NAMESPACES=device/google/taimen device/google/wahoo vendor/google/camera hardware/google/camera hardware/google/pixel hardware/qcom/msm8998 vendor/qcom/taimen/proprietary
============================================
```

## Create update.zip
Go to out/target/product/taimen/images and type `zip update.zip android-info.txt boot.img dtbo.img vbmeta.img system.img system_other.img vendor.img`. Now you can flash update.zip with `fastboot -w flash update.zip`. Optionally, you can flash bootloader.img and radio.img using `fastboot flash bootloader/radio bootloader/radio.img` (you can't flash bootloader/radio using ZIP).

## Modify system.img after compilation and before flashing
You must unset unshare_blocks from system.img to mount it with rw:
```
simg2img system.img system.raw.img
e2fsck -y -E unshare_blocks system.raw.img
mkdir system/
mount system.raw.img system/
# modify your system, for testing I recommend set ro.adb.secure=0 in system/build.prop to get logcat access
umount system/
img2simg system.raw.img system.img
fastboot flash system system.img
```

## How to undo everything?
Download https://dl.google.com/dl/android/aosp/taimen-rp1a.201005.004.a1-factory-2f5c4987.zip and flash it using `flash-all.bat`/`flash-all.sh`.
