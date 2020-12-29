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

打开AndroidStudio 打开 android/ijkplayer，运行ijkplayer-example，可以连上手机进行测试
```
要注意的是，需要看下自己的手机CPU架构是32位还是64位，并进行variant的切换
```
![切换变体](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fd84740b1f747399fbc96374e33c1df~tplv-k3u1fbpfcp-watermark.image)
