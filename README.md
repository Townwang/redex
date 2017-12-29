ReDex：一个Android的字节码优化器

ReDex是一个最初在Facebook开发的Android字节码（dex）优化器。它提供了读取，写入和分析.dex文件的框架，以及一组使用此框架来优化字节码的优化传递。由ReDex优化的APK应该比它的源更小更快。

快速入门指南

依赖

我们使用包管理器来解决第三方库的依赖关系。

Mac OS X

您将需要安装命令行工具的Xcode。要获得命令行工具，请使用：

xcode-select --install
使用自制程序安装依赖关系：

brew install autoconf automake libtool python3
brew install boost jsoncpp
Ubuntu 14.04 LTS（64位）

sudo apt-get install \
    g++ \
    automake \
    autoconf \
    autoconf-archive \
    libtool \
    libboost-all-dev \
    liblz4-dev \
    liblzma-dev \
    make \
    zlib1g-dev \
    binutils-dev \
    libjemalloc-dev \
    libiberty-dev \
    libjsoncpp-dev
实验：Windows 10（64位）

您需要Visual Studio 2017.Visual Studio 2015也是可能的，但需要修复一些C ++编译错误。我们使用vcpkg作为依赖关系。从他们的文档安装vcpkg ：

cd c:\tools
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
安装必要的库x64-windows-static：

.\vcpkg install boost --triplet x64-windows-static
.\vcpkg install zlib --triplet x64-windows-static
.\vcpkg install jsoncpp --triplet x64-windows-static
下载，构建和安装

从GitHub获取ReDex：

git clone https://github.com/facebook/redex.git
cd redex
现在，使用autoconf和make来构建ReDex。

# if you're using gcc, please use gcc-4.9
autoreconf -ivf && ./configure && make -j4
sudo make install
实验：Mac，Linux和Windows的CMake

或者，使用CMake构建。请注意，当前CMakeLists.txt只实现了redex-all二进制的规则。我们将很快支持安装和测试。

生成构建文件。默认情况下，它使用Makefile：

# Assume you are in redex directory
mkdir build-cmake
cd build-cmake
# .. is the root source directory of Redex
cmake ..
如果你喜欢忍者构建系统：

cmake .. -G Ninja
在Windows上，首先CMAKE_TOOLCHAIN_FILE从输出中获取"vcpkg integrate install"，然后：

cmake .. -G "Visual Studio 15 2017 Win64"
 -DVCPKG_TARGET_TRIPLET=x64-windows-static
 -DCMAKE_TOOLCHAIN_FILE="C:/tools/vcpkg/scripts/buildsystems/vcpkg.cmake"
构建redex-all：

cmake --build .
在Windows上，您可以从Visual Studio构建。Redex.sln已经生成。

你应该看到一个redex-all可执行文件，可执行文件应该显示大约45次。

./redex-all --show-passes
测试

或者，您可以运行我们的单元测试套件。我们使用gtest，它是通过安装脚本下载的。

./test/setup.sh
cd test
make check
用法

要使用ReDex，首先构建您的应用程序并为其找到APK。然后运行：

redex path/to/your.apk -o path/to/output.apk
如果您需要关于每个传递的一些统计信息，您可以打开跟踪：

export TRACE=1
结果output.apk应该比输入更小更快。请享用！

文档

现在我们只有有限的文档描述了几个Redex优化过程的示例以及Redex（包括Docker）的部署。

更多信息

使用ReDex优化Android字节码的博客提供了Redex项目的概述。

问题

GitHub上的问题被分配了优先级，这反映了它们的紧迫性以及它们可能会被解决的时间。

P0：现在安装！一个严重的问题，现在应该有人在这个问题上工作。
P1：高优先级。一个人应该积极努力的重要问题。
P2：中等优先。队列中很重要的一个问题是要尽快处理。
P3：低优先级。一个重要的问题，可能会在稍后处理。
P4：愿望清单：一个有优点但低优先级的问题，如果在合理的时间之后没有解决，那么这个问题就有可能被修剪。
执照

ReDex获得BSD许可。我们还提供额外的专利授权。

常问问题

我得到“无法找到zipalign。请参阅README.md解决此问题”

zipalign是与Android SDK捆绑在一起的优化步骤。你需要告诉redex在哪里找到它。例如，如果您安装了SDK /path/to/android/sdk，请尝试：

ANDROID_SDK=/path/to/android/sdk redex [... arguments ...]
您也可以添加zipalign到您的PATH，例如：

PATH=/path/to/android/sdk/build-tools/xx.y.zz:$PATH redex [... arguments ...]
我的应用程序无法安装 Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES]

运行redex后，您需要重新签署您的应用程序。您可以使用以下说明手动重新签名：http : //developer.android.com/tools/publishing/app-signing.html#signing-manually。

你也可以告诉redex为你签字。如果你想用调试键签名，你可以简单地做：

redex --sign [ ... arguments ...]
如果你想用你的释放键签名，你需要提供相应的参数：

--sign Sign the apk after optimizing it
-s [KEYSTORE], --keystore [KEYSTORE]
-a [KEYALIAS], --keyalias [KEYALIAS]
-p [KEYPASS], --keypass [KEYPASS]
我的应用程序崩溃与MethodNotFoundException，ClassNotFoundException，NoSuchFieldException，或者类似的东西。我该如何解决？

Redex可能已被删除或重命名。Redex对于删除它认为无法访问的东西非常积极。但是，Redex通常不知道实体可以到达的反射或其他复杂的方式。

以下是确保Redex不会删除或重命名内容的方法：

注释您想要保留的任何类，方法或字段@DoNotStrip。

添加到您的redex配置（在json的最高级别），以防止删除：

"keep_annotations": [
  "Lcom/path/to/your/DoNotStrip;"
]
并将其添加到您的配置，以防止重命名：

"RenameClassesPassV2" : {
  "dont_rename_annotated": [
    "Lcom/path/to/your/DoNotStrip;"
  ]
}
并定义DoNotStrip：

package com.path.to.your;
public @interface DoNotStrip {}
这与ProGuard相比如何？

ReDex在概念上类似于ProGuard，因为它们都优化了字节码。然而，ReDex优化.dex字节码，而ProGuard在将.class字节码降到.dex之前优化.class字节码。在.dex上进行操作有时是一个优点：可以考虑一个内联候选方法所使用的虚拟寄存器的数量，并且可以控制dex文件中类的布局。但ProGuard具有许多ReDex不具备的功能（例如，ReDex不会删除ProGuard所没有的未使用的方法参数）。

在我们看来，比较ReDex和ProGuard有点儿苹果和橘子，因为我们专注于在ProGuard之上增加价值的优化。我们使用这两种工具来优化Facebook应用程序。我们报告的性能和尺寸改进（对于dex尺寸和冷启动时间的约25％）是基于在已经使用ProGuard优化的应用程序上使用ReDex。没有ProGuard，我们不打算测量性能。

如何DexGuard？

DexGuard在dex上运行，但是由于它是封闭源代码，我们还没有对它进行评估。我们不使用它在Facebook，我们没有计划开始。
