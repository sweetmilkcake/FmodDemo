# FmodDemo
这是一个演示如何使用Fmod的Demo.

因为fmod的官方的Example使用的Visual Studio来构建工程，所以FmodDemo主要是演示如何把官方的Example移植到Android Studio当中.

## You can Learn
* Gradle配置
* Cmake配置
* NDK相关知识

## Download
https://github.com/sweetmilkcake/FmodDemo/releases

## Environment
Android Studio 3.1.3 (确保可以编译NDK)
![sdk_tools](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/sdk_tools.png)

## How to Port
1. 创建一个新的项目，支持C++
![c++support](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/c++support.png)

2. 选择C++支持的版本和依赖库
![c++14](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/c++14.png)

3. 把api/lowlevel/lib中的文件拷贝到libs目录下
![libs](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/libs.png)

4. 右键Add As Libs，添加fmod.jar包支持
![fmodjar](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/fmodjar.png)

5. 把api/lowlevel/inc目录整个拷贝到cpp目录下
![inc](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/inc.png)
![inc_as](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/inc_as.png)

6. 把api/lowlevel/examples/media目录下的所有文件拷贝到assets
![media](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/media.png)
![media_as](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/media_as.png)

7. 配置Gradle文件
![Gradle](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/Gradle.png)
```
arguments "-DANDROID_STL=stlport_shared"
```
表示支持stlport_shared，因为MainActivity.java当中需要System.loadLibrary("stlport_shared");

```
ndk {
    // Specifies the ABI configurations of your native
    // libraries Gradle should build and package with your APK.
    abiFilters 'x86', 'armeabi-v7a', 'arm64-v8a'
}
```
根据libs中的ABI支持，添加到Gradle中，因为我所使用的NDK版本为R17，不再支持armeabi
![libs](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/libs.png)
>ABIs [armeabi] are not supported for platform. Supported ABIs are [armeabi-v7a, arm64-v8a, x86, x86_64].

```
sourceSets {
    main {
        jniLibs.srcDirs 'libs'
    }
}
```
Android Studio默认动态库存放的位置在app/src/main/jniLibs目录下，这里统一放到libs目录下，所以需要重新设置jniLibs的目录为libs

8. CMake配置(CMakeLists.txt)
Cmake配置主要是让编译系统能找到对应的文件进行编译，语法比较简单不细说
```make
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Sets lib_src_DIR to the path of the target CMake project.
set( distribution_DIR ${CMAKE_SOURCE_DIR}/libs )

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             play_sound

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/play_sound.cpp
             src/main/cpp/common_platform.cpp
             src/main/cpp/common.cpp )

add_library( # Sets the name of the library.
             fmod

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             IMPORTED )

set_target_properties( # Specifies the target library.
                       fmod

                       # Specifies the parameter you want to define.
                       PROPERTIES IMPORTED_LOCATION

                       # Provides the path to the library you want to import.
                       ${distribution_DIR}/${ANDROID_ABI}/libfmod.so )

add_library( # Sets the name of the library.
             fmodL

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             IMPORTED )

set_target_properties( # Specifies the target library.
                       fmodL

                       # Specifies the parameter you want to define.
                       PROPERTIES IMPORTED_LOCATION

                       # Provides the path to the library you want to import.
                       ${distribution_DIR}/${ANDROID_ABI}/libfmodL.so )

# Specifies a path to native header files.
include_directories( src/main/cpp/inc/ )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       play_sound
                       fmod
                       fmodL

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

9. AndroidManifest.xml添加相应权限
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

10. 替换MainActivity (把lowlevel中example的MainActivity.java替换到工程中)
![mainactivity](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/mainactivity.png)

11. 修正包名
如果你当初用这个包名org/fmod/example创建工程，那么这一步可以跳过。
![package](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/package.png)
![jni](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/jni.png)
>其实这里就是java代码和cpp代码的调用接口。

12. 效果
如果出现下面界面，那就表示移植成功
![show](https://github.com/sweetmilkcake/FmodDemo/blob/master/Screenshots/show.png)

## Thanks
* https://www.fmod.com/
* ......
