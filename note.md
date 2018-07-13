Installation
============
This page gives instructions on how to build and install the tvm package from
scratch on various systems. It consists of two steps:

1. First build the shared library from the C++ codes (`libtvm.so` for linux/osx and `libtvm.dll` for windows).
2. Setup for the language packages (e.g. Python Package).

To get started, clone tvm repo from github. It is important to clone the submodules along, with ``--recursive`` option.



    git clone --recursive https://github.com/dmlc/tvm

```
COMMENTS:
To get jdk path: /usr/libexec/java_home -V
To use maven3 and gradle
Set env value:
export JAVA_HOME = path/to/jdk1.8
export PATH=$PATH:JAVA_HOME/bin
```


Build the Shared Library
------------------------

Our goal is to build the shared libraries:

- On Linux the target library are `libtvm.so, libtvm_topi.so`
- On OSX the target library are `libtvm.dylib, libtvm_topi.dylib`
- On Windows the target library are `libtvm.dll, libtvm_topi.dll`

`COMMENTS: I didn't use this but followed the previous instruction`

    sudo apt-get update
    sudo apt-get install -y python python-dev python-setuptools gcc libtinfo-dev zlib1g-dev

The minimal building requirements are

- A recent c++ compiler supporting C++ 11 (g++-4.8 or higher)
- CMake 3.5 or higher
- We highly recommend to build with LLVM to enable all the features.
- It is possible to build without llvm dependency if we only want to use CUDA/OpenCL

We use cmake to build the library.
The configuration of tvm can be modified by `config.cmake`.


- First, check the cmake in your system, you do not have cmake   
```COMMENTS: brew install cmake```
  you can obtain the latest version from [official website](https://cmake.org/download/)
- First create a build directory, copy the ``cmake/config.cmake`` to the directory.

 
		
      cd ~/tvm 
      mkdir build
      cp cmake/config.cmake build
	  
      COMMETS: Three main changes to use llvm, rpc and opencl
      set(USE_LLVM /Users/xin/Documents/x-lab/llvm/bin/llvm-config)
      set(USE_RPC ON)
      set(USE_OPENCL ON)
			
- Edit ``build/config.cmake`` to customize the compilation options

- TVM optionally depends on LLVM. LLVM is required for CPU codegen that needs LLVM.

  - LLVM 4.0 or higher is needed for build with LLVM. Note that verison of LLVM from default apt may lower than 4.0.
  - Since LLVM takes long time to build from source, you can download pre-built version of LLVM from
    [LLVM Download Page](http://releases.llvm.org/download.html).


    - Unzip to a certain location, modify ``build/config.cmake`` to add ``set(USE_LLVM /path/to/your/llvm/bin/llvm-config)``
    - You can also directly set ``set(USE_LLVM ON)`` and let cmake search for a usable version of LLVM.

  - You can also use [LLVM Nightly Ubuntu Build](https://apt.llvm.org/)

    - Note that apt-package append ``llvm-config`` with version number.
      For example, set ``set(LLVM_CONFIG llvm-config-4.0)`` if you installed 4.0 package

- We can then build tvm and related libraries.

      cd build
      cmake ..
      make -j4

If everything goes well, we can go to the specific language installation section.



Python Package Installation
---------------------------

The python package is located at python
There are several ways to install the package:

1. Set the environment variable `PYTHONPATH` to tell python where to find
   the library. For example, assume we cloned `tvm` on the home directory
   `~`. then we can added the following line in `~/.bashrc`.
   It is **recommended for developers** who may change the codes.
   The changes will be immediately reflected once you pulled the code and rebuild the project (no need to call ``setup`` again)


       export PYTHONPATH=/path/to/tvm/python:/path/to/tvm/topi/python:/path/to/tvm/nnvm/python:${PYTHONPATH}


2. Install tvm python bindings by `setup.py`:


       # install tvm package for the current user
       # NOTE: if you installed python via homebrew, --user is not needed during installaiton
       #       it will be automatically installed to your user directory.
       #       providing --user flag may trigger error during installation in such case.
       
       export MACOSX_DEPLOYMENT_TARGET=10.9  # This is required for mac to avoid symbol conflicts with libstdc++
       cd tvm
       cd python; python setup.py install --user; cd ..
       cd topi/python; python setup.py install --user; cd ../..
       cd nnvm/python; python setup.py install --user; cd ../..
      
Android TVM RPC
========

This folder contains Android RPC app that allows us to launch an rpc server on a Android device and connect to it through python script and do testing on the python side as normal TVM RPC.
```
COMMENTS: 
Download NDK for toolchain
You may need set env value: 
export PATH=/Users/xin/Library/Android/sdk/ndk-bundle:$PATH
```
You will need JDK, [Android NDK](https://developer.android.com/ndk) and an Android device to use this.

Build and Installation
-----


* Build APK
```
COMMENTS:
brew install gradle
gradle for bulid apk (Set environment value export JAVA_HOME = ~/jdk1.8 PATH=$PATH:JAVA_HOME/bin)
jdk1.8 is compatible for maven3 and gradle to run
```
We use [Gradle](https://gradle.org) to build. Please follow [the installation instruction](https://gradle.org/install) for your operating system.

Before you build the Android application, please refer to [TVM4J Installation Guide](https://github.com/dmlc/tvm/blob/master/jvm/README.md) and install tvm4j-core to your local maven repository. You can find tvm4j dependency declare in `app/build.gradle`. Modify it if it is necessary.

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.0.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    compile 'com.android.support:design:26.0.1'
    compile 'ml.dmlc.tvm:tvm4j-core:0.0.1-SNAPSHOT'
    testCompile 'junit:junit:4.12'
}
```


* Build with OpenCL

This application does not link any OpenCL library unless you configure it to. In `app/src/main/jni/make` you will find JNI Makefile config `config.mk`. Copy it to `app/src/main/jni` and modify it.

```bash
cd apps/android_rpc/app/src/main/jni
cp make/config.mk .
```

Here's my setting for `config.mk`.

```makefile
APP_ABI = arm64-v8a
 
APP_PLATFORM = android-26
 
# whether enable OpenCL during compile
USE_OPENCL = 1
 
# the additional include headers you want to add, e.g., SDK_PATH/adrenosdk/Development/Inc
ADD_C_INCLUDES = /Users/xin/Documents/x-lab/Android/OpenCL-Headers
 
# the additional link libs you want to add, e.g., ANDROID_LIB_PATH/libOpenCL.so
ADD_LDLIBS = /Users/xin/Documents/x-lab/Android/libOpenCL.so
```

```
COMMENTS:
#In which, ADD_C_INCLUDES is the standard OpenCL headers, you can download from: 
https://github.com/KhronosGroup/OpenCL-Headers
#In which, ADD_LDLIBS is the mobile phone's opencl lib, You can use adb pull to get the file to your MacBook:
adb pull /system/vendor/lib64/libOpenCL.so ./
```

Note that you should specify the correct GPU development headers for your android device. Run `adb shell dumpsys | grep GLES` to find out what GPU your android device uses. It is very likely the library (libOpenCL.so) is already present on the mobile device. For instance, I found it under `/system/vendor/lib64`. You can do `adb pull /system/vendor/lib64/libOpenCL.so ./` to get the file to your desktop.

After you setup the `config.mk`, follow the instructions below to build the Android package.

Now use Gradle to compile JNI, resolve Java dependencies and build the Android application together with tvm4j. Run following script to generate the apk file.

```bash
export ANDROID_HOME=[Path to your Android SDK, e.g., ~/Android/sdk]
cd apps/android_rpc
gradle clean build
```

```
COMMENTS:
It's okay for use debug.apk to test. And remember to uninstall the previous app before reinstall.
```

In `app/build/outputs/apk` you'll find `app-release-unsigned.apk`, use `dev_tools/gen_keystore.sh` to generate a signature and use `dev_tools/sign_apk.sh` to get the signed apk file `app/build/outputs/apk/tvmrpc-release.apk`.

Upload `tvmrpc-release.apk` to your Android device and install it.

Cross Compile and Run on Android Devices
 ==========

Architecture and Android Standalone Toolchain
--------

In order to cross compile a shared library (.so) for your android device, you have to know the target triple for the device. (Refer to [Cross-compilation using Clang](https://clang.llvm.org/docs/CrossCompilation.html) for more information). Run `adb shell cat /proc/cpuinfo` to list the device's CPU information.

Now use NDK to generate standalone toolchain for your device. For my test device, I use following command.
		`COMMENTS:find make-standalone-toolchain.sh in ndk`
```bash
cd /opt/android-ndk/build/tools/
./make-standalone-toolchain.sh --platform=android-26 --use-llvm --arch=arm64 --install-dir=/opt/android-toolchain-arm64
```

If everything goes well, you will find compile tools in `/opt/android-toolchain-arm64/bin`. For example, `bin/aarch64-linux-android-g++` can be used to compile C++ source codes and create shared libraries for arm64 Android devices.

Cross Compile and Upload to the Android Device
---------

First start a proxy server using 
```
$ python -m tvm.exec.rpc_proxy

COMMENTS:
And this will show off:
INFO:root:RPCProxy: client port bind to 0.0.0.0:9090
INFO:root:RPCProxy: Websock port bind to 8888

``` 
and make your Android device connect to this proxy server via TVM RPC application.

```
COMMENTS:
Open the Android app and type in the IP addr/port/key you set in python script.
You may need $ ifconfig to see IP addr.
If the Android device is connect, it will show:

INFO:root:RPCProxy: client port bind to 0.0.0.0:9090
INFO:root:RPCProxy: Websock port bind to 8888
INFO:root:Handler ready TCPSocketProxy:10.66.65.194:server:android

```
Then checkout [android\_rpc/tests/android\_rpc\_test.py](https://github.com/dmlc/tvm/blob/master/apps/android_rpc/tests/android_rpc_test.py) and run,
		``
		
```
COMMENTS: I write this in .py
os.environ["TVM_ANDROID_RPC_PROXY_HOST"] = "0.0.0.0"
os.environ["TVM_NDK_CC"] = "/Users/xin/my-android-toolchain/android-toolchain-arm64/bin/aarch64-linux-android-g++"
proxy_host = os.environ["TVM_ANDROID_RPC_PROXY_HOST"]
proxy_port = 9090
key = "android"
```
or you can do as follow
```
# Specify the proxy host
export TVM_ANDROID_RPC_PROXY_HOST=0.0.0.0
# Specify the standalone Android C++ compiler
export TVM_NDK_CC=/opt/android-toolchain-arm64/bin/aarch64-linux-android-g++
python android_rpc_test.py
```

This will compile TVM IR to shared libraries (CPU and OpenCL) and run vector additon on your Android device. On my test device, it gives following results.

```bash
TVM: Initializing cython mode..
[01:21:43] src/codegen/llvm/codegen_llvm.cc:75: set native vector to be 32 for target aarch64
[01:21:43] src/runtime/opencl/opencl_device_api.cc:194: Initialize OpenCL platform 'Apple'
[01:21:43] src/runtime/opencl/opencl_device_api.cc:214: opencl(0)='Iris' cl_device_id=0x1024500
[01:21:44] src/codegen/llvm/codegen_llvm.cc:75: set native vector to be 32 for target aarch64
Run GPU test ...
0.000155807 secs/op
Run CPU test ...
0.00139824 secs/op
```

You can define your own TVM operators and test via this RPC app on your Android device to find the most optimized TVM schedule.

Run MobileNet on Android devices
========
- This test is modified from [Benchmarking Deep Neural Networks on ARM CPU/GPU](https://github.com/merrymercy/tvm-mali)
- The test code is [MobileNet_AS_RPC.py](https://github.com/Xiraaa/tvm/blob/master/tutorials/mobileNet_AS_RPC.py)
- If you successfully run the test in the previous part, you just need to modify part of the code.
1. Set to be address of tvm proxy.
```
os.environ["TVM_ANDROID_RPC_PROXY_HOST"] = "0.0.0.0"
os.environ["TVM_NDK_CC"] = "/Users/xin/my-android-toolchain/android-toolchain-arm64/bin/aarch64-linux-android-g++"
proxy_host = os.environ["TVM_ANDROID_RPC_PROXY_HOST"]
proxy_port = 9090
key = "android"
```
2. Change target configuration. Run `adb shell cat /proc/cpuinfo` to find the arch.
```
arch = "arm64"
target = "llvm -target=%s-linux-android" % arch
```
3. Load model(Existing bug: Setting dtype = "float16" leads to crash)
```
net, params = nnvm.testing.mobilenet.get_workload(
        batch_size=1, image_shape=image_shape, dtype=dtype)
```
4. Compile
```
opt_level = 2 if dtype == 'float32' else 1
with nnvm.compiler.build_config(opt_level=opt_level):
graph, lib, params = nnvm.compiler.build(
    net, tvm.target.mali(), shape={"data": data_shape}, params=params,
    dtype=dtype, target_host=target)
```

5. Upload model to remote device(The extension of export file must be .so)
```
tmp = util.tempdir()
lib_fname = tmp.relpath('net.so')
lib.export_library(lib_fname, ndk.create_shared)

remote = rpc.connect(proxy_host, proxy_port, key=key)
remote.upload(lib_fname)
ctx = remote.cl(0)
rlib = remote.load_module('net.so')
rparams = {k: tvm.nd.array(v, ctx) for k, v in params.items()}
print('Run GPU test ...')
```
6. Create graph runtime
```
module = runtime.create(graph, rlib, ctx)
module.set_input('data', tvm.nd.array(np.random.uniform(size=(data_shape)).astype(dtype)))
module.set_input(**rparams)
```

7. The num of runs for warm up and test(Exsiting bug: when num_test is over 300, program will crash)
```
num_warmup = 50
num_test   = 300
warm_up_timer = module.module.time_evaluator("run", ctx, num_warmup)
warm_up_timer()
ftimer = module.module.time_evaluator("run", ctx, num_test)
prof_res = ftimer()
print("backend: TVM-mali\tmodel: %s\tdtype: %s\tcost:%.4f" % ("mobileNet", dtype, prof_res.mean))
```
