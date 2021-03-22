# Windows10下编译YGOPRO（2021.03.21）

今天来录一下windows10下编译ygopro的过程和踩过的坑。我会尽量把编译过程中我遇到的问题都复现一遍，以便大家在遇到问题时有解决方案可以参考。整理之后，我会把这个文档传到github上。

## 0. 默认读者已经掌握的技能

- 能够打开Windows Powershell并执行简单操作
  - 使用cd命令切换目录
  - 知道`..`代表上级目录、`Tab`可以自动补全等
  - 推荐下载并安装[VS Code](https://code.visualstudio.com/)
- 能够下载并安装[WinRAR](www.win-rar.com)
- 能够下载并安装[git](https://git-scm.com/download/)
  - 虽然不安装，直接下载源码也可以，但还是建议安装一下
- 能够下载并安装[Visual Studio 2019](https://visualstudio.microsoft.com/zh-hans/downloads/)
- 能够下载Premake4，并添加到环境变量
  - 官方教程中给的链接下载的是premake源码，可以直接下载编译好的[premake4命令](https://premake.github.io/)
- 能够打开[官方教程](https://github.com/Fluorohydride/ygopro/wiki/build)

## 1. 下载ygopro源码

切换到一个舒适的工作目录并下载源码，我的目录是：
>D:\ygospace\
>>ygo_compile.md（此Markdown文档，本次编译记录）  
>>ygopro（使用git克隆源码之后将出现该文件夹，本次演示中使用）

在Windows Powershell中切换到`D:\ygospace\`目录并输入
> git clone https://github.com/Fluorohydride/ygopro.git
> cd ygopro
> git submodule update --init --recursive

**可能问题1**：OpenSSL报错或者GNU报错。一般都是因为本地端口被封了，不支持https协议克隆代码，常见于公司内网。
> fatal: unable to access 'https://github.com/Fluorohydride/ygopro.git/': OpenSSL SSL_read: Connection was reset, errno 10054

**解决方法**：使用ssh协议
> git clone git@github.com:Fluorohydride/ygopro.git

**可能问题2**：下载子模块过程中疑似卡住，命令行迟迟没有输出

**解决方法**：等！在网络不是很好的情况下，下载script模块可能要花很长时间，因为脚本很多。

## 2. 下载依赖

在`D:\ygospace\ygopro\`里创建一个新文件夹`packages`用来保存安装包。

官方教程中提供的依赖：
- [libevent2](https://sourceforge.net/projects/levent/files/libevent/libevent-2.0/libevent-2.0.22-stable.tar.gz)
- [freetype](http://gnuwin32.sourceforge.net/downlinks/freetype-bin-zip.php)
- [irrlicht](http://downloads.sourceforge.net/irrlicht/irrlicht-1.8.1.zip)
- [lua](http://www.lua.org/ftp/lua-5.2.0.tar.gz)
- [sqlite3.source](https://www.sqlite.org/2015/sqlite-amalgamation-3081002.zip)和[sqlite3.dll](https://www.sqlite.org/2015/sqlite-dll-win32-x86-3081002.zip)

下载并解压这些压缩包。

**可能问题**：链接无法下载`lua`

**解决方法**：`http`改成`https`

## 3. 编译event
打开`x86 Native Tools Command Prompt for VS 2019`，之后也请一直使用该命令行。虽然`Developer Command Prompt for VS 2019`默认也是32位编译，但是保险起见，我们还是使用x86命令行。

切换到工作路径并输入
> d:  
> cd ygospace\ygopro\packages\libevent-2.0.22-stable  
> nmake Makefile.nmake  

在`D:\ygospace\ygopro\`里创建一个新文件夹`event`。将编译出的`include`文件夹移动到`D:\ygospace\ygopro\event`，将`WIN32-Code`里的所有文件都移动到`D:\ygospace\ygopro\event\include`中，将`libevent.lib`移动到`D:\ygospace\ygopro\event\lib`并更名为`event.lib`。官方教程中说编译出来的是`event.lib`，但实际上是`libevent.lib`。

**可能问题**：nmake报错

**解决方法**：检查是不是误用了64位编译的命令行

## 4. 编译freetype

官方教程中提供的freetype链接下载的直接就是编译好的库，可以直接拷贝出来。

在`D:\ygospace\ygopro\`里创建一个新文件夹`freetype`。将编译出的`include`和`lib`都移动到`D:\ygospace\ygopro\freetype`即可。

## 5. 编译irrlicht

### 5.1
打开`D:\ygospace\ygopro\packages\irrlicht-1.8.1\source\Irrlicht\Irrlicht10.0.sln`，弹出窗口是否重定向项目至已有平台工具集，点“确定”。如果没有弹出该窗口，或者一不小心点击“取消”。可以右键解决方案里面`irrlicht(Visual Studio 2010)`，在`配置属性->常规`中更改`平台工具集`为Visual Studio 2019(v142)。

  **跳过该步骤会出现问题**：无法找到平台工具集
> MSB8020	无法找到 Visual Studio 2010 的生成工具(平台工具集 =“v100”)。若要使用 v100 生成工具进行生成，请安装 Visual Studio 2010 生成工具。或者，可以升级到当前 Visual Studio 工具，方式是通过选择“项目”菜单或右键单击该解决方案，然后选择“重定解决方案目标”。	Irrlicht	C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Microsoft\VC\v160\Microsoft.CppBuild.targets

### 5.2
在`irrlich\include\IrrCompileConfig.h`中注释掉`#define _IRR_COMPILE_WITH_DIRECT3D_9_`和`#define _IRR_COMPILE_WITH_DIRECTINPUT_JOYSTICK_`。实际上是可以不注释掉的，但是需要安装别的东西会很麻烦。

  **跳过该步骤会出现问题**：“d3dx9shader.h”
>C1083	无法打开包括文件: “d3dx9shader.h”: No such file or directory	Irrlicht	D:\ygospace\ygopro\packages\irrlicht-1.8.1\source\Irrlicht\CD3D9ShaderMaterialRenderer.h

### 5.3
将`配置属性->C/C++->代码生成->运行库`修改为`多线程调试DLL(/MDd)`。这是因为之后ygopro项目是按照`多线程调试DLL(/MDd)`编译的，子模块编译要和父模块一致。

  **跳过该步骤会出现问题**：后面编译ygopro时，MTd和MDd不匹配
>LNK2038	检测到“RuntimeLibrary”的不匹配项: 值“MTd_StaticDebug”不匹配值“MDd_DynamicDebug”(CGUIImageButton.obj 中)	ygopro	D:\ygospace\ygopro\build\Irrlicht.lib(CGUIButton.obj)

### 5.4
在`配置属性->C/C++->预处理器->预处理器定义`中删掉`_ITERATOR_DEBUG_LEVEL=0`。`Debug`模式默认是`_ITERATOR_DEBUG_LEVEL=2`，但是irrlicht项目非要规定为`0`，而ygopro项目是按照`Debug`默认`_ITERATOR_DEBUG_LEVEL=2`编译的，子模块编译要和父模块一致。

后续步骤中也要确保每次在VS 2019中编译时都保持“_ITERATOR_DEBUG_LEVEL”相同，一般`Debug`模式默认是2，`Release`模式默认是0，需要注意有没有特殊设置在`配置属性->C/C++->预处理器->预处理器定义`里。

B战上另外一位up主在介绍如何编译ygopro时[【秋名之巅】《YGOPro》 编译过程详解](https://www.bilibili.com/video/BV1Zt4y1a7ZU)，说有很多步骤都不应该设置`Debug`而是`Release`。但实际上只需要前后保持一致就可以了，可能正是因为irrlicht的默认设置中，明明是`Debug`模式，却按照`Release`模式设置了`_ITERATOR_DEBUG_LEVEL=0`。

**跳过该步骤会出现问题**：后面编译ygopro时，检测到“_ITERATOR_DEBUG_LEVEL”的不匹配项
> LNK2038	检测到“_ITERATOR_DEBUG_LEVEL”的不匹配项: 值“0”不匹配值“2”(CGUIImageButton.obj 中)	ygopro	D:\ygospace\ygopro\build\Irrlicht.lib(CGUIButton.obj)

确认一下`配置属性->常规`中的`配置类型`和`输出目录`，配置类型如果是`.dll`要手动修改为`.lib`，并将`输出目录`中的`bin`修改为`lib`。请注意按照`Debug+Win32`编译，生成错误窗口提示`无法启动程序`是正常的，因为我们编译出来的是`irrlicht.lib`静态库文件，本来也无法单独运行。

在`D:\ygospace\ygopro\`里创建一个新文件夹`irrlicht`。将编译出的`include`移动到`D:\ygospace\ygopro\irrlicht`，并将`lib\Win32-visualstudio\Irrlicht.lib`[^irrlichtwin64]移动到`D:\ygospace\ygopro\irrlicht.lib`。目标lib文件大概在40M左右，如果太小可能就是编译失败了，检查设置，配置类型是否是静态库`.lib`等。官方教程中给的是Win64-visualstudio，似乎是笔误。

## 6. 编译lua

官方教程中提供的lua链接版本是5.2，但是5.2已经不适用了。

**使用lua5.2版本会出现的问题**：lua_KContext等
> C2065	“lua_KContext”: 未声明的标识符	ocgcore	D:\ygospace\ygopro\ocgcore\libgroup.cpp

在`D:\ygospace\ygopro\packages\`里创建一个新文件夹`luaCompile`用来存放工程。新建一个“空项目”，使用C++ for Windows从头开始操作，不提供基础文件。将`D:\ygospace\ygopro\packages\lua-5.3.6\src`文件夹中全部`.h`和`.hpp`后缀的文件都添加到项目的头文件中，将`.c`后缀的文件都添加到项目的源文件中。（技巧是按照类型排序）

在`配置属性->常规`中更改`配置类型`为`静态库(.lib)`。

**跳过该步骤可能会出现问题**：只编译出`.exe`而没有`.lib`文件

请注意按照`Debug+x86`编译，生成错误窗口提示`无法启动程序`是正常的，因为我们编译出来的是`lua.lib`静态库文件，本来也无法单独运行。

在`D:\ygospace\ygopro\`里创建一个新文件夹`lua`。将`D:\ygospace\ygopro\packages\lua-5.3.6\src`文件夹中`lua.h`、`lualib.h`、`lua.hpp`、`luaconf.h`、`lauxlib.h`复制到`D:\ygospace\ygopro\lua\include`中，将编译出来的`Debug\lua.lib`复制到`D:\ygospace\ygopro\lua\lib`中。

按照官方教程，如果使用VS 2019编译需要添加一些额外的设置，例如`配置属性->链接器->命令行`中添加`/FORCE`、`配置属性->C/C++->高级->禁用特定警告`设置为`4996`。但是lua5.3已经不会报`C4996`的错误了，所以不需要加，而生成静态库`.lib`不需要链接器选项，`/FORCE`也就没用了，甚至根本不会有这一设置。

## 7. 编译sqlite3

将`D:\ygospace\ygopro\packages\sqlite-dll-win32-x86-3081002`文件夹中的全部内容拷贝到`D:\ygospace\ygopro\packages\sqlite-amalgamation-3081002`中。打开`x86 Native Tools Command Prompt for VS 2019`，在`D:\ygospace\ygopro\packages\sqlite-amalgamation-3081002`目录下输入
> lib /def:sqlite3.def /out:sqlite3.lib

在`D:\ygospace\ygopro\`里创建一个新文件夹`sqlite3`。将`D:\ygospace\ygopro\packages\sqlite-amalgamation-3081002`文件夹中`sqlite3.h`、`sqlite3ext.h`复制到`D:\ygospace\ygopro\sqlite3\include`中，将编译出来的`sqlite3.lib`复制到`D:\ygospace\ygopro\sqlite3\lib`中。

其实这里如果不创建`include`和`lib`文件夹，直接放在`sqlite3`文件夹下更方便。但是考虑到目录整齐，我们还是保持和其他库一致。

## 8. 使用premake4

按照官方教程需要先删掉`premake4.lua`下的这些行：
>if os.is("windows") then  
include "event"  
include "freetype"  
include "irrlicht"  
include "lua"  
include "sqlite3"  
end  

这是因为这些文件夹下没有`premake4.lua`文件，如果纳入到项目中的话，premake4命令会不知所措。
> cannot open D:/ygospace/ygopro/lua/premake4.lua: No such file or directory

## 9. 编译ygo项目

由于我们使用的是premake4命令，支持的最高vs版本就是2010，所以使用vs2019打开时仍然会弹出平台工具集重定向，点“确定”就好。如果没能做到，解决方法见前文5.1。

按照官方教程需要将如下文件移动到premake4生成的`build`文件夹下：
- script/  
- single/  
- textures/  
- cards.cdb  
- lflist.conf  
- strings.conf  
- system.conf  

实际上直接克隆的源码只需要移动如下文件：
- script/  
- sound/  
- textures/  
- lflist.conf  
- strings.conf  
- system.conf  

按照官方教程需要在`system.conf`中修改`use_d3d`为0，但实际上已经修改好了，不需要动，可以检查一下。

按照官方教程需要在`gframe`文件夹下创建一个空白的`ygopro.rc`文件，实际上是编译出来文件的缩略图。

接下来需要添加大量的库，主要是前面创建的5组`include`和`lib`。但在实际编译过程中，有一些奇怪的路径也需要编译进来。同时，这一步还会遇到奇奇怪怪的问题，所以我们从什么都没改开始，每报一个错，就改动一点解决它。

**报错1**：cspmemvfs报错无法打开包括文件: “sqlite3.h”: No such file or directory

**改动1**：右键cspmemvfs在`配置属性->C/C++->常规->附加包含目录`中添加`..\sqlite3\include`

默认这些项目，包括ocgcore和ygopro似乎都会添加`..\sqlite3`，所以前面说不创建`include`和`lib`可能会更方便一些。

**报错2**：ocgcore找不到lua头文件

**改动2**：右键ocgcore在`配置属性->C/C++->常规->附加包含目录`中添加`..\lua\include`

**报错3**：ygopro找不到freetype/config/fthreader.h

**改动3**：右键ocgcore在`配置属性->C/C++->常规->附加包含目录`中添加`..\freetype\include\freetype2`

**报错4**：“初始化”: 无法从“const T *”转换为“const wchar_t *”

点击报错跳转到问题代码，观察到等式右边返回的是irr::c8指针，不能强转为wchar_t*。注意到如果没有定义宏`_WIN32`，可以使用buffer转过来，那显然我们也按照不是_WIN32的情况来就行了。

**改动4**：注释掉错误的IF分支

目前我还没有尝试使用自己编译的程序联机和人对战，因此注释掉IF分支的方法只能说暂时允许编译通过。宏`_WIN32`表示的是Windows系统，该处代码的强转到底是否会受到Windows上buffer编码的影响有待验证。不过我感觉问题不大，因为只是指针地址的转换。

**报错5**：无法解析的外部符号，lua相关

**改动5**：`ocgcore\interpreter.h`中添加`#include "lua.hpp"`

ygopro是C++语言写的，但是lua是c语言编译的，为了让C++能够调用C函数的库，需要使用extern "C"这种写法，但是`ocgcore\interpreter.h`里没这么搞。当然也可以将`lua.hpp`里的头粘贴到`interpreter.h`里面，效果是等价的。

```
extern "C" {  
    #include "lua.h"  
    #include "lualib.h"  
    #include "lauxlib.h"  
}
```  

**可能问题1**：编译过程中会遇到大量C4828警告

**解决方法**：放置play，不要管她

**可能问题2**：无法解析的外部符号，irrlicht相关

**解决方法**：检查包含库目录下的`irrlicht.lib`，会不会粘贴过来了一个只有60k左右的空壳子文件。

**可能问题3**：在解决前面的一些bug之后重新生成项目，出现一些看不出所以的报错

**解决方法**：清理项目，删掉`bin`和`obj`并重新生成项目可以解决大多数问题

## 10. 运行ygopro项目

编译好的ygopro程序在`bin\debug`文件夹下。

**报错1**：找不到freetype6.dll

**改动1**：搞一个

可以在这里`D:\ygospace\ygopro\packages\freetype-2.3.5-1-bin\bin`找到，复制到`ygopro.exe`相同目录，或者系统目录（请谨慎操作，除非你清楚你在做什么）中。

**报错2**：找不到zlib1.dll

**改动2**：搞一个

可以在这里`D:\ygospace\ygopro\packages\irrlicht-1.8.1\source\Irrlicht\zlib`找到zlib的源文件。但是自己编译太麻烦，建议下载一个。注意下载32位的。
> https://cn.dll-files.com/zlib1.dll.html

**报错3**：程序无法正常启动，0xc00007b

**改动3**：搞一个sqlite3.dll

可以在这里`D:\ygospace\ygopro\packages\sqlite-amalgamation-3081002`找到。

**报错4**：出现`error.log`文件
> Failed to create Irrlicht Engine device!

**改动4**：`ygopro.exe`和前面的`.dll`文件移动到`build`文件夹下

需要`ygopro.exe`和资源放在一起才能正确创建irrlich引擎。

**报错5**：出现`error.log`文件
> Failed to load card database (cards.cdb)!

**改动5**：搞一个cards.cdb
> https://github.com/mycard/ygopro-database

**问题6**：没有卡图、没有声音等其他问题

**改动6**：Github上搞点资源
> https://github.com/

## 11. 总结

自己编译令人头大，所以为什么不直接用别人已经写好的一件安装包呢？：）
> [娱乐力量全开](http://ygocore.ys168.com/)  
> [萌卡平台](https://github.com/mycard/ygopro/releases)  
> [233服](https://ygo233.com/download)  

最后为独立搓出来ygopro的圆神献上最高的敬意！
### Respect!





