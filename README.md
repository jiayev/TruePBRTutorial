# Jiaye的社区着色器TruePBR贴图使用和制作入门教程

新版本的Community Shaders（社区着色器）已经发布了，其中包含了许多重量级的新功能。其中，最为重量级的可能就是TruePBR。TruePBR能够彻底改变老滚5陈旧的材质系统，让它跨过现代游戏材质渲染的门槛：PBR，也就是基于物理的渲染。

如果你不知道什么是PBR，下面我会简单介绍一下。你也可以直接跳过介绍部分。

## 什么是PBR？它与原版材质系统有什么不同？

PBR是基于物理的渲染（Physically Based Rendering）的缩写。它是一种渲染技术，能够更加真实地模拟物体的光照、反射、折射等物理特性。PBR的出现，让游戏材质的制作更加直观，同时也让游戏画面更加真实。

PBR有多种实现方式，但最常见，也是我们在TruePBR中使用的实现方式是基于金属度和粗糙度的PBR。社区着色器中的TruePBR采用了与UE4相似的实现方式，一般来说，对于一个材质，我们需要如下几张贴图：

- Albedo（反照率）：描述物体的基础颜色，与原版材质系统中的Diffuse贴图类似，但不因附加任何光照信息。（也就是说，不能有阴影、高光等信息）
- Normal（法线）：描述物体的凹凸细节，与原版材质系统中的法线贴图基本无区别。
- Metallic（金属度）：一张灰度信息图，描述物体的金属度，黑表示非金属，白表示金属。
- Roughness（粗糙度）：一张灰度信息图，描述物体的表面粗糙度，0表示完全光滑，1表示完全粗糙。
- AO（环境光遮蔽）：描述物体表面的环境光遮蔽信息，可以用来增强物体的立体感。
- Specular（高光）：与原版系统的高光贴图并不一样，这里的高光贴图描述的是物体的反照率。对于绝大多数材质，这张贴图可以不用（使用默认的白色）。

此外，TruePBR还支持Glow（发光）贴图，次表面散射贴图、视差贴图等。简单来说，它比现在流行的CPM复杂视差材质更加强大。

## 如何使用TruePBR？

以下的文章分为两个部分：玩家和模组作者。

### 玩家：

前提条件：安装了新版本的Community Shaders。
目前，Nexus版本还不支持PBR材质。你可以在Discord获取最新测试版，或者使用我提供的版本。

注意事项：不可使用ENB。
避免使用Auto Parallax。如果你要使用PBR地形，那就不要使用Terrain Parallax Blending Fix。

必须工具：ParallaxGen https://www.nexusmods.com/skyrimspecialedition/mods/120946

简单来说，你需要先像安装任何其他模组一样安装PBR材质包。PBR材质不会与你安装的任何普通材质包产生冲突，所以你无需担心任何问题。不过，这样并不能直接在游戏中看到效果。安装PBR材质并启用后，你需要使用ParallaxGen来将你所有的模型修改为PBR材质。

首先，将ParallaxGen安装到一个独立的位置，不要作为Mod添加到管理器里。

这里我们使用Mod Organizer 2来演示。首先，打开MO2，然后点击更改可执行程序。

点击添加，然后输入名称，在程序中选择ParallaxGen.exe所在的路径。

然后，你就可以在MO2中看到ParallaxGen了。点击运行。新版本中，ParallaxGen启动时，会显示选项界面。确保里面的游戏路径、Mod管理器类型和路径没有错，确保右上角选中TruePBR，然后开始Patching。过一会可能会弹出窗口，要求你确定PBR材质的覆盖顺序，一般直接确定不会有问题。整个过程一般需要几分钟，取决于你的模组数量和配置。注意，你每次安装新的PBR材质包后都需要运行一次ParallaxGen，所以建议是在安装完所有你想要PBR材质包后再运行。（当然，每次安装新的都跑一遍也没问题）

ParallaxGen会在它的目录下生成一个ParallaxGen_Output.zip文件，你需要将这个zip文件作为Mod安装到MO2中。让这个Mod覆盖一切（放到最底下），然后启用它。这样，你就可以在游戏中看到PBR材质了。

每次跑ParallaxGen时，你都需要禁用或删除之前的ParallaxGen_Output模组。

需要注意的是，如果你安装了PBR材质，就必须在Community Shaders开启的情况下启动游戏，不然你只会看到一片白。如果你不想使用PBR材质了，在MO2中取消选择PBR材质包，并且取消选择ParallaxGen_Output即可。

我们将很快在Nexus上看到更多PBR材质包。

### 模组作者：

本部分也分为两部分：将原版材质转换为PBR材质和制作新的PBR材质。注意，本教程并非贴图制作教程，而是如何将贴图应用到游戏中的教程。不论是哪一种，我们最终需要的文件都是一样的。本教程默认你懂英语，并且能够科学上网。

#### 将原版材质转换为PBR材质：

原版材质主要有几个部分：Diffuse贴图（一般后缀为_d或没有后缀），法线贴图（一般后缀为_n），高光贴图（一般后缀为_s），环境遮罩贴图（一般后缀为_m或_em），发光贴图（_g），次表面散射贴图（_sk）。由于原版材质缺乏规范性，实际的文件名可能会有所不同。此外，很多mod还可能有视差贴图（_p）。

而对于TruePBR材质，我们需要以下文件：

- albedo贴图（无后缀）
- normal贴图（_n）
- rmaos贴图（_rmaos）（rmaos是Roughness、Metallic、AO、Specular的缩写）

这三个贴图是必须的，其他的贴图是可选的。

可选：

- 视差贴图（_p）
- 发光贴图（_g）
- 次表面散射贴图（_s）...

详细的贴图类型，可以参考TruePBR的文档。https://github.com/doodlum/skyrim-community-shaders/wiki/True-PBR

开始处理前，我假设你已经对贴图制作和修改有一定了解，并且会使用一定的制作工具，如PS、Substance Painter等。这里我还推荐使用Materialize，它是一个免费的贴图制作工具，可以快速生成PBR贴图；以及Quixel Mixer，它就像Substance Painter一样，但是免费，还有大量来自Megascans的贴图资源。

还推荐一个非常好用的工具：Chainner。它是一个节点式的工具，可以用于批量处理贴图，比如调整亮度、对比度等，还可以接入一些AI模型用于超采样等。我会附上两个我做好的chn文件，用于转换原版贴图到png、将处理好的png转换为dds。

那么，我们需要对原版材质进行的处理，我分为几个步骤：

1. 出于某种原因（杯赛的奇妙代码），原版Diffuse贴图都是被调暗的。为了将它们还原为真实的亮度，在这里需要使用一个Exist大佬从杯赛代码里翻出的公式：`diffuse = pow(LinearToGamma(diffuse) / PI, 1 / 1.5)`
   
    这个公式进行逆运算，就是对Diffuse贴图进行Gamma 1.5校正（也就是1.5次方），然后乘以π。这样，我们就得到了原始的Diffuse贴图。这个公式可能并不适用于所有的贴图（显然，mod作者大多数不知道b社的奇妙公式），但是对于大多数原版贴图来说，效果还是不错的。你也可以手动把它们调亮、调节对比度等。

2. 这样得到的Diffuse贴图还包含大量的阴影，高光等信息。不同人有不同的处理方法，反正我们的目标是得到一个不包含光影信息的Albedo贴图，只记载材质的本色。你可以使用PS的HDR滤镜、Highpass滤镜、高光阴影调整等等，或者从头画一个新的。也有很多不同的Delight技术。

3. 对于法线贴图，一般来说，原版的法线贴图是可以直接使用的。同时，许多原版材质使用法线贴图的Alpha通道（也就是透明度）来存储高光信息，你可以将其单独导出来，反色，作为一个初步的Roughness贴图。

4. 粗糙度贴图是一个比较重要的贴图，它描述了物体表面的粗糙度。如果前面的步骤中你已经得到了一个初步的粗糙度贴图，那么你可以直接使用。或者在其基础上调整明暗，来接近你想要的效果。如果没有，你可以使用Materialize等工具来生成一个粗糙度贴图。Github上还有一个PBRify项目，里面可以下载到使用Diffuse贴图生成粗糙度贴图的AI模型（可以在Chainner中使用）。

5. 对于Metallic贴图，一般来说，原版材质的环境遮罩贴图可以直接使用。（但最好将其对比度调的很高，确保金属部分尽量为全白，非金属为全黑）你可以将其调整亮度、对比度等，来得到一个合适的Metallic贴图。如果没有，你可以手动绘制，（这很简单，只要把金属部分涂白就行），或者使用Materialize等工具来生成一个金属度贴图。

6. AO贴图一般可以通过Materialize等工具生成，或者在Substance Painter中烘焙得到。

7. 对于大多数材质来说，你可以使用一个全白的Specular贴图。只有部分材质需要注意，比如水、雪这种流体的反照率可能低于正常的材质，而钻石等宝石的反照率可能会很高。你可以在 https://physicallybased.info/ 查看一些常见材质的反照率。

8. 对于视差贴图，你可以使用Materialize等工具生成。视差贴图只要求是一张灰度图。之前的视差贴图可以直接使用。

9. 其它贴图将在之后统一介绍。绝大多数材质只需要上面的几张贴图就可以了。

#### 制作新的PBR材质：

对于本身就使用PBR工作流的模组作者来说，这个过程会简单很多。你只需要将你的贴图导出为对应的格式即可。你需要导出的贴图有：

- Albedo
- Normal（注意，需要导出为DirectX格式）
- 粗糙度
- 金属度
- AO
- Specular（可选）
- 视差、高度贴图（可选）

TruePBR使用这些贴图的方式，请参考UE4的PBR。

#### 保存贴图并放入Mod：

现在，我们已经得到了所有的贴图，我们需要将它们保存为DDS格式。以下是格式要求：

- Albedo：保存为sRGB色彩空间，如果有透明度则保存为BC7，否则保存为BC1。无后缀。
- Normal：保存为BC7，线性色彩空间。_n后缀。
- RMAOS：RMAOS的红绿蓝通道分别是粗糙度、金属度、AO贴图，Alpha通道则为Specular贴图。保存为BC1，线性色彩空间。（如果你使用了Specular贴图，那么保存为BC7）_rmaos后缀。
- 视差贴图：保存为BC4灰度格式。_p后缀。
- 任何有色彩信息的贴图，如发光贴图、次表面散射贴图等，保存为sRGB色彩空间。同样，有透明度则保存为BC7，否则保存为BC1。

一般来说，我们推荐将RMAOS和视差贴图的分辨率减半（对于部分贴图，甚至可以四分之一），以减少显存占用。当然，这就自己做取舍了。

你可以使用Nvidia的Texture Tools来保存贴图，也可以使用Chainner。

之后，我们需要把它们放入对应的路径。与原版材质不同的是，需要放入\textures\pbr\。比如，原版材质的路径是\textures\armor\iron\ironarmor.dds，那么PBR材质的路径就是\textures\pbr\armor\iron\ironarmor.dds。

最后，为了让ParallaxGen能够识别这些贴图，我们需要在\PBRNifPatcher文件夹中放好一个json文件（文件名随意）。

示例：

```json
[
    {
        "texture": "armor\\iron\\ironarmor", 
        "emissive": false, 
        "parallax": true, 
        "subsurface": false,  
        "specular_level" : 0.04, 
        "subsurface_color": [1,1,1], 
        "roughness_scale" : 1, 
        "subsurface_opacity" : 1, 
        "displacement_scale" : 0.35
    },
    ...
]
```

这个json文件的作用是告诉ParallaxGen如何处理这些贴图。texture是匹配的Diffuse贴图的路径。默认状态下，ParallaxGen会认为对应的法线贴图是ironarmor_n.dds，RMAOS贴图是ironarmor_rmaos.dds，视差贴图是ironarmor_p.dds等等。下面的值则是一些参数，比如是否发光、是否有次表面散射、高光强度、次表面散射颜色、粗糙度缩放、次表面散射透明度、视差强度等等。

需要注意的是，specular_level是一个很重要的参数，真实的Specular值是这个参数乘Specular贴图的值。当你的Specular贴图是全白的时候，这个参数就是最终的Specular值，即0.04，这也是绝大多数材质的Specular值。对于雪、水，最终的Specular值约为0.02，而钻石等宝石的Specular值可能会很高。在PhysicallyBased.info上可以查到一些常见材质的Specular值，乘以0.08即可得到这个参数。

此外，roughness_scale是粗糙度的缩放，subsurface_color是次表面散射的颜色，subsurface_opacity是次表面散射的透明度，这三个值我们一般不需要修改（修改贴图本身即可）。而displacement_scale是视差强度，一般来说，0.35是一个不错的值。当然，这需要根据你的视差贴图来调整。

如果启用多层视差贴图等功能，你需要在json文件中添加更多的条目。具体参考TruePBR的文档。

请注意，ParallaxGen并不会真的自动识别你有哪些贴图，而是根据json文件中的路径来识别。
如果你的贴图文件名与默认的不同，你可以在json文件中指定。比如，如果你的法线贴图是ironarmor_normal.dds，那么你可以在json文件中指定`"slot2": "textures\\pbr\\armor\\iron\\ironarmor_normal.dds"`。这里的槽位对应Nif文件中的槽位。

槽位对应关系如下：

- 1：Albedo/Diffuse
- 2：Normal
- 3：发光贴图
- 4：Parallax
- 5：不使用
- 6：RMAOS
- 7：多层视差贴图、涂层贴图的法线和粗糙度（csr）
- 8：次表面散射贴图（_s）（或涂层贴图的颜色）

参考：https://docs.google.com/spreadsheets/d/1WJaqIXpk44ISA2bFxi1a054JQD26tFqF-thDC7mHesc/

现在，你的Mod文件夹应该是这样的：

```
textures
    pbr
        armor
            iron
                ironarmor.dds
                ironarmor_n.dds
                ironarmor_rmaos.dds
                ironarmor_p.dds
PBRNifPatcher
    iron.json
```

把它安装到MO2中，启用，然后运行ParallaxGen，你就可以在游戏中看到你的PBR材质了。ParallaxGen会搜寻所有使用你json文件中指定的贴图路径的nif，将它们对应的贴图路径修改为PBR材质的路径，并且给对应的nif文件添加PBR flag。

#### Landscape（地形）的PBR

Landscape部分稍有特殊。但也并不复杂。
除了像普通的贴图一样放进文件夹以外，你还需要做两件事：
- 使用xEdit，修改地形贴图对应的Texture Set，把贴图路径指向你的PBR贴图。
- 在Mod文件夹里，新建PBRTextureSets文件夹。并且每个你修改了的Texture Set，对应建立一个同名的Json文件。比如，Texture Set的EDID为LandscapeDirt01，则文件名为LandscapeDirt01.json。
内容为：
```json
{
	"roughnessScale" : 1.0,
	"displacementScale" : 0.6,
	"specularLevel" : 0.04
}
```

和你的普通PBR贴图的参数并无区别，除了不需要指定贴图地址。有了这个文件，Community Shaders才会把这个Texture Set认定为PBR。
