---
layout: post
title: "DLC System in UE"
description: ""
date: 2023-1-4
feature_image: images/2023.1.4/0.png
tags: [Unreal]
---

<!--more-->

## Patching

- 在 Project Settings 中开启 ChunkDownloader 相关支持，并将其添加到 PrivateDependencyModuleNames 中
- UE 能够以 .pak 文件的形式交付应用程序主可执行文件之外的资产。为此，需要将资产整理成文件块，即烘焙过程可以识别的资产文件组
- 将内容从内部格式转换为特定于平台的格式的过程称为烘焙，烘焙项目时，UE 将游戏资产分成单独的数据块（chunk），这些数据块可以单独发布，例如以 DLC 和补丁形式发布。数据块是由引擎的资产管理系统识别的带编号资产集，当烘焙项目时，每个数据块都会生成 .pak 文件，然后可以通过内容分发系统进行发布
- 主资产是可以由资产管理器直接操作的资产，而次资产是当主要资产引用它们时自动加载的资产。主资产是在烘焙和数据块划分过程中引用的类型
- 在打补丁的过程中，引擎会将所有后期烘焙内容与最初发布的已烘培内容进行比较，并据此确定补丁中包含的内容。最小的内容块是单个包（如.ulevel或.uasset），因此如果包中有任何更改，那么整个包都将被包含在补丁中。补丁文件具有更高的优先级，因此首先加载其中的任何内容
- 我尝试了打补丁的构建方法，但是生成出来需要将原来的 .pak 都删掉，然后复制所有补丁文件，感觉这和换了个 exe 也没啥太大的区别...
- 使用 UnrealPak.exe 打包 `.\UnrealPak.exe Test_Enc.Pak -Create=C:\Users\didi\Desktop\Unreal\EffectsDemo\Saved\Cooked\Windows\EffectsDemo\Content\DLC\（需要打包的文件地址） -compress -encrypt -encryptindex -aes=88888888888888888888888888888888（32 位密钥）` 查看 .pak 内容 `.\UnrealPak.exe Test.pak -list`
- WidgetReflector 控件反射器

## Pak

- 通过 FPakPlatformFile 来读取 .pak 文件，创建 FPakFile 指针，指向文件平台和 pak 文件路径，并将文件系统挂载到 pak 下，然后就可以读取 pak 下的文件。通过这种方式我可以在编辑器中打开本项目打包的 .pak，但是 build 出来的 exe 不行，以及无法读取其他项目生成的 .pak

```C++
//第一步
    IPlatformFile& InnerPlatform = FPlatformFileManager::Get().GetPlatformFile();
 
    PakPlatformFile = new FPakPlatformFile();
    PakPlatformFile->Initialize(&InnerPlatform, TEXT(""));
 
    FPlatformFileManager::Get().SetPlatformFile(*PakPlatformFile);
 
    const FString PakFileFullName = TEXT("C:\\TestDLCTestProject-Windows.pak");
    FString MountPoint(FPaths::EngineContentDir());
 
    FPakFile* Pak = new FPakFile(&InnerPlatform, *PakFileFullName, false);
    if (Pak->IsValid())
    {
        PakPlatformFile->Mount(*PakFileFullName,1000,*MountPoint);
 
        TArray<FString> Files;
        Pak->FindPrunedFilesAtPath(Files, *(Pak->GetMountPoint()), true, false, true);

    	for (FString& Filename : Files)
    	{
    		if (Filename.EndsWith(TEXT(".umap")))
			{
    			FString NewFileName = Filename;
    			NewFileName.RemoveFromEnd(TEXT(".umap"));
    			int32 Pos_ = NewFileName.Find("/Content/");
    			NewFileName = NewFileName.RightChop(Pos_ + 9);
    			NewFileName = "/Game" + NewFileName;
        		
    			FName mapName = FName(NewFileName);
    			UGameplayStatics::OpenLevel(GetWorld(), mapName);
    			break;
			}
    	}
    }
```

- 我尝试使用了插件 Pak Loader，能够打开 Plugins 下制作的关卡，但是来自 /Engine/ 和 /Game/ 目录下的材质、Mesh 等，有的无法被正确读取，同时我还很担心 Plugin 下的 GameMode 不能被正确加载应该怎么办，以及该插件没有测试非 Windows 平台，我担心其他平台无法正确打包与解析
- 最终还是用了做 DLC 的方案，需要在 Project Launcher 中配置 MainGame 和 DLC，DLC 的名字要和 Plugin 的名字相同，然后 DLC 会生成一个 .pak，将它复制到 MainGame 的 Paks 文件夹下就能打开了，UE 应该是进行了自动挂载，需要记得在 .build.cs 文件中 Include 上该插件，以及如果已经 Build 的 exe 加上新加的插件是无法被正确挂载的，但在旧的插件下放新的地图是可以的，但还是会出现 /Engine/ 材质无法加载的问题
- 在生成 pak 时，需要取消勾选 Project Settings 下的 Share Material，因为这种机制压缩了材质，会让挂载的 pak 里一些纹理找不到正确的位置，Use Io Store 不知道有啥用，但是最好也取消勾选，这样 DLC 的材质就能正确加载啦，但我不知道为什么 Pak Loader 不行，道理来说应该是一样的
- 可以看到我这个项目下 Mount point 是 ../../../，引擎下的资源是 Engine/Content/，项目资源是 ProjectName/Content/，插件的资源是 ProjectName/Plugins/PluginName/Content/

![](../images/2023.1.4/0.png)

![](../images/2023.1.4/1.png)

- 我之前测试 PakLoader 打包的资源文件结构如下，是一致的，也就是说问题不在 pak 这里，也对，毕竟都是用 Project Launcher 打包的，应该还是引擎本身对 pak 的加载方式有特殊的处理，直接读 pak 导致一些文件索引目录不一致

![](../images/2023.1.4/2.png)

![](../images/2023.1.4/3.png)

- 这倒是提醒我了，我没有用同一个项目打包加载测试 Pak Loader，也许我在同一个项目下 Pak Loader 是可行的？但感觉这样的话和用引擎直接做 DLC 也没有区别了，需要在同一个项目下提前设好 Plugin 才能正确加载，暂时没空细看了，有空了可以去看看源码，在 LaunchEngineLoop.cpp
- 