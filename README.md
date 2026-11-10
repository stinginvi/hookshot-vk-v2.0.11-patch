# Hookshot v2.0.11 Vulkan 渲染器改动

这是 [Hookshot](https://github.com/NotRexed/Hookshot) v2.0.11 的修改版，只包含 `tools/gpu/vk` 里改过的 7 个源文件。协议是 GNU GPL-3.0 或更高版本，见 `LICENSE`。

这里没有游戏。不要把 `hookshot.exe`、`romfs`、`exefs`、密钥、`image.bin`、着色器缓存、管线缓存或存档放进这个目录。每个人用自己的游戏，在本机的 Hookshot Builder v2.0.11 里编译。

上游现在的主分支比 v2.0.11 新，接口已经对不上。这些文件只覆盖 v2.0.11 构建器里的同名文件，不要合并进当前主分支。

## 改了什么

- 启动时如果着色器戳记没变，仍会把管线状态列表记进管线笔记，再预编译进内存。预测只在需要时跑一次。
- 图形程序或管线还没编译好时，这一次绘制跳过，重放线程不再等它。`RX_VK_PIPE_SKIP=0` 恢复成等待。
- 读图（一帧绘制少于 2000）时，后台编译线程用普通优先级；正常游戏（不少于 2500 次绘制）时降回低于普通。`RX_VK_NO_WORKER_BOOST=1` 保持低于普通。
- 自上次统计以来，经 PCIe 读取的范围超过约 1 GB 的那一块内存池会放进显存镜像。749 MB 的总预算对其余池子不变；腾不出闲置镜像时，只有这一块可以超出预算。`RX_VK_NO_MIRROR_PROMOTE=1` 关闭这项。池子地址每台机器不同。

## 怎么编译

对方需要自己有游戏，并且已经用 Hookshot Builder v2.0.11 生成过。把 `gpu/vk` 下的 7 个文件覆盖到：

`%LOCALAPPDATA%\HookshotBuilder\versions\v2.0.11\HookshotBuilder\tools\gpu\vk\`

游戏不要开着。在 PowerShell 里：

```powershell
$tools = "$env:LOCALAPPDATA\HookshotBuilder\versions\v2.0.11\HookshotBuilder\tools"
$env:HOOKSHOT_MINGW = "$tools\llvm-mingw"
$env:RX_CPM = "$tools\cpm"
$env:RX_VK_ONLY = "1"
$env:RX_RELEASE = "1"
$env:PATH = "$tools\llvm-mingw\bin;" + $env:PATH
Set-Location $tools
& "$tools\python\python.exe" -I nx2/build.py --standalone --jobs 4
Copy-Item -Force "$tools\work\sa\build\hookshot_vk.dll" "你的游戏目录\bin\hookshot_vk.dll"
```

编出来的 DLL 只匹配 v2.0.11 的 `hookshot.exe`。启动器的 Update 会用官方源码重编并盖掉这个 DLL，之后需要再覆盖源文件、重新编译。
