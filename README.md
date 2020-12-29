## 前言

最近由于项目中使用到https的流媒体视频，使用ijkplayer进行播放时报了"Protocol not found"的异常，去项目的Issues里面查找了下，发现是ijkplayer默认不支持https，不过官方已经提供好了编译脚本，需要自己手动编译集成，下面是编译过程，以及遇到的一些坑。

## 编译环境

- OS: Xubuntu 20.04-LTS
```!
任意发行版都可以，只要安装好下面所需的软件和依赖即可
```
- NDK: r10e
```!
ijkplayer官方指定的版本，高版本应该也能编译，需要自己去试，下载地址：

https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
```
- SDK: 30.0.5
```!
版本在23+的应该都可以
```
- AndroidStudio: 4.1.1
```!
版本在2.1.3+的都可以
```
- Yasm: 1.3.0
```!
Yasm是英特尔x86架构下的一个汇编器和反汇编器，安装

sudo apt install yasm
```
- Make: 4.2.1

- gcc: 9.3.0

- g++: 9.3.0
```!
make gcc g++可以手动下载，也可以通过下面的命令自动安装

sudo apt install build-essential
```
- Git: 2.25.1
```!
sudo apt install git
```

## 开始编译

### 1.配置环境变量

编辑~/.bash_profile 或者 ~/.profile，添加下面的环境变量

```
export ANDROID_SDK=<your sdk path>
export ANDROID_NDK=<your ndk path>
export PATH=$PATH:$ANDROID_SDK/tools:$ANDROID_NDK
```

### 2.下载代码

下载ijkplayer主工程
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer
cd ijkplayer
git checkout -B latest k0.8.8
```

初始化项目（会下载ffmpeg、openssl有条件的最好配置好代理，否则会很慢）
```
./init-android.sh
# 用于支持Https
./init-android-openssl.sh
```

修改编译的类型，默认是精简版，完整版支持的格式更多，但是相应的体积更大，根据自己的需要进行选择
```
cd config
rm module.sh
ln -s module-default.sh module.sh #完整版
ln -s module-lite.sh module.sh #精简版
ln -s module-lite-hevc.sh module.sh #包含HEVC的精简版
```
禁用掉linux-perf（不禁用掉会提示，后面编译会报错"./libavutil/timer.h:38:31: fatal error: linux/perf_event.h: No such file or directory"）
```
#此处对应上面选择的编译类型的脚本
vim module-default.sh 

# 在文件的最后加入下面的代码
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-linux-perf"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-bzlib"
```

开始编译
```
cd android/contrib
./compile-openssl.sh clean
#编译openssl
./compile-openssl.sh all

./compile-ffmpeg.sh clean
#编译ffmpeg
./compile-ffmpeg.sh all

cd ..
#编译ijkplayer本体
./compile-ijk.sh all
```

## 运行ijkplayer-example

1.升级gradle版本（新版本AndroidStudio必须要升级，否则会报："Gradle sync failed: Unsupported method: SyncIssue.getMultiLineMessage()"）
```
修改android/ijkplayer/build:gradle文件
classpath 'com.android.tools.build:gradle:4.1.1'

修改android/ijkplayer/gradle/wrapper/gradle-wrapper.properties文件
distributionUrl=https\://services.gradle.org/distributions/gradle-6.5-all.zip
```
2.增加flavor Dimension（新版gradle要求每个flavor必须要有一个Dimension）
```
修改android/ijkplayer/ijkplayer-example/build:gradle文件

flavorDimensions "platform"
productFlavors {
	all32 {
		dimension "platform"
		minSdkVersion 9
	}
	all64 {
		dimension "platform"
		minSdkVersion 21
	}
}
```

3.增加google()依赖仓库（不添加会报错"Could not find com.android.tools.build:aapt2:4.1.1-6503028"）
```
修改android/ijkplayer/build:gradle文件
buildscript {
    repositories {
        google()	//添加google()依赖仓库
        jcenter()
    }
}

allprojects {
    repositories {
        google()	//添加google()依赖仓库
        jcenter()
    }
}
```
至此，AndroidStudio应该可以顺利的编译并运行ijkplayer-example，可以连上手机进行测试
```
要注意的是，需要看下自己的手机CPU架构是32位还是64位，并进行variant的切换
```
![切换变体](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fd84740b1f747399fbc96374e33c1df~tplv-k3u1fbpfcp-watermark.image)

编译好的项目地址：

https://github.com/U2tzJTNE/ijkplayer/tree/latest

最后，附上demo运行成功的截图

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c00999f1d4aa4ff78704d82aebae1278~tplv-k3u1fbpfcp-watermark.image)
