# FramePackLoop

**FramePackLoop** 是基于 lllyasviel 的 **lllyasviel/FramePack**，并针对循环动画进行了专门功能扩展的软件。

此功能为基于 FramePack 基本功能应用的实验性实现，请事先理解未来可能无法使用。

## 背景

循环视频在视频的待机画面、背景、图标、贴图等多种场景中被广泛使用。  
FramePack 默认附带了一个可通过单张图片生成视频的工具，但缺少使动画首尾自然衔接的功能，并不适合制作循环视频。

此外，虽然已有能够设置关键帧的软件出现，但仅仅在首尾设置相同的图像，也存在无法平滑循环的问题。

**FramePackLoop** 正是为了验证能否利用 FramePack 原本的实现，将首尾平滑地连接起来而开发的。

## FramePackLoop 的基础知识

在 FramePackLoop 中，循环视频通过以下步骤生成：

1. 生成主视频
2. 生成连接主视频首尾的衔接视频
3. 将主视频与衔接视频合并成一个单循环视频
4. 重复该单循环视频，生成最终循环视频

另外，FramePackLoop 中视频长度不是用“秒”，而是以“Section（分段）”为单位管理。  
1 Section 约相当于 1 秒，但严格来说并非正好 1 秒。

## 参数说明

在 FramePackLoop 中，相对原版 FramePack，追加或更改了以下参数。

![FramePackLoopParameter](images/image.jpg)

- **Main Video Length**  
  主视频的分段数。

- **Connection Video Length**  
  衔接视频的分段数。理想情况是与主视频分段数相同，但即使较小也没有问题。

> 总视频长度为  
> `Main Video Length + Connection Video Length`  
> 例：Main Video Length=1，Connection Video Length=1 → 约 2 秒

- **Padding Video Length**  
  - 设为 `0` 时，会直接从输入图像开始生成循环视频。
  - 设为非 `0` 时，会先播放一段动画后进入循环。  
  若 Padding Video Length Checker 的值不变，则不会产生变更效果。
  - 若不自然或使用设定图片时，建议设为 1 以上（大致 3 以内为宜）。

- **Loop Num**  
  输出的循环次数。  
  由于是通过复制单循环视频的方式，即使设置较大的数值，处理时间也不会有太多增加。

### 面向希望反复生成视频用户的参数
为希望在夜间等时段反复生成视频的用户，准备了可重复生成视频的参数。

![FramePackLoopParameter](images/image3.jpg)

- **Generation Count**  
设置视频生成次数。将按指定次数生成视频。  
**当 Generation Count 为 2 或以上时，Seed 会使用随机值。**  
当前生成次数会显示在预览下方的 “Generation Index”（计数从 1 开始）。

- **Progress Option**  
过程预览及文件保存方式。  
  - **All Progress File Output**：输出所有过程文件。  
  - **Reduce Progress File Output**：过程文件会以相同文件名覆盖保存，减少输出文件数量。  
  会在 output 文件夹中生成 system_preview.mp4，供预览使用。  
  ※生成过程中请勿打开此文件。  
  - **Without Preview**：不输出过程预览。输入图像及过程文件也不会保存，仅输出最终结果。因此最终输出的生成速度会略微提升。  
  - **"Without VAE Decode"**：（面向高级用户）将 latent 转化为图像并输出 Latent 文件。  
  由于不进行 VAE 解码，每轮生成时间更短。
  - **Decode Latent File**：（面向高级用户）将指定的 Latent 文件转换为视频。  

- **Seed**  
视频生成所使用的种子值。  
此值仅在 Generation Count 为 1 时有效。  
生成视频时所使用的 Seed 会记录在输出的视频文件名中。

### Without VAE Decode 模式及 Decode Latent File 模式的使用场景
1. 找种子  
FramePack 中，动作内容在很大程度上依赖于所使用的 Seed。因此，找到能实现理想动作的种子有时非常重要。  
对于较短的视频，通过查看 Latent 图像，可以一定程度上推测各个 Seed 会生成怎样的动作。  

![FramePackLoopParameter](images/image9.png)

使用 “Without VAE Decode” 模式，将 Step 数设为 10 左右，生成大量 Latent 图像，可高效找到可能产生理想动作的种子。
然后，固定找到的 Seed，将 Step 数设为 25 重新生成视频，就更容易获得接近目标的影像。

2. 大批量生成
在 FramePack 中，生成包含不理想动作的视频并不少见。对这些视频进行 VAE 解码的时间有时会白白浪费。  

因此，在进行大批量生成时，可先仅生成大量的 Latent 图像和 Latent 文件，之后只挑选出可能包含理想动作的 Latent 图像进行 VAE 解码并转为视频，这样就有更高几率高效地获得理想视频。  

这种情况下，使用 “Without VAE Decode” 模式，将 Step 数设为 25 进行视频生成。

### Latent 文件的解码方法
1. 使用以下界面上传 Latent 文件。  
  ![FramePackLoopParameter](images/image10.jpg)

2. 在 Seed 中设定生成该 Latent 文件时所使用的 Seed。
3. Generation Count 设为 1。
4. 勾选 Decode Latent File。
5. 点击 Start Generation 开始解码。

解码 Latent 文件时，Step 等参数基本需要与生成该 Latent 文件时的设置值保持一致。  
解码后输出的视频文件名，其索引会与 Latent 文件相似。

## 输出文件

`output` 文件夹下会输出以下文件（其他文件删除也没有问题）。

- `XXXX_{Seed}_1loop.mp4`  
  - 单循环视频。可在视频编辑软件或直播软件中循环播放使用。
  - ※根据使用的编辑软件不同，可能会因丢帧而无法正常循环（CapCut 和 OBS Studio 已确认正常动作）。

- `XXXX_{Seed}_loop_{Loop Num}.mp4`  
  - 将单循环视频连接起来的长时长视频。

- `XXXX_{Seed}_latent.png`  
  - latent 文件转换而成的图像。
- `XXXX_{Seed}_latent.pt`  
  - latent 文件。

## 安装与运行方法

安装方法与原版 FramePack 相同。  
搭建好环境后，执行以下命令。

```bash
python demo_gradio_loop.py
```

已安装 FramePack 的用户，只需将 demo_gradio_loop.py 放到原版 demo_gradio.py 所在的文件夹，执行上述命令即可启动。  

已在 FramePack 的 2025/04/28 的 main 分支（提交编号 6da55e8）确认动作。  
如需复制使用，请更新至这一版本。

### 有限的 Windows 包支持

虽为有限支持，我们为使用 FramePack Windows 包的用户准备了可简单添加 FramePackLoop 的附加包。  
※前提是已安装 FramePack 的 Windows 版。

此为依赖于原始实现的有限支持，请理解未来可能不再提供支持。  
此外，我们尽量做到不影响原始包，但万一出现问题，还请见谅。

[>>> 点击此处下载附加包 <<<](https://github.com/red-polo/FramePackLoop/releases/download/windows-v1.6/run_loop.zip)

将下载文件中的 run_loop 文件夹按图示放置到 FramePack Windows 版的安装文件夹中。

![FramePackLoopParameter](images/image4.jpg)

然后运行 run_loop 文件夹内的 run_loop.bat，即可启动 FramePackLoop。  
输出文件会保存在 run_loop 文件夹下的 output 文件夹中。

![FramePackLoopParameter](images/image8.jpg)

卸载时，请删除 run_loop 文件夹即可。

## LoRA 的使用方法
FramePackLoop 中可以使用 LoRA。  
请在 **启动 FramePackLoop 之前** 编辑文件夹内的 `lora_setting.json` 文件。

![FramePackLoopParameter](images/image20.jpg)

填写内容请参考 `lora_setting_sample.json`。例如如下所示，填入 LoRA 文件路径及应用强度。

```json
[
    {
        "file":"./lora_models/lora1.safetensors",
        "scale":1.0
    }
]
```

设置后启动 FramePackLoop，控制台会显示 LoRA 的加载情况。请确认 LoRA 是否被正确加载。

![FramePackLoopParameter](images/image21.jpg)

注意：LoRA 无法在运行中动态更改。如需更改 LoRA，请重启应用程序。

使用由 [Musubi Tuner](https://github.com/kohya-ss/musubi-tuner) 创建的 FramePack 专用 LoRA 最为有效，  
但针对 HunyuanVideo 的 LoRA 也有部分案例显示有效。

## 系统选项
### 以视频形式展示 Latent
通过将 system_setting.json 中的 preview.type 设为 "video"，可将 next latent 的显示方式改为视频。
会在 output 文件夹中生成一个名为 "preview.mp4" 的文件，生成过程中请勿打开。打开可能导致生成出错。
将 Next Latent 显示从图像变为视频。  
```
{
    "preview":{
        "type":"video",
        "height":200
    } 
}
```

## 少量技术解说

<span style="font-size: 200%; color: red;">重要通知（2025/05/02）</span>

经过进一步验证，遗憾地得出结论：过去的信息几乎不会被反映。  
对过去动作信息能够被反映抱有期待的用户，非常抱歉。

追加验证的结果，我们判断即使参考过去时间帧也几乎无效。  
除参考过去帧外，我们也做了若干努力使视频在视觉上自然连接，因此生成的视频在**视觉上**是漂亮的循环，  
但**动作**能否平滑循环，基本取决于运气。

我本人也进一步学习 FramePack，探讨了能否参考过去信息，  
但由于能否参考过去信息很大程度上依赖于模型，短期内似乎难以实现。

另一方面，在我本人使用本工具以及众多用户使用过程中，确实感受到即使是“碰运气”，也有可能生成完成度较高的循环素材。  
也曾考虑将工具设为非公开，但因有用户喜爱并使用，今后将继续公开，请大家理解其中存在运气成分并加以使用。

~~本软件聚焦于“衔接视频”的生成。  
FramePack 在按分段生成视频时，会参考输入图像及“未来侧的下一分段”信息。~~

~~**FramePackLoop** 在生成衔接视频时，不仅参考“未来”，还扩展为可参考“过去侧的上一分段”信息。（※未验证）~~

~~这里以循环视频为前提，过去与未来如下：~~

~~- **未来**：主视频的开头附近~~
~~- **过去**：主视频的结尾附近~~

~~FramePack 原本就有赋予未来信息的机制，  
尝试能否也赋予过去信息后发现，**获得了良好结果**，于是整合成了循环视频制作软件。  
（※但未对效果进行正式验证）~~

此外，原 FramePack 实现中的“分段视频重叠”也对视频的平滑连接有效。

# FramePackConnect（beta）

**FramePackConnect** 是基于 lllyasviel 的 **lllyasviel/FramePack**，提供在视频间进行平滑补间功能的软件。

视频间的补间不仅在制作长视频时有用，在游戏开发中为循环素材进行平滑衔接等场景也十分有用。

此功能为基于 FramePack 基本功能应用的实验性实现，请事先理解未来可能无法使用。

## FramePackConnect 的简单用法
![FramePackConnectUI](images/image6.jpg)

准备两个用 FramePack 制作的、想要连接的视频。  
视频需满足以下条件：
* 1 秒以上，建议 3 秒以上。
* 由 FramePack 制作、画面尺寸相同的视频。

将开头的视频上传到左侧 HeadVideo。  
将末尾的视频上传到右侧 TailVideo。

输入连接部分的 Prompt，点击 StartGeneration 按钮即开始生成。  
output 文件夹中会输出连接后的视频文件。

※原视频大部分会被保留，但在连接过程中会有少许变化。

## 参数说明

FramePackConnect 相比 FramePack，追加或更改了以下参数。

![FramePackConnectParameter](images/image7.jpg)

- **Connect Video Length**  
  为补间而生成的视频分段数。

### 面向希望反复生成视频用户的参数
与 FramePackLoop 相同。

## 输出文件

`output` 文件夹下会输出以下文件。

- `XXXX_{Seed}_connect.mp4`  
  - 两个视频补间后的视频。

## 安装与运行方法

安装方法与原版 FramePack 相同。  
搭建好环境后，执行以下命令。

```bash
python demo_gradio_connect.py
```

已安装 FramePack 的用户，只需将 demo_gradio_connect.py 放到原版 demo_gradio.py 所在文件夹，执行上述命令即可启动。  

已在 FramePack 的 2025/04/28 的 main 分支（提交编号 6da55e8）确认动作。  
如需复制使用，请更新至这一版本。

## 限制事项
・当尾部视频为 3 秒以下时，视频的最后几帧可能会被截掉。  
・不显示处理过程文件及预览。

### 有限的 Windows 包支持

虽为有限支持，我们为使用 FramePack Windows 包的用户准备了可简单添加 FramePackConnect 的附加包。  
※前提是已安装 FramePack 的 Windows 版。

此为依赖于原始实现的有限支持，请理解未来可能不再提供支持。  
此外，我们尽量做到不影响原始包，但万一出现问题，还请见谅。

[>>> 点击此处下载附加包 <<<](https://github.com/red-polo/FramePackLoop/releases/download/windows-v1.6/run_loop.zip)

将下载文件中的 run_loop 文件夹按图示放置到 FramePack Windows 版的安装文件夹中。

![FramePackLoopParameter](images/image4.jpg)

然后运行 run_loop 文件夹内的 run_connect.bat，即可启动 FramePackConnect。  
输出文件会保存在 run_loop 文件夹下的 output 文件夹中。

![FramePackLoopParameter](images/image8.jpg)

卸载时，请删除 run_loop 文件夹即可。

# FramePackVideo2LoopVideo (beta)

**FramePackVideo2LoopVideo** 是一款输入由 lllyasviel 的 **lllyasviel/FramePack** 制作的视频，并生成循环化视频的软件。

随着 FramePack-F1 的出现，催生了对从视频生成循环视频的需求，因此开发了此工具。

本软件正在开发中，规格可能变更。

此外，此功能为基于 FramePack 基本功能应用的实验性实现，请事先理解未来可能无法使用。

## FramePackVideo2LoopVideo 的简单用法
![FramePackConnectUI](images/image11.jpg)

准备一个用 FramePack 制作的、想要循环化的视频。  
视频需满足以下条件：
* 1 秒以上，建议 3 秒以上。
* 由 FramePack 制作的视频。

将视频上传到 InputVideo。

输入 Prompt，点击 StartGeneration 按钮，即开始生成循环视频。  
output 文件夹中会输出连接后的视频文件。

※注意事项  
- 原视频大部分会被保留，但在连接过程中会有少许变化。
- 原视频最后几帧可能会消失。
- 若原视频的第一帧与最后一帧差异不大，可能很难顺利做成循环。

## 参数说明

FramePackLoop 相比原版 FramePack，追加或更改了以下参数。

![FramePackLoopParameter](images/image12.jpg)

- **Connection Video Length**  
  衔接视频的分段数。大致 1 分段为 1 秒。

- **Loop Num**  
  输出的循环次数。  
  由于是通过复制单循环视频的方式，即使设置较大的数值，处理时间也不会有太多增加。

### 面向希望反复生成视频用户的参数
与 FramePackLoop 相同。

## 输出文件

`output` 文件夹下会输出以下文件（其他文件删除也没有问题）。

- `XXXX_{Seed}_1loop.mp4`  
  - 单循环视频。可在视频编辑软件或直播软件中循环播放使用。
  - ※根据使用的编辑软件不同，可能会因丢帧而无法正常循环（CapCut 和 OBS Studio 已确认正常动作）。

- `XXXX_{Seed}_loop_{Loop Num}.mp4`  
  - 将单循环视频连接起来的长时长视频。

## 安装与运行方法

安装方法与原版 FramePack 相同。  
搭建好环境后，执行以下命令。

```bash
python demo_gradio_video2loop.py
```

已安装 FramePack 的用户，只需将 demo_gradio_video2loop.py 放到原版 demo_gradio.py 所在的文件夹，执行上述命令即可启动。  

已在 FramePack 的 2025/04/28 的 main 分支（提交编号 6da55e8）确认动作。  
如需复制使用，请更新至这一版本。

### 有限的 Windows 包支持

虽为有限支持，我们为使用 FramePack Windows 包的用户准备了可简单添加 FramePackVideo2LoopVideo 的附加包。  
※前提是已安装 FramePack 的 Windows 版。

此为依赖于原始实现的有限支持，请理解未来可能不再提供支持。  
此外，我们尽量做到不影响原始包，但万一出现问题，还请见谅。

[>>> 点击此处下载附加包 <<<](https://github.com/red-polo/FramePackLoop/releases/download/windows-v1.6/run_loop.zip)

将下载文件中的 run_loop 文件夹按图示放置到 FramePack Windows 版的安装文件夹中。

![FramePackLoopParameter](images/image4.jpg)

然后运行 run_loop 文件夹内的 run_video2loop.bat，即可启动 FramePackVideo2LoopVideo。  
输出文件会保存在 run_loop 文件夹下的 output 文件夹中。

![FramePackLoopParameter](images/image8.jpg)

卸载时，请删除 run_loop 文件夹即可。

# 未来计划

未定。

# 致谢

感谢 lllyasviel！  
感谢 kohya-ss！