---
layout: post
title: "[Claude+DSv4 译] Filament 中的基于物理的渲染"
date: 2026-07-07
categories: tech
author: "Romain Guy, Mathias Agopian — 翻译: matheecs"
description: "Google Filament 引擎 PBR 文档的完整中文翻译，涵盖材质系统、光照模型、成像管线等物理渲染核心理论与实现。"
---

> **原文**: [Physically Based Rendering in Filament](https://google.github.io/filament/Filament.md.html)  
> **作者**: [Romain Guy](https://github.com/romainguy), [Mathias Agopian](https://github.com/pixelflinger)  
> **源码仓库**: [google/filament](https://github.com/google/filament)  
> **翻译风格**: 科技类学术风格，忠实原文，保留所有数学公式、代码块及文献引用。

---

**Filament 中的基于物理的渲染**

<div align="center"><img src="https://google.github.io/filament/images/filament_logo.png"/></div>

# 关于

本文档是 [Filament 项目](https://github.com/google/filament) 的一部分。如发现本文档中的错误，请使用[项目的问题追踪器](https://github.com/google/filament/issues)进行报告。

## 作者

- [Romain Guy](https://github.com/romainguy), [@romainguy](https://twitter.com/romainguy)
- [Mathias Agopian](https://github.com/pixelflinger), [@darthmoosious](https://twitter.com/darthmoosious)

# 概述

Filament 是一个基于 Android 的物理渲染引擎。Filament 的目标是为 Android 开发者提供一套工具和 API，使他们能够轻松创建高质量的 2D 和 3D 渲染。

本文档旨在解释 Filament 中使用的材质和光照模型背后的方程和理论。本文档可作为 Filament 贡献者或对引擎内部实现感兴趣的开发者的参考。我们将在必要时提供代码片段，以使理论与实践之间的关系尽可能清晰。

本文档并非设计文档。它专注于算法本身，其内容可用于在任何引擎中实现 PBR。不过，本文档也会解释我们为何选择某些特定算法/模型而非其他方案。

除非另有说明，本文档中的所有 3D 渲染图均在引擎内部生成（原型或产品版本）。其中许多 3D 渲染图是在 Filament 开发的早期阶段捕获的，不能代表最终质量。

## 原则

实时渲染是一个活跃的研究领域，对于每一个需要实现的功能，都有大量的方程、算法和实现可供选择（例如，《渲染实时阴影》一书就是一本 400 页的概述，总结了数十种阴影渲染技术）。因此，在做出明智的决策之前，我们必须首先明确我们的目标（或原则，遵循 Brent Burley 的开创性论文《Physically-based shading at Disney》[#Burley12]）。

实时移动性能
:   我们的首要目标是设计和实现一个能够在移动平台上高效运行的渲染系统。主要目标 GPU 为 OpenGL ES 3.x 级别。

质量
:   我们的渲染系统将强调整体画面质量。但为了支持低端和中端性能 GPU，我们会接受一定程度的质量折中。

易用性
:   美工人员需要能够频繁且快速地迭代他们的资产，我们的渲染系统必须使他们能够直观地完成这一工作。因此，我们必须提供易于理解的参数（例如，不使用镜面反射幂次参数）。

    我们也理解并非所有开发者都有与美工人员合作的奢侈条件。我们系统的基于物理的方法将使开发者能够无需理解背后的实现理论即可制作出视觉上可信的材质。

    对于美工人员和开发者而言，我们的系统将依赖尽可能少的参数，以减少试错成本，并让用户能够快速掌握材质模型。

    此外，任何参数值的组合都应产生物理上可信的结果。难以创建物理上不可信的材质。

熟悉性
:   我们的系统应在所有可能之处使用物理单位：距离以米或厘米为单位，色温以开尔文为单位，光照单位以流明或坎德拉为单位，等等。

灵活性
:   基于物理的方法不应排斥非真实感渲染。例如，用户界面可能需要无光照材质。

部署体积
:   虽然与本文档内容没有直接关系，但需要强调的是，我们希望尽可能保持渲染库的小巧，以便任何应用都可以将其打包，而不会使二进制文件增大到不可接受的程度。

## 基于物理的渲染

我们选择采用 PBR，是因为它在艺术效果和制作效率方面具有优势，并且与我们设定的目标相一致。

基于物理的渲染是一种渲染方法，与传统的实时模型相比，它能更准确地表现材质及其与光的交互方式。PBR 方法核心中材质与光照的分离，使得创建在不同光照条件下都能呈现准确效果的逼真资产变得更加容易。

# 符号约定

$$
\newcommand{NoL}{n \cdot l}
\newcommand{NoV}{n \cdot v}
\newcommand{NoH}{n \cdot h}
\newcommand{VoH}{v \cdot h}
\newcommand{LoH}{l \cdot h}
\newcommand{fNormal}{f_{0}}
\newcommand{fDiffuse}{f_d}
\newcommand{fSpecular}{f_r}
\newcommand{fX}{f_x}
\newcommand{aa}{\alpha^2}
\newcommand{fGrazing}{f_{90}}
\newcommand{schlick}{F_{Schlick}}
\newcommand{nior}{n_{ior}}
\newcommand{Ed}{E_d}
\newcommand{Lt}{L_{\bot}}
\newcommand{Lout}{L_{out}}
\newcommand{cosTheta}{\left< \cos \theta \right> }
$$

本文档中的方程使用表 [symbols] 中描述的符号。

|          符号             |           定义
|:-------------------------:|:---------------------------|
| $v$                       | 视线单位向量
| $l$                       | 入射光单位向量
| $n$                       | 表面法线单位向量
| $h$                       | $l$ 和 $v$ 之间的半角单位向量
| $f$                       | BRDF
| $\fDiffuse$               | BRDF 的漫反射分量
| $\fSpecular$              | BRDF 的镜面反射分量
| $\alpha$                  | 粗糙度，通过输入的 `perceptualRoughness` 重新映射得到
| $\sigma$                  | 漫反射率
| $\Omega$                  | 球面域
| $\fNormal$                | 法线入射时的反射率
| $\fGrazing$               | 掠射角时的反射率
| $\chi^+(a)$               | 阶跃函数（若 $a > 0$ 则为 1，否则为 0）
| $n_{ior}$                 | 界面的折射率 (IOR)
| $\left< \NoL \right>$     | 截断到 [0..1] 的点积
| $\left< a \right>$        | 饱和值（截断到 [0..1]）
[表 [symbols]：符号定义]

# 材质系统

以下各节描述了多种材质模型，以简化对各种表面特征（如各向异性或透明涂层层）的说明。然而在实际中，其中一些模型被合并为一个模型。例如，标准模型、透明涂层模型和各向异性模型可以组合成一个更灵活、更强大的单一模型。请参考[材质文档](./Materials.md.html)以了解 Filament 中实现的材质模型的描述。
## 标准模型

我们的模型旨在表示标准材质外观。材质模型通过BSDF（双向散射分布函数，Bidirectional Scattering Distribution Function）进行数学描述，而BSDF本身又由两个函数组成：BRDF（双向反射分布函数，Bidirectional Reflectance Distribution Function）和BTDF（双向透射分布函数，Bidirectional Transmittance Function）。

由于我们的目标是模拟常见表面，标准材质模型将重点关注BRDF，而忽略BTDF或对其进行大幅近似。因此，我们的标准模型仅能正确模拟具有短平均自由程的反射性、各向同性、介电或导电表面。

BRDF描述了标准材质的表面响应，其由两个项组成的函数构成：
- 漫反射分量，即 $f_d$
- 镜面反射分量，即 $f_r$

表面、表面法线、入射光与这些项之间的关系如图[frFd]所示（此处暂不考虑次表面散射）：

<div align="center"><img src="https://google.github.io/filament/images/diagram_fr_fd.png"/></div>

完整的表面响应可表示为：

$$\begin{equation}\label{brdf}
f(v,l)=f_d(v,l)+f_r(v,l)
\end{equation}$$

该方程描述了来自单一方向的入射光所产生的表面响应。完整的渲染方程需要对整个半球面上的$l$进行积分。

常见的表面通常并非由平坦界面构成，因此我们需要一个能够描述光与不规则界面相互作用的模型。

微平面BRDF（microfacet BRDF）是一种非常适合此目的的物理可信BRDF。该BRDF理论认为，表面在微观层面上并非光滑，而是由大量随机朝向的平面表面碎片（称为微平面，microfacet）组成。图[microfacetVsFlat]展示了微观层面上平坦界面与不规则界面之间的差异：

<div align="center"><img src="https://google.github.io/filament/images/diagram_microfacet.png"/></div>

只有法线方向位于光线方向与视线方向之间的微平面才会反射可见光，如图[microfacets]所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_macrosurface.png"/></div>

然而，并非所有具有正确法线朝向的微平面都会贡献反射光，因为BRDF还考虑了遮蔽（masking）和阴影（shadowing）效应。如图[microfacetShadowing]所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_shadowing_masking.png"/></div>

微平面BRDF受_粗糙度_参数影响很大，该参数描述了表面在微观层面的光滑程度（低粗糙度）或粗糙程度（高粗糙度）。表面越光滑，微平面的排列越整齐，反射光越明显。表面越粗糙，朝向相机的微平面越少，入射光在反射后向相机以外方向散射，使得镜面高光呈现模糊效果。

图[roughness]展示了不同粗糙度的表面及其与光的相互作用。

<div align="center"><img src="https://google.github.io/filament/images/diagram_roughness.png"/></div>

!!! Note: 关于粗糙度
    用户设置的粗糙度参数在整个文档的着色器代码片段中被称为`perceptualRoughness`。名为`roughness`的变量是对`perceptualRoughness`进行重映射后的结果，详见[Parameterization]一节。

微平面模型由以下方程描述（其中x代表镜面反射或漫反射分量）：

$$\begin{equation}
\fX(v,l) = \frac{1}{| \NoV | | \NoL |}
\int_\Omega D(m,\alpha) G(v,l,m) f_m(v,l,m) (v \cdot m) (l \cdot m) dm
\end{equation}$$

项$D$用于建模微平面的分布（该项也称为NDF或法线分布函数，Normal Distribution Function）。该项在表面外观中起着至关重要的作用，如图[roughness]所示。

项$G$用于建模微平面的可见性（或称遮挡、阴影遮蔽）。

由于该方程对镜面反射和漫反射分量均适用，其差异在于微平面BRDF $f_m$本身。

值得注意的是，该方程用于在_微观层面_上对半球进行积分：

<div align="center"><img src="https://google.github.io/filament/images/diagram_micro_vs_macro.png"/></div>

上图显示，在宏观层面上，表面被视为平坦的。这有助于简化方程，因为我们假设从单一方向照明的着色片段对应于表面上的单个点。

然而在微观层面上，表面并非平坦，我们不能再假设为单条光线（不过我们可以假设入射光线是平行的）。由于微平面会使一束平行入射光线向不同方向散射，我们必须对半球面上的表面响应进行积分，上图中记作m。

对每个着色片段计算微平面半球上的完整积分显然不切实际。因此，我们将对镜面反射和漫反射分量分别采用积分近似方法。

## 介电体与导体

为了更好地理解下文展示的某些方程和行为，我们必须首先清楚理解金属（导体）与非金属（介电体）表面之间的区别。

我们之前看到，入射光照射到由BRDF描述的表面上时，光被反射为两个独立的分量：漫反射和镜面反射。这种行为的建模非常直观，如图[bsdfBrdf]所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_fr_fd.png"/></div>

这种建模是对光与表面实际相互作用方式的简化。实际上，部分入射光会穿透表面，在内部发生散射，然后以漫反射的形式再次从表面射出。这一现象如图[diffuseScattering]所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_scattering.png"/></div>

这就是导体与介电体之间的区别所在。纯金属材质不会发生次表面散射，这意味着没有漫反射分量（我们将在后面看到这会影响镜面反射分量的感知颜色）。散射发生在介电体中，这意味着它们同时具有镜面反射和漫反射分量。

因此，为了正确建模BRDF，我们必须区分介电体和导体（为清晰起见未显示散射），如图[dielectricConductor]所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_brdf_dielectric_conductor.png"/></div>

## 能量守恒

能量守恒是基于物理渲染中优秀BRDF的关键要素之一。能量守恒的BRDF要求镜面反射和漫反射的总能量小于入射总能量。如果没有能量守恒的BRDF，美术师必须手动确保表面反射的光线强度永远不会超过入射光线。

## 镜面反射BRDF

对于镜面反射项，$f_r$是一种镜面BRDF，可用菲涅耳定律建模，在微平面模型积分的Cook-Torrance近似中记作$F$：

$$\begin{equation}
f_r(v,l) = \frac{D(h, \alpha) G(v, l, \alpha) F(v, h, f0)}{4(\NoV)(\NoL)}
\end{equation}$$

考虑到实时渲染的约束，我们必须对$D$、$G$和$F$这三个项使用近似方法。[#Karis13a]整理了一份适用于Cook-Torrance镜面BRDF的这三个项的公式列表。后续章节将描述我们为这些项选择的方程。

### 法线分布函数（镜面D项）

[#Burley12]指出，长尾法线分布函数（NDF）非常适合真实世界表面。[#Walter07]描述的GGX分布是一种在高光区域具有长尾衰减和短峰值的分布，其公式简单，适合实时实现。在现代基于物理的渲染器中，它也是一种流行的模型，等效于Trowbridge-Reitz分布。

$$\begin{equation}
D_{GGX}(h,\alpha) = \frac{\aa}{\pi ( (\NoH)^2 (\aa - 1) + 1)^2}
\end{equation}$$

NDF的GLSL实现，如列表[specularD]所示，简洁高效。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float D_GGX(float NoH, float roughness) {
    float a = NoH * roughness;
    float k = roughness / (1.0 - NoH * NoH + a * a);
    return k * k * (1.0 / PI);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularD]: Implementation of the specular D term in GLSL]

我们可以通过使用半精度浮点数来改进这一实现。这种优化需要对原始方程进行修改，因为在半精度浮点数中计算$1 - (\NoH)^2$存在两个问题。首先，当$(\NoH)^2$接近1（高光区域）时，该计算存在浮点数对消问题。其次，$\NoH$在1附近没有足够的精度。

解决方案涉及拉格朗日恒等式：

$$\begin{equation}
| a \times b |^2 = |a|^2 |b|^2 - (a \cdot b)^2
\end{equation}$$

由于$n$和$h$都是单位向量，因此$|n \times h|^2 = 1 - (\NoH)^2$。这使我们能够通过简单的叉积运算直接使用半精度浮点数计算$1 - (\NoH)^2$。列表[specularDfp16]展示了最终的优化实现。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#define MEDIUMP_FLT_MAX    65504.0
#define saturateMediump(x) min(x, MEDIUMP_FLT_MAX)

float D_GGX(float roughness, float NoH, const vec3 n, const vec3 h) {
    vec3 NxH = cross(n, h);
    float a = NoH * roughness;
    float k = roughness / (dot(NxH, NxH) + a * a);
    float d = k * k * (1.0 / PI);
    return saturateMediump(d);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularDfp16]: Implementation of the specular D term in GLSL optimized for fp16]

### 几何阴影（镜面G项）

Eric Heitz在[#Heitz14]中指出，Smith几何阴影函数是正确且精确的$G$项。Smith公式如下：

$$\begin{equation}
G(v,l,\alpha) = G_1(l,\alpha) G_1(v,\alpha)
\end{equation}$$

$G_1$可以采用多种模型，通常设置为GGX公式：

$$\begin{equation}
G_1(v,\alpha) = G_{GGX}(v,\alpha) = \frac{2 (\NoV)}{\NoV + \sqrt{\aa + (1 - \aa) (\NoV)^2}}
\end{equation}$$

完整的Smith-GGX公式变为：

$$\begin{equation}
G(v,l,\alpha) = \frac{2 (\NoL)}{\NoL + \sqrt{\aa + (1 - \aa) (\NoL)^2}} \frac{2 (\NoV)}{\NoV + \sqrt{\aa + (1 - \aa) (\NoV)^2}}
\end{equation}$$

我们可以观察到，被除数$2 (\NoL)$和$2 (n \cdot v)$允许我们通过引入可见性函数$V$来简化原始函数$f_r$：

$$\begin{equation}
f_r(v,l) = D(h, \alpha) V(v, l, \alpha) F(v, h, f_0)
\end{equation}$$

其中：

$$\begin{equation}
V(v,l,\alpha) = \frac{G(v, l, \alpha)}{4 (\NoV) (\NoL)} = V_1(l,\alpha) V_1(v,\alpha)
\end{equation}$$

以及：

$$\begin{equation}
V_1(v,\alpha) = \frac{1}{\NoV + \sqrt{\aa + (1 - \aa) (\NoV)^2}}
\end{equation}$$

然而，Heitz指出，考虑微平面的高度以关联遮蔽和阴影可以得到更精确的结果。他定义的高度相关Smith函数如下：

$$\begin{equation}
G(v,l,h,\alpha) = \frac{\chi^+(\VoH) \chi^+(\LoH)}{1 + \Lambda(v) + \Lambda(l)}
\end{equation}$$

$$\begin{equation}
\Lambda(m) = \frac{-1 + \sqrt{1 + \aa tan^2(\theta_m)}}{2} = \frac{-1 + \sqrt{1 + \aa \frac{(1 - cos^2(\theta_m))}{cos^2(\theta_m)}}}{2}
\end{equation}$$

将$cos(\theta_m)$替换为$\NoV$，我们得到：

$$\begin{equation}
\Lambda(v) = \frac{1}{2} \left( \frac{\sqrt{\aa + (1 - \aa)(\NoV)^2}}{\NoV} - 1 \right)
\end{equation}$$

由此可推导出可见性函数：

$$\begin{equation}
V(v,l,\alpha) = \frac{0.5}{\NoL \sqrt{(\NoV)^2 (1 - \aa) + \aa} + \NoV \sqrt{(\NoL)^2 (1 - \aa) + \aa}}
\end{equation}$$

可见性项的GLSL实现，如列表[specularV]所示，由于需要两次`sqrt`运算，其开销略高于我们的预期。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float V_SmithGGXCorrelated(float NoV, float NoL, float roughness) {
    float a2 = roughness * roughness;
    float GGXV = NoL * sqrt(NoV * NoV * (1.0 - a2) + a2);
    float GGXL = NoV * sqrt(NoL * NoL * (1.0 - a2) + a2);
    return 0.5 / (GGXV + GGXL);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularV]: Implementation of the specular V term in GLSL]

注意到平方根下的所有项都是平方数且所有项都在$[0..1]$范围内，我们可以通过近似方法来优化此可见性函数：

$$\begin{equation}
V(v,l,\alpha) = \frac{0.5}{\NoL (\NoV (1 - \alpha) + \alpha) + \NoV (\NoL (1 - \alpha) + \alpha)}
\end{equation}$$

该近似在数学上并不精确，但节省了两次平方根运算，对于实时移动应用而言已经足够，如列表[approximatedSpecularV]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float V_SmithGGXCorrelatedFast(float NoV, float NoL, float roughness) {
    float a = roughness;
    float GGXV = NoL * (NoV * (1.0 - a) + a);
    float GGXL = NoV * (NoL * (1.0 - a) + a);
    return 0.5 / (GGXV + GGXL);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [approximatedSpecularV]: Implementation of the approximated specular V term in GLSL]

[#Hammon17]基于相同的观察（平方根可以被移除）提出了同样的近似方法。该方法通过将表达式重写为_lerp_（线性插值）来实现：

$$\begin{equation}
V(v,l,\alpha) = \frac{0.5}{lerp(2 (\NoL) (\NoV), \NoL + \NoV, \alpha)}
\end{equation}$$

### 菲涅耳效应（镜面F项）

菲涅耳效应对基于物理材质的视觉效果起着重要作用。该效应描述了观察者从表面看到的反射光量取决于观察角度这一事实。大面积水域是体验这一现象的完美方式，如图[fresnelLake]所示。当垂直向下看水面（法线入射）时，你可以看穿水体。然而，当向远处眺望（掠射角，此时感知到的光线与表面趋于平行）时，你会看到水面上的镜面反射变得更加明显。

反射光量不仅取决于观察角度，还取决于材质的折射率（IOR）。在法线入射（垂直于表面，即0度角）时，反射回的光量记为$\fNormal$，可从折射率导出，我们将在[反射率重映射]一节中看到。在掠射角反射回的光量记为$\fGrazing$，对于光滑材质接近100%。

<div align="center"><img src="https://google.github.io/filament/images/photo_fresnel_lake.jpg"/></div>

更正式地说，菲涅耳项定义了光在两种不同介质的界面处如何反射和折射，即反射能量与透射能量的比率。[#Schlick94]描述了一种用于Cook-Torrance镜面BRDF的菲涅耳项廉价近似方法：

$$\begin{equation}
F_{Schlick}(v,h,\fNormal,\fGrazing) = \fNormal + (\fGrazing - \fNormal)(1 - \VoH)^5
\end{equation}$$

常数$\fNormal$表示法线入射时的镜面反射率，对于介电体是无色（消色差）的，对于金属是有色的。实际值取决于界面的折射率。该项的GLSL实现需要使用`pow`函数，如列表[specularF]所示，该函数可以用几次乘法替代。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 F_Schlick(float u, vec3 f0, float f90) {
    return f0 + (vec3(f90) - f0) * pow(1.0 - u, 5.0);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularF]: Implementation of the specular F term in GLSL]

该菲涅耳函数可视为在入射镜面反射率与掠射角反射率（此处由$\fGrazing$表示）之间进行插值。对真实世界材质的观察表明，介电体和导体在掠射角都呈现无色镜面反射，且90度时的菲涅耳反射率为1.0。更精确的$\fGrazing$值将在[镜面反射遮挡]一节中讨论。

将$\fGrazing$设置为1后，可以通过略微重构代码来优化Schlick近似菲涅耳项的标量运算。结果如列表[scalarSpecularF]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 F_Schlick(float u, vec3 f0) {
    float f = pow(1.0 - u, 5.0);
    return f + f0 * (1.0 - f);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [scalarSpecularF]: Scalar optimization of the specular F term in GLSL]
## 漫反射BRDF

在漫反射项中，$f_m$ 为朗伯函数，BRDF的漫反射项变为：

$$\begin{equation}
\fDiffuse(v,l) = \frac{\sigma}{\pi} \frac{1}{| \NoV | | \NoL |}
\int_\Omega D(m,\alpha) G(v,l,m) (v \cdot m) (l \cdot m) dm
\end{equation}$$

我们的实现将改用简单的朗伯BRDF，该BRDF假设微半球上存在均匀的漫反射响应：

$$\begin{equation}
\fDiffuse(v,l) = \frac{\sigma}{\pi}
\end{equation}$$

在实际中，漫反射率$\sigma$在后续步骤中相乘，如清单[diffuseBRDF]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float Fd_Lambert() {
    return 1.0 / PI;
}

vec3 Fd = diffuseColor * Fd_Lambert();
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[清单[diffuseBRDF]：GLSL中朗伯漫反射BRDF的实现]

朗伯BRDF显然极为高效，且其结果与更复杂模型的结果足够接近。

然而，漫反射部分理想情况下应与高光项保持一致，并将表面粗糙度考虑在内。Disney漫反射BRDF[#Burley12]和Oren-Nayar模型[#Oren94]均将粗糙度考虑在内，并在掠射角处产生一定的后向反射。基于我们的限制条件，我们认为额外的运行时开销并不足以证明质量上的小幅提升是合理的。这种复杂的漫反射模型也使得基于图像的光照和球谐函数的表达与实现变得更加困难。

为完整起见，[#Burley12]中表达的Disney漫反射BRDF如下：

$$\begin{equation}
\fDiffuse(v,l) = \frac{\sigma}{\pi} \schlick(n,l,1,\fGrazing) \schlick(n,v,1,\fGrazing)
\end{equation}$$

其中：

$$\begin{equation}
\fGrazing=0.5 + 2 \cdot \alpha cos^2(\theta_d)
\end{equation}$$

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float F_Schlick(float u, float f0, float f90) {
    return f0 + (f90 - f0) * pow(1.0 - u, 5.0);
}

float Fd_Burley(float NoV, float NoL, float LoH, float roughness) {
    float f90 = 0.5 + 2.0 * roughness * LoH * LoH;
    float lightScatter = F_Schlick(NoL, 1.0, f90);
    float viewScatter = F_Schlick(NoV, 1.0, f90);
    return lightScatter * viewScatter * (1.0 / PI);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[清单[diffuseBRDF]：GLSL中Disney漫反射BRDF的实现]

图[lambert_vs_disney]展示了使用完全粗糙的电介质材质时，简单朗伯漫反射BRDF与更高质量的Disney漫反射BRDF之间的对比。为了便于比较，右侧球体被设置为镜面反射。两种BRDF的表面响应非常相似，但Disney BRDF在掠射角处展现出一些漂亮的后向反射（仔细观察球体的左边缘）。

<div align="center"><img src="https://google.github.io/filament/images/diagram_lambert_vs_disney.png" alt="图[lambert_vs_disney]：朗伯漫反射BRDF（左）与Disney漫反射BRDF（右）的对比"/></div>

我们可以允许美术人员/开发者根据其所需的质量和目标设备的性能来选择使用Disney漫反射BRDF。然而，需要注意的是，这里所表达的Disney漫反射BRDF并不满足能量守恒。

## 标准模型总结

**高光项（镜面反射项）**：Cook-Torrance镜面反射微平面模型，包含GGX法线分布函数、Smith-GGX高度相关可见性函数和Schlick菲涅尔函数。

**漫反射项**：朗伯漫反射模型。

完整的GLSL标准模型实现如清单[glslBRDF]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float D_GGX(float NoH, float a) {
    float a2 = a * a;
    float f = (NoH * a2 - NoH) * NoH + 1.0;
    return a2 / (PI * f * f);
}

vec3 F_Schlick(float u, vec3 f0) {
    return f0 + (vec3(1.0) - f0) * pow(1.0 - u, 5.0);
}

float V_SmithGGXCorrelated(float NoV, float NoL, float a) {
    float a2 = a * a;
    float GGXL = NoV * sqrt((-NoL * a2 + NoL) * NoL + a2);
    float GGXV = NoL * sqrt((-NoV * a2 + NoV) * NoV + a2);
    return 0.5 / (GGXV + GGXL);
}

float Fd_Lambert() {
    return 1.0 / PI;
}

void BRDF(...) {
    vec3 h = normalize(v + l);

    float NoV = abs(dot(n, v)) + 1e-5;
    float NoL = clamp(dot(n, l), 0.0, 1.0);
    float NoH = clamp(dot(n, h), 0.0, 1.0);
    float LoH = clamp(dot(l, h), 0.0, 1.0);

    // perceptually linear roughness to roughness (see parameterization)
    float roughness = perceptualRoughness * perceptualRoughness;

    float D = D_GGX(NoH, roughness);
    vec3  F = F_Schlick(LoH, f0);
    float V = V_SmithGGXCorrelated(NoV, NoL, roughness);

    // specular BRDF
    vec3 Fr = (D * V) * F;

    // diffuse BRDF
    vec3 Fd = diffuseColor * Fd_Lambert();

    // apply lighting...
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[清单[glslBRDF]：GLSL中BRDF的计算]
## 改进BRDF

我们在[能量守恒]一节中提到，能量守恒是优秀BRDF的关键要素之一。遗憾的是，之前探讨的BRDF存在两个问题，下面我们将对此进行分析。

### 漫反射的能量增益

Lambert漫反射BRDF没有考虑在表面发生反射、因而无法参与漫反射散射事件的那部分光线。

[TODO: 讨论fr+fd的相关问题]

### 镜面反射的能量损失

我们之前介绍的Cook-Torrance BRDF试图在微表面层面上模拟多次事件，但其仅考虑了光的单次反弹。这种近似在粗糙度较高时会导致能量损失，即表面不满足能量守恒。图[singleVsMultiBounce]展示了这种能量损失产生的原因。在单次反弹（或单次散射）模型中，击中表面的光线可能会被反射到另一个微表面上，从而由于遮挡和阴影项而被丢弃。然而，如果我们考虑多次反弹（多重散射），同一光线可能会从微表面场中逃逸，并最终反射回观察者方向。

<div align="center"><img src="https://google.github.io/filament/images/diagram_single_vs_multi_scatter.png" alt="Figure [singleVsMultiBounce]: Single scattering (left) vs multiscattering"/></div>

基于这一简单解释，我们可以直观地推导出：表面越粗糙，由于未能考虑多重散射事件而导致能量损失的可能性就越大。这种能量损失表现为粗糙材料变暗。金属表面尤其容易受到影响，因为其所有反射都是镜面反射。图[metallicRoughEnergyLoss]展示了这种变暗效应。通过多重散射，可以实现能量守恒，如图[metallicRoughEnergyPreservation]所示。

<div align="center"><img src="https://google.github.io/filament/images/material_metallic_energy_loss.png" alt="Figure [metallicRoughEnergyLoss]: Darkening increases with roughness due to single scattering"/></div>

<div align="center"><img src="https://google.github.io/filament/images/material_metallic_energy_preservation.png" alt="Figure [metallicRoughEnergyPreservation]: Energy preservation with multiscattering"/></div>

我们可以使用白色熔炉（white furnace），即一个设置为纯白色的均匀照明环境，来验证BRDF的能量守恒性质。当实现能量守恒时，一个完全反射的金属表面（$\fNormal = 1$）无论粗糙度如何，都应无法与背景区分开来。图[whiteFurnaceLoss]展示了使用前几节中介绍的镜面BRDF时此类表面的表现。随着粗糙度增加，能量损失显而易见。相比之下，图[whiteFurnacePreservation]表明，考虑多重散射事件可以解决能量损失问题。

<div align="center"><img src="https://google.github.io/filament/images/material_furnace_energy_loss.png" alt="Figure [whiteFurnaceLoss]: Darkening increases with roughness due to single scattering"/></div>

<div align="center"><img src="https://google.github.io/filament/images/material_furnace_energy_preservation.png" alt="Figure [whiteFurnacePreservation]: Energy preservation with multiscattering"/></div>

多重散射微表面BRDF在[#Heitz16]中有深入讨论。遗憾的是，该论文仅提出了多重散射BRDF的随机评估方法，因此该方案不适用于实时渲染。Kulla和Conty在[#Kulla17]中提出了一种不同的方法。其思路是添加一个能量补偿项，作为额外的BRDF叶瓣，如公式$\ref{energyCompensationLobe}$所示：

$$\begin{equation}\label{energyCompensationLobe}
f_{ms}(l,v) = \frac{(1 - E(l)) (1 - E(v)) F_{avg}^2 E_{avg}}{\pi (1 - E_{avg}) (1 - F_{avg}(1 - E_{avg}))}
\end{equation}$$

其中$E$是镜面BRDF $f_r$的方向性反照率，且$\fNormal$设为1：

$$\begin{equation}
E(l) = \int_{\Omega} f(l,v) (\NoV) dv
\end{equation}$$

$E_{avg}$是$E$的余弦加权平均值：

$$\begin{equation}
E_{avg} = 2 \int_0^1 E(\mu) \mu d\mu
\end{equation}$$

类似地，$F_{avg}$是菲涅尔项的余弦加权平均值：

$$\begin{equation}
F_{avg} = 2 \int_0^1 F(\mu) \mu d\mu
\end{equation}$$

$E$和$E_{avg}$都可以预先计算并存储在查找表中。而$F_{avg}$在使用Schlick近似时可以大幅简化：

$$\begin{equation}\label{averageFresnel}
F_{avg} = \frac{1 + 20 \fNormal}{21}
\end{equation}$$

这个新叶瓣与原始的单次散射叶瓣（先前记为$f_r$）相结合：

$$\begin{equation}
f_{r}(l,v) = f_{ss}(l,v) + f_{ms}(l,v)
\end{equation}$$

在[#Lagarde18]中，Lagarde和Golubev（感谢Emmanuel Turquin的贡献）指出，公式$\ref{averageFresnel}$可以简化为$\fNormal$。他们还提出通过添加一个缩放的GGX镜面叶瓣来实现能量补偿：

$$\begin{equation}\label{energyCompensation}
f_{ms}(l,v) = \fNormal \frac{1 - E(l)}{E(l)} f_{ss}(l,v)
\end{equation}$$

关键洞察在于，$E(l)$不仅可以预先计算，还可以与基于图像的光照预积分共享。因此，多重散射能量补偿公式变为：

$$\begin{equation}\label{scaledEnergyCompensationLobe}
f_r(l,v) = f_{ss}(l,v) + \fNormal \left( \frac{1}{r} - 1 \right) f_{ss}(l,v)
\end{equation}$$

其中$r$定义为：

$$\begin{equation}
r = \int_{\Omega} D(l,v) V(l,v) \left< \NoL \right> dl
\end{equation}$$

如果我们将$r$存储在[基于图像的光照]一节中介绍的DFG查找表中，就可以以极小的代价实现镜面能量补偿。代码清单[energyCompensationImpl]展示了其实现直接来源于公式$\ref{scaledEnergyCompensationLobe}$的转换。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 energyCompensation = 1.0 + f0 * (1.0 / dfg.y - 1.0);
// Scale the specular lobe to account for multiscattering
Fr *= pixel.energyCompensation;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [energyCompensationImpl]: 能量补偿镜面叶瓣的实现]

请参阅[基于图像的光照]一节和[多重散射的预积分]一节，了解DFG查找表的推导和计算方法。

## 参数化

[#Burley12]中描述的Disney材质模型是一个很好的起点，但其参数过多，不适用于实时实现。此外，我们希望标准材质模型既易于理解，也便于艺术家和开发者使用。

### 标准参数

表[standardParameters]描述了满足我们约束条件的参数列表。


       Parameter      |      Definition
---------------------:|:---------------------
**BaseColor**         | 非金属表面的漫反射反照率，以及金属表面的镜面反射颜色
**Metallic**          | 表面表现为电介质（0.0）还是导体（1.0）。通常用作二值（0或1）
**Roughness**         | 表面的感知光滑度（0.0）或粗糙度（1.0）。光滑表面呈现清晰的反射
**Reflectance**       | 电介质表面在法线入射时的菲涅尔反射率。替代了显式的折射率
**Emissive**          | 附加漫反射反照率，用于模拟自发光表面（如霓虹灯等）。该参数在带有泛光（bloom）效果的HDR管线中尤为有用
**Ambient occlusion** | 定义环境光对表面点的可达程度。这是一个介于0.0和1.0之间的逐像素阴影因子。该参数将在光照部分详细讨论
[Table [standardParameters]: 标准模型的参数]

图[material_parameters]展示了金属度、粗糙度和反射率参数如何影响表面的外观。

<div align="center"><img src="https://google.github.io/filament/images/material_parameters.png" alt="Figure [material_parameters]: From top to bottom: varying metallic, varying dielectric roughness, varying metallic roughness, varying reflectance"/></div>

### 类型与范围

理解材质模型不同参数的类型和范围非常重要，如表[standardParametersTypes]所示。


       Parameter      |    Type and range
---------------------:|:---------------------
**BaseColor**         | 线性RGB [0..1]
**Metallic**          | 标量 [0..1]
**Roughness**         | 标量 [0..1]
**Reflectance**       | 标量 [0..1]
**Emissive**          | 线性RGB [0..1] + 曝光补偿
**Ambient occlusion** | 标量 [0..1]
[Table [standardParametersTypes]: 标准模型参数的范围和类型]

需要注意的是，这里描述的类型和范围是着色器所期望的格式。API和/或工具UI应当允许使用其他对艺术家更直观的类型和范围来指定参数。

例如，基础颜色可以在sRGB空间中表示，并在发送到着色器之前转换为线性空间。将金属度、粗糙度和反射率参数表示为0到255之间的灰度值（从黑到白）对艺术家来说也很有用。

另一个例子：自发光参数可以表示为色温和强度，以模拟黑体发出的光。

### 重映射

为了使标准材质模型对艺术家更易用、更直观，我们必须对_baseColor_、_roughness_和_reflectance_参数进行重映射。

#### 基础颜色重映射

材质的基础颜色受该材质"金属性"的影响。电介质具有非彩色的镜面反射率，但保留其基础颜色作为漫反射颜色。而导体则将其基础颜色用作镜面反射颜色，并且没有漫反射分量。

因此，光照方程必须使用漫反射颜色和$\fNormal$，而非基础颜色。漫反射颜色可以很容易地从基础颜色计算得出，如代码清单[baseColorToDiffuse]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 diffuseColor = (1.0 - metallic) * baseColor.rgb;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [baseColorToDiffuse]: GLSL中将基础颜色转换为漫反射颜色]

#### 反射率重映射

**电介质**

菲涅尔项依赖于$\fNormal$，即法线入射角处的镜面反射率，且对于电介质是非彩色的。我们将采用[#Lagarde14]中描述的电介质表面重映射方法：

$$\begin{equation}
\fNormal = 0.16 \cdot reflectance^2
\end{equation}$$

目标是使$\fNormal$映射到一个能够表示常见电介质表面（4%反射率）和宝石（8%至16%）菲涅尔值的范围。该映射函数设计为：输入反射率为0.5（或在线性RGB灰度中的128）时，输出4%的菲涅尔反射率值。图[reflectance]展示了这些常见值及其与映射函数的关系。

<div align="center"><img src="https://google.github.io/filament/images/diagram_reflectance.png" alt="Figure [reflectance]: Common reflectance values"/></div>

如果已知折射率（例如，空气-水界面的IOR为1.33），菲涅尔反射率可计算如下：

$$\begin{equation}\label{fresnelEquation}
\fNormal(n_{ior}) = \frac{(\nior - 1)^2}{(\nior + 1)^2}
\end{equation}$$

如果已知反射率值，我们可以计算相应的IOR：

$$\begin{equation}
n_{ior} = \frac{2}{1 - \sqrt{\fNormal}} - 1 
\end{equation}$$

表[commonMatReflectance]描述了各类材质可接受的菲涅尔反射率值（没有真实世界材质的反射率低于2%）。


          Material         |    Reflectance   |        IOR       |   Linear value
--------------------------:|:-----------------|:-----------------|:----------------
Water                      | 2%               | 1.33             | 0.35
Fabric                     | 4% to 5.6%       | 1.5 to 1.62      | 0.5 to 0.59
Common liquids             | 2% to 4%         | 1.33 to 1.5      | 0.35 to 0.5
Common gemstones           | 5% to 16%        | 1.58 to 2.33     | 0.56 to 1.0
Plastics, glass            | 4% to 5%         | 1.5 to 1.58      | 0.5 to 0.56
Other dielectric materials | 2% to 5%         | 1.33 to 1.58     | 0.35 to 0.56
Eyes                       | 2.5%             | 1.38             | 0.39
Skin                       | 2.8%             | 1.4              | 0.42
Hair                       | 4.6%             | 1.55             | 0.54
Teeth                      | 5.8%             | 1.63             | 0.6
Default value              | 4%               | 1.5              | 0.5
[Table [commonMatReflectance]: 常见材质的反射率（来源：Real-Time Rendering 第4版）]

表[fNormalMetals]列出了一些金属的$\fNormal$值。这些值以sRGB形式给出，在我们的材质模型中应作为基础颜色使用。关于如何从测量数据计算这些sRGB颜色的说明，请参考附录中的[镜面反射颜色]一节。


    Metal  | $\fNormal$ in sRGB  |  Hexadecimal |               Color
----------:|:-------------------:|:------------:|-------------------------------------------------------
Silver     | 0.97, 0.96, 0.91    | #f7f4e8     | <div style="background-color: #f7f4e8; width: 60px">&nbsp;</div>
Aluminum   | 0.91, 0.92, 0.92    | #e8eaea     | <div style="background-color: #e8eaea; width: 60px">&nbsp;</div>
Titanium   | 0.76, 0.73, 0.69    | #c1baaf     | <div style="background-color: #c1baaf; width: 60px">&nbsp;</div>
Iron       | 0.77, 0.78, 0.78    | #c4c6c6     | <div style="background-color: #c4c6c6; width: 60px">&nbsp;</div>
Platinum   | 0.83, 0.81, 0.78    | #d3cec6     | <div style="background-color: #d3cec6; width: 60px">&nbsp;</div>
Gold       | 1.00, 0.85, 0.57    | #ffd891     | <div style="background-color: #ffd891; width: 60px">&nbsp;</div>
Brass      | 0.98, 0.90, 0.59    | #f9e596     | <div style="background-color: #f9e596; width: 60px">&nbsp;</div>
Copper     | 0.97, 0.74, 0.62    | #f7bc9e     | <div style="background-color: #f7bc9e; width: 60px">&nbsp;</div>
[Table [fNormalMetals]: 常见金属的$\fNormal$值]

所有材质在掠射角处的菲涅尔反射率均为100%，因此我们将在评估镜面BRDF $\fSpecular$时按如下方式设置$\fGrazing$：

$$\begin{equation}
\fGrazing = 1.0
\end{equation}$$

图[grazing_reflectance]展示了一个红色塑料球。如果仔细观察球体的边缘，可以发现掠射角处的非彩色镜面反射。

<div align="center"><img src="https://google.github.io/filament/images/material_grazing_reflectance.png" alt="Figure [grazing_reflectance]: The specular reflectance becomes achromatic at grazing angles"/></div>

**导体**

金属表面的镜面反射率是彩色的：

$$\begin{equation}
\fNormal = baseColor \cdot metallic
\end{equation}$$

代码清单[fNormal]展示了如何计算电介质和金属材质的$\fNormal$。可以看到，在金属情况下，镜面反射的颜色来源于基础颜色。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 f0 = 0.16 * reflectance * reflectance * (1.0 - metallic) + baseColor * metallic;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [fNormal]: 在GLSL中计算电介质和金属材质的$\fNormal$]

#### 粗糙度重映射与钳制

用户设置的粗糙度（此处称为`perceptualRoughness`）通过以下公式重映射到感知线性范围：

$$\begin{equation}
\alpha = perceptualRoughness^2
\end{equation}$$

图[roughness_remap]展示了一个银色金属表面随着粗糙度增加（从0.0到1.0）的效果对比：使用未修改的粗糙度值（下方）和使用重映射后的值（上方）。

<div align="center"><img src="https://google.github.io/filament/images/material_roughness_remap.png" alt="Figure [roughness_remap]: Roughness remapping comparison: perceptually linear roughness (top) and roughness (bottom)"/></div>

通过这一视觉对比，显而易见重映射后的粗糙度对艺术家和开发者来说更易于理解。如果没有这种重映射，闪亮的金属表面将不得不被限制在0.0到0.05之间的极小范围内。

Brent Burley在他的报告[#Burley12]中也做出了类似的观察。在尝试了其他重映射方法（例如三次映射和二次映射）之后，我们得出结论：这种简单的平方重映射在实现视觉上令人愉悦且直观的结果的同时，对实时应用而言计算成本低廉。

最后但同样重要的是，需要注意的是，粗糙度参数在运行时的各种计算中会被使用，而有限的浮点精度可能会成为问题。例如，在移动GPU上，_mediump_精度浮点数通常以半精度浮点数（fp16）实现。

这会在计算光照方程中的小值（如$\frac{1}{perceptualRoughness^4}$，即GGX计算中的粗糙度平方项）时导致问题。半精度浮点数能表示的最小值是$2^{-14}$或$6.1 \times 10^{-5}$。为避免在不支持非规格化数（denormals）的设备上出现除以0的情况，$\frac{1}{roughness^4}$的结果不能低于$6.1 \times 10^{-5}$。为此，我们必须将粗糙度钳制到0.089，此时结果为$6.274 \times 10^{-5}$。

还应避免非规格化数以防止性能下降。粗糙度也不应设为0，以避免明显的除以0情况。

由于我们还希望镜面高光具有最小尺寸（接近0的粗糙度会产生几乎不可见的高光），因此应在着色器中将粗糙度钳制到安全范围内。这种钳制还有一个额外的好处，即可以修正低粗糙度下可能出现的镜面走样[^frostbiteRoughnessClamp]。

[^frostbiteRoughnessClamp]: Frostbite引擎将解析光源的粗糙度钳制到0.045，以减少镜面走样。这在使用单精度浮点数（fp32）时是可行的。

### 混合与分层

如[#Burley12]和[#Neubelt13]所述，该模型允许通过对不同参数进行简单插值来实现不同材质之间的鲁棒混合。特别是，这允许使用简单的遮罩来分层不同材质。

例如，图[materialBlending]展示了Ready at Dawn工作室如何在《The Order: 1886》中使用材质混合与分层，从简单的材质库（金、铜、木、锈等）创建出复杂的外观。

<div align="center"><img src="https://google.github.io/filament/images/material_blending.png" alt="Figure [materialBlending]: Material blending and layering. Source: Ready at Dawn Studios"/></div>

材质的混合与分层本质上是对材质模型各参数的插值。图[material_interpolation]展示了一个从闪亮的金属铬到粗糙红色塑料的插值过程。尽管中间混合材质在物理意义上不太合理，但它们在视觉上是可信的。

<div align="center"><img src="https://google.github.io/filament/images/material_interpolation.png" alt="Figure [material_interpolation]: Interpolation from shiny chrome (left) to rough red plastic (right)"/></div>

### 制作基于物理的材质

一旦理解了四个主要参数（基础颜色、金属度、粗糙度和反射率）的本质，设计基于物理的材质就相当容易了。

我们提供了一份[有用的图表/参考指南](./Material%20Properties.pdf)，帮助艺术家和开发者制作自己的基于物理的材质。

<div align="center"><img src="https://google.github.io/filament/images/material_chart.jpg" alt="Crafting physically based materials"/></div>

此外，以下是使用我们材质模型的快速总结：

所有材质
:   **基础颜色** 应不包含光照信息，但微遮挡除外。

    **金属度** 几乎是二值变量。纯导体的金属度为1，纯电介质的金属度为0。应尽量使用接近0或1的值。中间值用于表面类型之间的过渡（例如金属到铁锈）。

非金属材质
:   **基础颜色** 代表反射颜色，应为50-240（严格范围）或30-240（宽松范围）内的sRGB值。

    **金属度** 应为0或接近0。

    **反射率** 如果无法找到合适的值，应设为127 sRGB（0.5线性，4%反射率）。不要使用低于90 sRGB（0.35线性，2%反射率）的值。

金属材质
:   **基础颜色** 同时代表镜面反射颜色和反射率。使用亮度为67%至100%（170-255 sRGB）的值。氧化或脏污的金属应使用比清洁金属更低的亮度，以考虑非金属成分。

    **金属度** 应为1或接近1。

    **反射率** 被忽略（从基础颜色计算得出）。
## 清漆涂层模型

之前描述的标准材质模型非常适合由单层构成的各向同性表面。遗憾的是，多层材质相当普遍，特别是在标准层之上覆盖有一层薄半透明层的材质。这类材质的现实世界例子包括汽车油漆、易拉罐、漆木、亚克力等。

<div align="center"><img src="https://google.github.io/filament/images/material_clear_coat.png"/></div>

清漆涂层可以作为标准材质模型的扩展进行模拟，通过添加第二个镜面叶瓣来实现，这意味着需要计算第二个镜面BRDF。为简化实现和参数化，清漆涂层始终为各向同性的电介质。基层可以是标准模型允许的任何类型（电介质或导体）。

由于入射光会穿过清漆涂层，我们还必须考虑能量损失，如图 [clearCoatModel] 所示。然而，我们的模型不会模拟相互反射和折射行为。

<div align="center"><img src="https://google.github.io/filament/images/diagram_clear_coat.png"/></div>

### 清漆涂层镜面BRDF

清漆涂层将使用与标准模型相同的 Cook-Torrance 微表面BRDF进行建模。由于清漆涂层始终是各向同性的电介质，且粗糙度值较低（参见 [清漆涂层参数化] 节），我们可以选择更经济的 DFG 项，而不会明显牺牲视觉质量。

对 [#Karis13a] 和 [#Burley12] 所列各项的调研表明，我们在标准模型中已使用的 Fresnel 项和 NDF 项在计算上并不比其他项更昂贵。[#Kelemen01] 描述了一个更简单的项，可以替代我们的 Smith-GGX 可见性项：

$$\begin{equation}
V(l,h) = \frac{1}{4(\LoH)^2}
\end{equation}$$

如 [#Heitz14] 所示，这种遮罩-阴影函数并非基于物理原理，但其简单性使其非常适合实时渲染。

总之，我们的清漆涂层BRDF是一个 Cook-Torrance 镜面微表面模型，采用 GGX 法线分布函数、Kelemen 可见性函数和 Schlick Fresnel 函数。清单 [kelemen] 展示了其 GLSL 实现的简洁性。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float V_Kelemen(float LoH) {
    return 0.25 / (LoH * LoH);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [kelemen]: Kelemen 可见性项的 GLSL 实现]

**关于 Fresnel 项的说明**

镜面BRDF的 Fresnel 项需要 $\fNormal$，即法线入射角下的镜面反射率。该参数可以根据界面的折射率计算得出。我们假设清漆涂层由聚氨酯或类似材料制成，这是一种[常用于涂料和清漆的常见化合物](https://en.wikipedia.org/wiki/List_of_polyurethane_applications#Varnish)。空气-聚氨酯界面[的 IOR 为 1.5](http://www.clearpur.com/transparent-polyurethanes/)，由此我们可以推导出 $\fNormal$：

$$\begin{equation}
\fNormal(1.5) = \frac{(1.5 - 1)^2}{(1.5 + 1)^2} = 0.04
\end{equation}$$

这对应于 4% 的 Fresnel 反射率，我们知道这是常见电介质材料的特征。

### 在表面响应中的集成

由于我们必须考虑清漆涂层带来的能量损失，我们可以将方程 $\ref{brdf}$ 中的BRDF重新表述如下：

$$\begin{equation}
f(v,l)=\fDiffuse(v,l) (1 - F_c) + \fSpecular(v,l) (1 - F_c) + f_c(v,l)
\end{equation}$$

其中 $F_c$ 是清漆涂层BRDF的 Fresnel 项，$f_c$ 是清漆涂层BRDF。

### 清漆涂层参数化

清漆涂层材质模型包含标准材质模型先前定义的所有参数，外加表 [clearCoatParameters] 中描述的两个参数。


        Parameter      |      Definition
----------------------:|:---------------------
**ClearCoat**          | 清漆涂层强度。标量，取值范围 0 到 1
**ClearCoatRoughness** | 清漆涂层的感知平滑度或粗糙度。标量，取值范围 0 到 1
[Table [clearCoatParameters]: 清漆涂层模型参数]

清漆涂层粗糙度参数以与标准材质粗糙度参数类似的方式进行重映射和钳制。

图 [clearCoat] 和图 [clearCoatRoughness] 展示了清漆涂层参数如何影响表面外观。

<div align="center"><img src="https://google.github.io/filament/images/material_clear_coat1.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/material_clear_coat2.png"/></div>

清单 [clearCoatBRDF] 展示了清漆涂层材质模型在重映射、参数化并集成到标准表面响应后的 GLSL 实现。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void BRDF(...) {
    // compute Fd and Fr from standard model

    // remapping and linearization of clear coat roughness
    clearCoatPerceptualRoughness = clamp(clearCoatPerceptualRoughness, 0.089, 1.0);
    clearCoatRoughness = clearCoatPerceptualRoughness * clearCoatPerceptualRoughness;

    // clear coat BRDF
    float  Dc = D_GGX(clearCoatRoughness, NoH);
    float  Vc = V_Kelemen(clearCoatRoughness, LoH);
    float  Fc = F_Schlick(0.04, LoH) * clearCoat; // clear coat strength
    float Frc = (Dc * Vc) * Fc;

    // account for energy loss in the base layer
    return color * ((Fd + Fr) * (1.0 - Fc) + Frc);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clearCoatBRDF]: 清漆涂层BRDF的 GLSL 实现]

### 基层修改

清漆涂层的存在意味着我们需要重新计算 $\fNormal$，因为它通常基于空气-材质界面。因此，基层需要基于清漆涂层-材质界面来计算 $\fNormal$。

这可以通过从 $\fNormal$ 计算材质的折射率，然后基于新计算的折射率和清漆涂层的折射率（1.5）来计算新的 $\fNormal$ 来实现。

首先，我们计算基层的折射率：

$$
IOR_{base} = \frac{1 + \sqrt{\fNormal}}{1 - \sqrt{\fNormal}}
$$

然后，我们根据这个新的折射率计算新的 $\fNormal$：

$$
f_{0_{base}} = \left( \frac{IOR_{base} - 1.5}{IOR_{base} + 1.5} \right) ^2
$$

由于清漆涂层的折射率是固定的，我们可以将两个步骤合并以简化：

$$
f_{0_{base}} = \frac{\left( 1 - 5 \sqrt{\fNormal} \right) ^2}{\left( 5 - \sqrt{\fNormal} \right) ^2}
$$

我们还应该根据清漆涂层的折射率修改基层的表观粗糙度，但目前我们选择暂不处理。

## 各向异性模型

之前描述的标准材质模型只能描述各向同性表面，即各个方向性质相同的表面。然而，许多真实世界的材质（如拉丝金属）只能使用各向异性模型来复现。

<div align="center"><img src="https://google.github.io/filament/images/material_anisotropic.png"/></div>

### 各向异性镜面BRDF

之前描述的各向同性镜面BRDF可以修改以处理各向异性材质。Burley 通过使用各向异性 GGX NDF 来实现这一点：

$$\begin{equation}
D_{aniso}(h,\alpha) = \frac{1}{\pi \alpha_t \alpha_b} \frac{1}{((\frac{t \cdot h}{\alpha_t})^2 + (\frac{b \cdot h}{\alpha_b})^2 + (\NoH)^2)^2}
\end{equation}$$

这个 NDF 不幸地依赖于两个额外的粗糙度参数，记作 $\alpha_b$（沿副切向方向的粗糙度）和 $\alpha_t$（沿切向方向的粗糙度）。Neubelt 和 Pettineo [#Neubelt13] 提出了一种通过使用 _anisotropy_（各向异性）参数从 $\alpha_t$ 推导 $\alpha_b$ 的方法，该参数描述了材质两个粗糙度值之间的关系：

$$
\begin{align*}
  \alpha_t &= \alpha \\
  \alpha_b &= lerp(0, \alpha, 1 - anisotropy)
\end{align*}
$$

[#Burley12] 中定义的关系则不同，提供了更令人愉悦和直观的结果，但计算成本稍高：

$$
\begin{align*}
  \alpha_t &= \frac{\alpha}{\sqrt{1 - 0.9 \times anisotropy}} \\
  \alpha_b &= \alpha \sqrt{1 - 0.9 \times anisotropy}
\end{align*}
$$

我们转而选择遵循 [#Kulla17] 中描述的关系，因为它可以创建锐利的高光：

$$
\begin{align*}
  \alpha_t &= \alpha \times (1 + anisotropy) \\
  \alpha_b &= \alpha \times (1 - anisotropy)
\end{align*}
$$

请注意，该 NDF 除了法线方向外，还需要切向和副切向方向。由于这些方向在法线贴图中已经需要，提供它们可能不是问题。

最终的实现如清单 [anisotropicBRDF] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float at = max(roughness * (1.0 + anisotropy), 0.001);
float ab = max(roughness * (1.0 - anisotropy), 0.001);

float D_GGX_Anisotropic(float NoH, const vec3 h,
        const vec3 t, const vec3 b, float at, float ab) {
    float ToH = dot(t, h);
    float BoH = dot(b, h);
    float a2 = at * ab;
    highp vec3 v = vec3(ab * ToH, at * BoH, a2 * NoH);
    highp float v2 = dot(v, v);
    float w2 = a2 / v2;
    return a2 * w2 * w2 * (1.0 / PI);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [anisotropicBRDF]: Burley 各向异性 NDF 的 GLSL 实现]

此外，[#Heitz14] 提出了一个各向异性遮罩-阴影函数以匹配高度相关的 GGX 分布。通过使用可见性函数，遮罩-阴影项可以大大简化：

$$\begin{equation}
G(v,l,h,\alpha) = \frac{\chi^+(\VoH) \chi^+(\LoH)}{1 + \Lambda(v) + \Lambda(l)}
\end{equation}$$

$$\begin{equation}
\Lambda(m) = \frac{-1 + \sqrt{1 + \alpha_0^2 tan^2(\theta_m)}}{2} = \frac{-1 + \sqrt{1 + \alpha_0^2 \frac{(1 - cos^2(\theta_m))}{cos^2(\theta_m)}}}{2}
\end{equation}$$

其中：

$$\begin{equation}
\alpha_0 = \sqrt{cos^2(\phi_0)\alpha_x^2 + sin^2(\phi_0)\alpha_y^2}
\end{equation}$$

经过推导，我们得到：

$$\begin{equation}
V_{aniso}(\NoL,\NoV,\alpha) = \frac{1}{2((\NoL)\hat{\Lambda}_v+(\NoV)\hat{\Lambda}_l)} \\
\hat{\Lambda}_v = \sqrt{\alpha^2_t(t \cdot v)^2+\alpha^2_b(b \cdot v)^2+(\NoV)^2} \\
\hat{\Lambda}_l = \sqrt{\alpha^2_t(t \cdot l)^2+\alpha^2_b(b \cdot l)^2+(\NoL)^2}
\end{equation}$$

项 $ \hat{\Lambda}_v $ 对于每个光源都是相同的，可以在需要时仅计算一次。最终的实现如清单 [anisotropicV] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float at = max(roughness * (1.0 + anisotropy), 0.001);
float ab = max(roughness * (1.0 - anisotropy), 0.001);

float V_SmithGGXCorrelated_Anisotropic(float at, float ab, float ToV, float BoV,
        float ToL, float BoL, float NoV, float NoL) {
    float lambdaV = NoL * length(vec3(at * ToV, ab * BoV, NoV));
    float lambdaL = NoV * length(vec3(at * ToL, ab * BoL, NoL));
    float v = 0.5 / (lambdaV + lambdaL);
    return saturateMediump(v);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [anisotropicV]: 各向异性可见性函数的 GLSL 实现]

### 各向异性参数化

各向异性材质模型包含标准材质模式先前定义的所有参数，外加表 [anisotropicParameters] 中描述的一个额外参数。


        Parameter      |      Definition
----------------------:|:---------------------
**Anisotropy**         | 各向异性程度。标量，取值范围 -1 到 1
[Table [anisotropicParameters]: 各向异性模型参数]

无需进一步的重映射。请注意，负值将使各向异性沿副切向方向对齐，而不是切向方向。图 [anisotropyParameter] 展示了各向异性参数如何影响粗糙金属表面的外观。

<div align="center"><img src="https://google.github.io/filament/images/materials/anisotropy.png"/></div>
## 次表面模型

[TODO]

### 次表面高光 BRDF

[TODO]

### 次表面参数化

[TODO]

## 布料模型

此前描述的所有材质模型都旨在模拟宏观和微观层面上的致密表面。然而，衣物和织物通常由松散连接的线构成，这些线会吸收并散射入射光。先前介绍的微平面 BRDF 由于其基本假设——表面由表现如同完美镜面的随机沟槽构成——在再现布料的特性方面表现不佳。与硬表面相比，布料的特点是具有更柔和的高光瓣，衰减范围大，并且存在由前向/后向散射引起的绒毛照明。某些织物还表现出双色调高光颜色（例如天鹅绒）。

图 [materialCloth] 展示了传统的微平面 BRDF 如何无法捕捉牛仔布样本的外观。表面显得僵硬（几乎像塑料），更像一块防水布而非衣物。该图还展示了由吸收和散射引起的较柔和的高光瓣对于忠实再现织物有多么重要。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_cloth.png" alt="图 [materialCloth]: 使用传统微平面 BRDF（左图）和我们的布料 BRDF（右图）渲染的牛仔布外观对比"/></div>

天鹅绒是布料材质模型的一个有趣用例。如图 [materialVelvet] 所示，这类织物由于前向和后向散射而表现出强烈的边缘光。这些散射事件是由直立于织物表面的纤维引起的。当入射光来自与视线方向相反的方向时，纤维会将光向前散射。类似地，当入射光来自与视线方向相同的方向时，纤维会将光向后散射。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_cloth_velvet.png" alt="图 [materialVelvet]: 展示前向和后向散射的天鹅绒织物"/></div>

由于纤维是可弯曲的，理论上我们应该能够模拟梳理表面的能力。虽然我们的模型没有复现这一特性，但它确实模拟了可见的正面高光贡献，这可归因于纤维方向的随机变化。

需要注意的是，有些类型的织物仍然最适合用硬表面材质模型来模拟。例如，皮革、丝绸和缎子可以使用标准或各向异性材质模型来再现。

### 布料高光 BRDF

我们使用的布料高光 BRDF 是 Ashikhmin 和 Premoze 在 [#Ashikhmin07] 中描述的改进型微平面 BRDF。在他们的工作中，Ashikhmin 和 Premoze 指出，分布项对 BRDF 的贡献最大，并且阴影/遮蔽项对于他们的天鹅绒分布而言并非必要。分布项本身是一个倒置的高斯分布。这有助于实现绒毛照明（前向和后向散射），同时添加一个偏移量来模拟正面高光贡献。所谓的"天鹅绒" NDF 定义如下：

$$\begin{equation}
D_{velvet}(v,h,\alpha) = c_{norm}(1 + 4 exp\left(\frac{-{cot}^2\theta_{h}}{\alpha^2}\right))
\end{equation}$$

这个 NDF 是同一位作者在 [#Ashikhmin00] 中描述的 NDF 的变体，主要修改包括添加了一个偏移量（此处设为 1）和一个幅度（4）。在 [#Neubelt13] 中，Neubelt 和 Pettineo 提出了这个 NDF 的归一化版本：

$$\begin{equation}
D_{velvet}(v,h,\alpha) = \frac{1}{\pi(1 + 4\alpha^2)} (1 + 4 \frac{exp\left(\frac{-{cot}^2\theta_{h}}{\alpha^2}\right)}{{sin}^4\theta_{h}})
\end{equation}$$

对于完整的高光 BRDF，我们也遵循 [#Neubelt13] 的方法，用更平滑的变体替换了传统的分母：

$$\begin{equation}\label{clothSpecularBRDF}
f_{r}(v,h,\alpha) = \frac{D_{velvet}(v,h,\alpha)}{4(\NoL + \NoV - (\NoL)(\NoV))}
\end{equation}$$

天鹅绒 NDF 的实现如代码清单 [clothBRDF] 所示，经过优化以适配半浮点格式，并避免计算昂贵的余切，转而依赖三角恒等式。请注意，我们从该 BRDF 中移除了菲涅尔分量。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float D_Ashikhmin(float roughness, float NoH) {
    // Ashikhmin 2007, "Distribution-based BRDFs"
	float a2 = roughness * roughness;
	float cos2h = NoH * NoH;
	float sin2h = max(1.0 - cos2h, 0.0078125); // 2^(-14/2), so sin2h^2 > 0 in fp16
	float sin4h = sin2h * sin2h;
	float cot2 = -cos2h / (a2 * sin2h);
	return 1.0 / (PI * (4.0 * a2 + 1.0) * sin4h) * (4.0 * exp(cot2) + sin4h);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clothBRDF]: 在 GLSL 中实现 Ashikhmin 的天鹅绒 NDF]

在 [#Estevez17] 中，Estevez 和 Kulla 提出了另一种 NDF（称为"Charlie"光泽度），它基于指数化的正弦函数而非倒置高斯分布。这个 NDF 具有多个优点：其参数化更自然直观，提供更柔和的外观，并且如公式 $\ref{charlieNDF}$ 所示，其实现更简单：

$$\begin{equation}\label{charlieNDF}
D(m) = \frac{(2 + \frac{1}{\alpha}) sin(\theta)^{\frac{1}{\alpha}}}{2 \pi}
\end{equation}$$

[#Estevez17] 还提出了一个新的遮蔽项，但由于其计算开销，我们在此省略。我们转而使用 [#Neubelt13] 中的可见性项（如上文公式 $\ref{clothSpecularBRDF}$ 所示）。
该 NDF 的实现如代码清单 [clothCharlieBRDF] 所示，经过优化以适配半浮点格式。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float D_Charlie(float roughness, float NoH) {
    // Estevez and Kulla 2017, "Production Friendly Microfacet Sheen BRDF"
    float invAlpha  = 1.0 / roughness;
    float cos2h = NoH * NoH;
    float sin2h = max(1.0 - cos2h, 0.0078125); // 2^(-14/2), so sin2h^2 > 0 in fp16
    return (2.0 + invAlpha) * pow(sin2h, invAlpha * 0.5) / (2.0 * PI);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clothCharlieBRDF]: 在 GLSL 中实现"Charlie"NDF]

#### 光泽颜色

为了更好地控制布料的外观，并让用户能够再现双色调高光材质，我们引入了直接修改高光反射率的能力。图 [materialClothSheen] 展示了使用我们称为"光泽颜色"的参数的示例。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_cloth_sheen.png" alt="图 [materialClothSheen]: 蓝色织物，无光泽（左）和有光泽（右）的对比"/></div>

### 布料漫反射 BRDF

我们的布料材质模型仍然依赖于朗伯漫反射 BRDF。但对其进行了轻微修改以实现能量守恒（类似于我们清漆材质模型的能量守恒），并提供了一个可选的次表面散射项。这个额外项并非基于物理，可用于模拟光在某些类型织物中的散射、部分吸收和再发射。

首先，是不含可选次表面散射的漫反射项：

$$\begin{equation}
f_{d}(v,h) = \frac{c_{diff}}{\pi}(1 - F(v,h))
\end{equation}$$

其中 $F(v,h)$ 是公式 $\ref{clothSpecularBRDF}$ 中布料高光 BRDF 的菲涅尔项。在实际操作中，我们选择在漫反射分量中省略 $1 - F(v, h)$ 项。其效果较为微妙，我们认为这不值得增加额外的计算成本。

次表面散射使用包裹漫反射照明技术实现，采用其能量守恒形式：

$$\begin{equation}
f_{d}(v,h) = \frac{c_{diff}}{\pi}(1 - F(v,h)) \left< \frac{\NoL + w}{(1 + w)^2} \right> \left< c_{subsurface} + \NoL \right>
\end{equation}$$

其中 $w$ 是介于 0 和 1 之间的值，定义了漫反射光应在终止线周围包裹的程度。为避免引入另一个参数，我们固定 $w = 0.5$。请注意，使用包裹漫反射照明时，漫反射项不得乘以 $\NoL$。这种廉价的次表面散射近似的效果如图 [materialClothSubsurface] 所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_cloth_subsurface.png" alt="图 [materialClothSubsurface]: 白色织物（左列）与带有棕色次表面散射的白色织物（右列）对比"/></div>

我们完整的布料 BRDF 实现（包括光泽颜色和可选的次表面散射）可在代码清单 [clothFullBRDF] 中找到。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// specular BRDF
float D = distributionCloth(roughness, NoH);
float V = visibilityCloth(NoV, NoL);
vec3  F = sheenColor;
vec3 Fr = (D * V) * F;

// diffuse BRDF
float diffuse = diffuse(roughness, NoV, NoL, LoH);
#if defined(MATERIAL_HAS_SUBSURFACE_COLOR)
// energy conservative wrap diffuse
diffuse *= saturate((dot(n, light.l) + 0.5) / 2.25);
#endif
vec3 Fd = diffuse * pixel.diffuseColor;

#if defined(MATERIAL_HAS_SUBSURFACE_COLOR)
// cheap subsurface scatter
Fd *= saturate(subsurfaceColor + NoL);
vec3 color = Fd + Fr * NoL;
color *= (lightIntensity * lightAttenuation) * lightColor;
#else
vec3 color = Fd + Fr;
color *= (lightIntensity * lightAttenuation * NoL) * lightColor;
#endif
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clothFullBRDF]: 在 GLSL 中实现我们的布料 BRDF]

### 布料参数化

布料材质模型包含了为标准材质模型先前定义的所有参数，但 **_metallic_** 和 **_reflectance_** 除外。表 [clothParameters] 中描述的两个额外参数也可用。


       Parameter      |      Definition
---------------------:|:---------------------
**SheenColor**        | 高光色调，用于创建双色调高光织物（默认为 0.04，与标准反射率一致）
**SubsurfaceColor**   | 光线经过材质散射和吸收后，漫反射颜色的色调
[Table [clothParameters]: 布料模型参数]


要创建类似天鹅绒的材质，可将基色设置为黑色（或深色）。色度信息应设置在光泽颜色上。要创建更常见的织物（如牛仔布、棉布等），可使用基色来表示色度，并使用默认光泽颜色或将光泽颜色设置为基色的亮度。

# 光照

光照环境的正确性和一致性对于实现逼真的视觉效果至关重要。在调研了现有的渲染引擎（如 Unity 或 Unreal Engine 4）以及传统的实时渲染文献后，很明显一致性很少能够实现。
以Unreal Engine为例，它允许美术师以流明（光功率单位）为单位指定点光源的"亮度"。然而，方向光的亮度却使用一种任意的无名单位来表示。为了匹配一个光功率为5,000流明的点光源的亮度，美术师必须使用亮度为10的方向光。这种不匹配使得美术师在添加、移除或修改光源时，难以保持场景的视觉一致性。

仅使用任意单位是一种连贯的解决方案，但这使得重用照明设置变得困难。例如，一个室外场景会使用亮度为10的方向光作为太阳，而所有其他光源都将相对于该值进行定义。将这些光源移到室内环境中会使它们变得过亮。

因此，我们的目标是让所有照明默认情况下就是正确的，同时给予美术师足够的自由度来实现所需的视觉效果。我们将支持多种光源，分为两大类：直接照明和间接照明：

**直接照明**：点光源、光度学光源、面光源。

**间接照明**：基于图像的照明（IBL），包括局部[^localProbesMobile]和远处光照探针。

[^localProbesMobile]: 在移动设备上支持局部光照探针可能成本过高，我们将首先专注于设置在无限远处的远距离光照探针。

## 单位

以下章节将讨论如何实现各种类型的光源，并且所提出的方程使用了表 [lightUnits] 中总结的不同符号和单位。

| 光度学术语 | 符号 | 单位 |
|-----------:|:----:|:----|
| 光功率 | $\Phi$ | 流明 ($lm$) |
| 发光强度 | $I$ | 坎德拉 ($cd$) 或 $\frac{lm}{sr}$ |
| 照度 | $E$ | 勒克斯 ($lx$) 或 $\frac{lm}{m^2}$ |
| 亮度 | $L$ | 尼特 ($nt$) 或 $\frac{cd}{m^2}$ |
| 辐射功率 | $\Phi_e$ | 瓦特 ($W$) |
| 光视效能 | $\eta$ | 流明每瓦 ($\frac{lm}{W}$) |
| 光视效率 | $V$ | 百分比 (%) |
[表 [lightUnits]: 光度学单位]

为了实现完全一致的照明，我们必须使用能够反映真实场景中各种光强度之间比例的光单位。这些强度的变化范围很大，从家用灯泡的大约800 $lm$，到白天天空和太阳照明的120,000 $lx$。

实现照明一致性最简单的方法是采用物理光单位。这将实现照明设置的完全可重用性。使用物理光单位还允许我们使用基于物理的相机。

表 [lightTypesUnits] 显示了我们要支持的每种光源类型所对应的光单位。

| 光源类型 | 单位 |
|-----------:|:-----|
| 方向光 | 照度 ($lx$ 或 $\frac{lm}{m^2}$) |
| 点光源 | 光功率 ($lm$) |
| 聚光灯 | 光功率 ($lm$) |
| 光度学光源 | 发光强度 ($cd$) |
| 掩码光度学光源 | 光功率 ($lm$) |
| 面光源 | 光功率 ($lm$) |
| 基于图像的光源 | 亮度 ($\frac{cd}{m^2}$) |
[表 [lightTypesUnits]: 每种光源类型的强度单位]

**关于辐射功率单位的说明**

尽管市面上销售的灯泡通常在其包装上用流明来标示亮度，但人们通常习惯于通过灯泡所需的能量（瓦特）来指代其亮度。瓦特数仅表示灯泡消耗多少能量，并不表示其亮度。现在更节能的灯泡（卤素灯、LED等）已广泛可用，理解这一区别变得更为重要。

然而，由于美术师可能习惯于通过功率来估算光源的亮度，因此我们应允许用户使用功率单位来定义光源的亮度。转换方法如公式 $\ref{radiantPowerToLuminousPower}$ 所示。

$$\begin{equation}\label{radiantPowerToLuminousPower}
\Phi = \Phi_e \eta
\end{equation}$$

在公式 $\ref{radiantPowerToLuminousPower}$ 中，$\eta$ 是光源的光视效能，单位为流明每瓦。已知[最大可能的光视效能](http://en.wikipedia.org/wiki/Luminous_efficacy)为683 $\frac{lm}{W}$，我们也可以使用光视效率 $V$（也称为光视系数），如公式 $\ref{radiantPowerLuminousEfficiency}$ 所示。

$$\begin{equation}\label{radiantPowerLuminousEfficiency}
\Phi = \Phi_e 683 \times V
\end{equation}$$

表 [lightTypesEfficacy] 可用作参考，通过使用各类光源的光视效能或光视效率将瓦特转换为流明。更具体的数值可参见维基百科的[光视效能](http://en.wikipedia.org/wiki/Luminous_efficacy)页面。

| 光源类型 | 光视效能 $\eta$ | 光视效率 $V$ |
|-----------:|:----------------:|:-------------:|
| 白炽灯 | 14-35 | 2-5% |
| LED | 28-100 | 4-15% |
| 荧光灯 | 60-100 | 9-15% |
[表 [lightTypesEfficacy]: 各类光源的光视效能和光视效率]

### 光单位验证

使用物理光单位的一大优势是能够对我们的方程进行物理验证。我们可以使用专用设备来测量三种光单位。

#### 照度

到达表面的照度可以使用入射光测量仪进行测量。在我们的测试中，我们使用的是 [Sekonic L-478D](http://www.sekonic.com/products/l-478d/overview.aspx)，如图 [sekonic] 所示。

入射光测量仪使用一个白色漫射半球罩来捕捉到达表面的照度。根据所需测量的不同，正确朝向半球罩的方向非常重要。例如，在晴朗明亮的日子里，将半球罩垂直于太阳方向所得到的结果与将半球罩水平放置所得到的结果会大相径庭。

<div align="center"><img src="https://google.github.io/filament/images/photo_light_meter.jpg"/></div>

#### 亮度

表面上的亮度，即入射光与表面的乘积，可以使用亮度计（通常也称为点测光表）进行测量。入射光测量仪使用漫射半球从所有方向捕捉光线，而点测光表则使用遮光罩从单一方向测量入射光。在我们的测试中，我们使用 [Sekonic 5度取景器](http://www.sekonic.com/products/l-478dr/accessories/np-finder-5-degree-for-l-478.aspx)，它可以替换 L-478D 上的漫射器，以测量5度锥角内的亮度。

<div align="center"><img src="https://google.github.io/filament/images/photo_incident_light_meter.jpg"/></div>

#### 发光强度

光源的发光强度无法直接测量，但如果知道测量设备与光源之间的距离，则可以从测得的照度推导出来。公式 $\ref{derivedLuminousIntensity}$ 是第 [Punctual lights] 节讨论的平方反比定律的一个简单应用。

$$\begin{equation}\label{derivedLuminousIntensity}
I = E \cdot d^2
\end{equation}$$

## 直接照明

我们在上一节中已经为渲染器支持的所有光源类型定义了光单位，但尚未为照明方程的结果定义光单位。选择物理光单位意味着我们将在着色器中计算亮度值，因此我们所有的光照评估函数都将计算任意给定点的亮度 $L_{out}$（或出射辐射率）。亮度取决于照度 $E$ 和 BSDF $f(v,l)$：

$$\begin{equation}\label{luminanceEquation}
L_{out} = f(v,l)E
\end{equation}$$

### 方向光

方向光的主要目的是重建室外环境中的重要光源，即太阳和/或月亮。虽然方向光在物理世界中并不真实存在，但任何距离光受体足够远的光源都可以假定为方向光（即所有入射光线都是平行的，如图 [directionalLight] 所示）。

<div align="center"><img src="https://google.github.io/filament/images/diagram_directional_light.png"/></div>

这种近似对于表面的漫反射响应效果非常好，但镜面反射响应却是不正确的。Frostbite 引擎通过将"太阳"方向光视为一个圆盘面光源来解决这个问题。然而，我们的测试表明，这种质量提升并不足以证明增加的计算成本是合理的。

我们之前提到，我们为方向光选择了照度单位（$lx$）。部分原因是我们能够轻松找到天空和太阳的照度值（在线或通过测光表），也是因为这样可以简化 $\ref{luminanceEquation}$ 中描述的亮度方程。

$$\begin{equation}\label{directionalLuminanceEquation}
L_{out} = f(v,l) E_{\bot} \left< \NoL \right>
\end{equation}$$

在简化的亮度方程 $\ref{directionalLuminanceEquation}$ 中，$E_{\bot}$ 是光源在垂直于该光源的表面的照度。如果方向光模拟太阳，则 $E_{\bot}$ 是太阳方向垂直于太阳方向的表面的照度。

表 [sunSkyIlluminance] 提供了太阳和天空照度的有用参考值，这些值是在加州三月的一个晴天测量得到的[^illuminanceMeasures]。

| 光源 | 上午10点 | 中午12点 | 下午5:30 |
|------------------:|---------:|---------:|---------:|
| $Sky_{\bot} + Sun_{\bot}$ | 120,000 | 130,000 | 90,000 |
| $Sky_{\bot}$ | 20,000 | 25,000 | 9,000 |
| $Sun_{\bot}$ | 100,000 | 105,000 | 81,000 |
[表 [sunSkyIlluminance]: 照度值，单位为 $lx$（满月的照度为 1 $lx$）]

动态方向光在运行时评估特别廉价，如代码清单 [glslDirectionalLight] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 l = normalize(-lightDirection);
float NoL = clamp(dot(n, l), 0.0, 1.0);

// lightIntensity is the illuminance
// at perpendicular incidence in lux
float illuminance = lightIntensity * NoL;
vec3 luminance = BSDF(v, l) * illuminance;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [glslDirectionalLight]: 在 GLSL 中实现方向光]

图 [directionalLightTest] 展示了使用设置为近似正午太阳的方向光（照度设置为110,000 $lx$）照亮一个简单场景的效果。为便于说明，仅显示直接照明。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_directional_light.png"/></div>

[^illuminanceMeasures]: 使用入射光测量仪（Sekonic L-478D）进行的测量

### 点光源

我们的引擎将支持两种类型的点光源，这在大多数（即使不是全部）渲染引擎中都很常见：点光源和聚光灯。这些类型的光源传统上在两个方面是不物理准确的：

1. 它们确实是点状的，且无限小。
2. 它们不遵循[平方反比定律](http://en.wikipedia.org/wiki/Inverse-square_law)。

第一个问题可以通过面光源来解决，但鉴于点光源的成本较低，在可能的情况下使用无限小的点光源被认为是实用的。

第二个问题很容易修复。对于给定的点光源，感知强度与观察者（更准确地说，是光受体）距离的平方成反比递减。

对于遵循平方反比定律的点光源，公式 $\ref{luminanceEquation}$ 中的项 $E$ 表示为公式 $\ref{punctualLightEquation}$ 中的形式，其中 $d$ 是从表面上一点到光源的距离。

$$\begin{equation}\label{punctualLightEquation}
E = L_{in} \left< \NoL \right> = \frac{I}{d^2} \left< \NoL \right>
\end{equation}$$

点光源和聚光灯之间的区别在于 $E$ 的计算方式，特别在于发光强度 $I$ 如何从光功率 $\Phi$ 计算得出。

#### 点光源

点光源仅由空间中的一个位置定义，如图 [pointLight] 所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_point_light.png"/></div>

点光源的光功率通过对发光强度在光源立体角上进行积分来计算，如公式 $\ref{pointLightLuminousPower}$ 所示。然后可以轻松地从光功率推导出发光强度。

$$\begin{equation}\label{pointLightLuminousPower}
\Phi = \int_{\Omega} I dl = \int_{0}^{2\pi} \int_{0}^{\pi} I d\theta d\phi = 4 \pi I \\
I = \frac{\Phi}{4 \pi}
\end{equation}$$

通过将 $\ref{punctualLightEquation}$ 中的 $I$ 和 $\ref{luminanceEquation}$ 中的 $E$ 进行简单替换，我们可以将点光源的亮度方程表示为光功率的函数（见 $\ref{pointLightLuminanceEquation}$）。

$$\begin{equation}\label{pointLightLuminanceEquation}
L_{out} = f(v,l) \frac{\Phi}{4 \pi d^2} \left< \NoL \right>
\end{equation}$$

图 [pointLightTest] 展示了使用受距离衰减影响的点光源照亮一个简单场景的效果。为便于说明，光线衰减效果被夸张显示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_point_light.png"/></div>

#### 聚光灯

聚光灯由空间中的一个位置、一个方向向量和两个锥角 $ \theta_{inner} $ 和 $ \theta_{outer} $ 定义（见图 [spotLight]）。这两个角度用于定义聚光灯的角度衰减衰减。因此，聚光灯的光照评估函数必须同时考虑平方反比定律和这两个角度，以正确评估亮度衰减。

<div align="center"><img src="https://google.github.io/filament/images/diagram_spot_light.png"/></div>

公式 $\ref{spotLightLuminousPower}$ 描述了如何以类似于点光源的方式计算聚光灯的光功率，其中使用 $ \theta_{outer} $ 作为聚光灯锥体的外角，范围在 [0..$\pi$] 内。

$$\begin{equation}\label{spotLightLuminousPower}
\Phi = \int_{\Omega} I dl = \int_{0}^{2\pi} \int_{0}^{\theta_{outer}} I d\theta d\phi = 2 \pi (1 - cos\frac{\theta_{outer}}{2})I \\
I = \frac{\Phi}{2 \pi (1 - cos\frac{\theta_{outer}}{2})}
\end{equation}$$

虽然这种公式在物理上是正确的，但它使得聚光灯有些难以使用：改变锥体的外角会改变照明水平。图 [spotLightTestFocused] 展示了由聚光灯照亮的同一场景，外角分别为55度和15度。观察随着锥孔径减小，照明水平如何增加。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_spot_light_focused.png"/></div>

照明和外锥角的耦合意味着美术师无法在不改变感知照明的情况下调整聚光灯的影响锥体。因此，为美术师提供一个参数来禁用这种耦合是有意义的。公式 $\ref{spotLightLuminousPowerB}$ 展示了为此目的如何构造光功率。

$$\begin{equation}\label{spotLightLuminousPowerB}
\Phi = \pi I \\
I = \frac{\Phi}{\pi} \\
\end{equation}$$

使用这种新的发光强度计算公式，图 [spotLightTest] 中的测试场景在两个锥孔径下都表现出相似的照明水平。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_spot_light.png"/></div>

如果将聚光灯的反射器替换为一个完美吸收光线的哑光漫射遮罩，这种新的公式也可以被认为是基于物理的。

聚光灯的评估函数可以用两种方式表示：

- **带光吸收器**
  $$\begin{equation}\label{spotAbsorber}
  L_{out} = f(v,l) \frac{\Phi}{\pi d^2} \left< \NoL \right> \lambda(l)
  \end{equation}$$
- **带光反射器**
  $$\begin{equation}\label{spotReflector}
  L_{out} = f(v,l) \frac{\Phi}{2 \pi (1 - cos\frac{\theta_{outer}}{2}) d^2} \left< \NoL \right> \lambda(l)
  \end{equation}$$

公式 $\ref{spotAbsorber}$ 和 $\ref{spotReflector}$ 中的项 $ \lambda(l) $ 是聚光灯的角度衰减因子，描述如下公式 $\ref{spotAngleAtt}$。

$$\begin{equation}\label{spotAngleAtt}
\lambda(l) = \frac{l \cdot spotDirection - cos\theta_{outer}}{cos\theta_{inner} - cos\theta_{outer}}
\end{equation}$$

#### 衰减函数

正确评估平方反比定律的衰减因子对于基于物理的点光源是强制性的。简单的数学公式对于实际实现来说却是不实用的：

1. 距离的平方除法在物体与光源相交或"接触"时可能导致除以0。

2. 每个光源的影响球体是无限的（$ \frac{I}{d^2} $ 是渐近的，永远不会达到0），这意味着为了正确着色一个像素，我们需要评估世界中的每一个光源。

第一个问题可以通过设定点光源并非真正的点光源，而是小面光源的假设来轻松解决。为此，我们可以简单地将点光源视为半径为1厘米的球体，如公式 $\ref{finitePunctualLight}$ 所示。

$$\begin{equation}\label{finitePunctualLight}
E = \frac{I}{max(d^2, {0.01}^2)}
\end{equation}$$

我们可以通过为每个光源引入一个影响半径来解决第二个问题。这种解决方案有几个优点。工具可以快速向美术师展示世界的哪些部分将受到每个光源的影响（工具只需在每个光源周围绘制一个球体）。渲染引擎可以利用这一额外信息更积极地进行光源剔除，而美术师/开发人员可以通过手动调整光源的影响半径来辅助引擎。

从数学上讲，光源的照度应在影响半径定义的边界处平滑地达到零。[#Karis13b] 提出了以一种使光源的大部分影响不受影响的方式对平方反比函数进行加窗处理。所提出的加窗方法如公式 $\ref{attenuationWindowing}$ 所述，其中 $r$ 是光源的影响半径。

$$\begin{equation}\label{attenuationWindowing}
E = \frac{I}{max(d^2, {0.01}^2)} \left< 1 - \frac{d^4}{r^4} \right>^2
\end{equation}$$

代码清单 [glslPunctualLight] 演示了如何在 GLSL 中实现基于物理的点光源。请注意，这段代码中使用的光强是发光强度 $I$，单位为 $cd$，是从 CPU 端的光功率转换而来的。这段代码片段未进行优化，一些计算可以卸载到 CPU 上（例如光源的平方反比衰减半径的平方，或聚光灯的缩放比例和角度）。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float getSquareFalloffAttenuation(vec3 posToLight, float lightInvRadius) {
    float distanceSquare = dot(posToLight, posToLight);
    float factor = distanceSquare * lightInvRadius * lightInvRadius;
    float smoothFactor = max(1.0 - factor * factor, 0.0);
    return (smoothFactor * smoothFactor) / max(distanceSquare, 1e-4);
}

float getSpotAngleAttenuation(vec3 l, vec3 lightDir,
        float innerAngle, float outerAngle) {
    // the scale and offset computations can be done CPU-side
    float cosOuter = cos(outerAngle);
    float spotScale = 1.0 / max(cos(innerAngle) - cosOuter, 1e-4)
    float spotOffset = -cosOuter * spotScale

    float cd = dot(normalize(-lightDir), l);
    float attenuation = clamp(cd * spotScale + spotOffset, 0.0, 1.0);
    return attenuation * attenuation;
}

vec3 evaluatePunctualLight() {
    vec3 l = normalize(posToLight);
    float NoL = clamp(dot(n, l), 0.0, 1.0);
    vec3 posToLight = lightPosition - worldPosition;

    float attenuation;
    attenuation  = getSquareFalloffAttenuation(posToLight, lightInvRadius);
    attenuation *= getSpotAngleAttenuation(l, lightDir, innerAngle, outerAngle);

    vec3 luminance = (BSDF(v, l) * lightIntensity * attenuation * NoL) * lightColor;
    return luminance;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [glslPunctualLight]: 在 GLSL 中实现点光源]

### 光度学光源

点光源是一种非常实用且高效的场景照明方式，但无法给美术师提供足够的光分布控制。建筑照明设计领域专注于设计照明系统以满足人类需求，需要考虑：

- 提供的光量
- 光的颜色
- 光在空间内的分布

我们迄今描述的照明系统可以轻松解决前两个问题，但我们需要一种方法来定义光在空间内的分布。光分布对于室内场景或某些类型的室外场景乃至道路照明尤为重要。图 [lightDistributionTest] 展示了由美术师控制光分布的场景。这种类型的分布控制广泛用于展示物体时（例如博物馆、商店或画廊）。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_photometric_lights.png"/></div>

光度学光源使用光度学分布文件来描述其强度分布。有两种常用的格式：IES（照明工程学会）和 EULUMDAT（欧洲流明数据格式），但我们将专注于前一种格式。IES 分布文件受到许多工具和引擎的支持，例如 Unreal Engine 4、Frostbite、Renderman、Maya 和 Killzone。此外，灯泡和灯具制造商通常提供 IES 光分布文件下载（例如 Philips 提供了[大量的 IES 文件](http://www.usa.lighting.philips.com/connect/tools_literature/photometric_data_1.wpd)）。当测量灯具或照明装置（其中光源被部分遮挡）时，光度学分布文件特别有用。灯具会阻挡在某些方向发射的光，从而塑造光的分布。

<div align="center"><img src="https://google.github.io/filament/images/photo_photometric_lights.jpg"/></div>

IES 分布文件存储了在被测光源周围球体上各个角度的发光强度。这种球坐标系通常被称为光度学网络，可以使用专门工具（如 [IESviewer](http://www.photometricviewer.com/)）进行可视化。下面的图 [xarrow] 展示了 Pixar [为 Renderman 提供的](http://renderman.pixar.com/view/DP25764) XArrow IES 分布文件的光度学网络。该图还展示了我们的工具 `lightgen` 在 3D 空间中对 XArrow IES 分布文件的渲染。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_xarrow.png"/></div>

IES 格式的文档记录很少，而且在网络上找到的文件之间存在语法差异的情况并不少见。理解 IES 分布文件的最佳资源是 Ian Ashdown 的"解析 IESNA LM-63 光度数据文件"文档 [#Ashdown98]。简而言之，IES 分布文件以坎德拉为单位存储光源周围各角度上的发光强度。对于每个测量的水平角度，提供了一系列不同垂直角度上的发光强度。然而，被测光源在水平方向上对称是相当常见的。上面展示的 XArrow 分布文件就是一个很好的例子：强度随垂直角度（垂直轴）变化，但在水平轴上是对称的。IES 分布文件中垂直角度的范围是0到180度，水平角度的范围是0到360度。

图 [lightenSamples] 展示了 Pixar 为 Renderman 提供的一系列 IES 分布文件，使用我们的 `lightgen` 工具渲染。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_lightgen_samples.png"/></div>

IES 分布文件可以直接应用于任何点光源（点或聚光）。为此，我们必须首先处理 IES 分布文件并生成一个光度学分布文件作为纹理。出于性能考虑，我们生成的光度学分布文件是一个1D纹理，表示特定垂直角度下所有水平角度的平均发光强度（即，每个像素代表一个垂直角度）。要真正表示一个光度学光源，我们应该使用2D纹理，但由于大多数光源在水平面上是完全或大部分对称的，我们可以接受这种近似。纹理中存储的值通过 IES 分布文件中定义的最大强度的倒数进行归一化。这使我们能够轻松地将纹理存储在任何浮点格式中，或者以牺牲一点精度为代价，存储在亮度8位纹理中（例如灰度PNG）。存储归一化值还允许我们将光度学分布文件视为一种掩码：

光度学分布文件作为掩码
:   发光强度由美术师通过设置光源的光功率来定义，与其他点光源一样。美术师定义的强度除以从 IES 分布文件计算出的光源强度。IES 分布文件包含发光强度，但它仅对裸灯泡有效，而测量的强度值则考虑了灯具。为了测量灯具（而非灯泡）的强度，我们使用分布文件中的强度对单位球体执行蒙特卡洛积分[^xarrowIntensity]。

光度学分布文件
:   发光强度来自分布文件本身。从1D纹理中采样的所有值只需乘以最大强度。为方便起见，我们还提供一个乘数。

光度学分布文件可以在渲染时作为一个简单的衰减因子应用。亮度方程 $\ref{photometricLightEvaluation}$ 描述了光度学点光源的评估函数。

$$\begin{equation}\label{photometricLightEvaluation}
L_{out} = f(v,l) \frac{I}{d^2} \left< \NoL \right> \Psi(l)
\end{equation}$$

项 $ \Psi(l) $ 是光度学衰减函数。它取决于光向量，也取决于光源的方向。聚光灯已经拥有一个方向向量，但我们也需要为光度学点光源引入一个方向向量。

光度学衰减函数可以通过在点光实现（代码清单 [glslPunctualLight]）中添加一个新的衰减因子，轻松地在 GLSL 中实现。修改后的实现如代码清单 [glslPhotometricPunctualLight] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float getPhotometricAttenuation(vec3 posToLight, vec3 lightDir) {
    float cosTheta = dot(-posToLight, lightDir);
    float angle = acos(cosTheta) * (1.0 / PI);
    return texture2DLodEXT(lightProfileMap, vec2(angle, 0.0), 0.0).r;
}

vec3 evaluatePunctualLight() {
    vec3 l = normalize(posToLight);
    float NoL = clamp(dot(n, l), 0.0, 1.0);
    vec3 posToLight = lightPosition - worldPosition;

    float attenuation;
    attenuation  = getSquareFalloffAttenuation(posToLight, lightInvRadius);
    attenuation *= getSpotAngleAttenuation(l, lightDirection, innerAngle, outerAngle);
    attenuation *= getPhotometricAttenuation(l, lightDirection);

    float luminance = (BSDF(v, l) * lightIntensity * attenuation * NoL) * lightColor;
    return luminance;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [glslPhotometricPunctualLight]: 在 GLSL 中实现来自光度学分布文件的衰减]

光源强度在 CPU 端计算（代码清单 [photometricLightIntensity]），并且取决于光度学分布文件是作为掩码使用还是直接使用。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float multiplier;
// Photometric profile used as a mask
if (photometricLight.isMasked()) {
    // The desired intensity is set by the artist
    // The integrated intensity comes from a Monte-Carlo
    // integration over the unit sphere around the luminaire
    multiplier = photometricLight.getDesiredIntensity() /
            photometricLight.getIntegratedIntensity();
} else {
    // Multiplier provided for convenience, set to 1.0 by default
    multiplier = photometricLight.getMultiplier();
}

// The max intensity in cd comes from the IES profile
float lightIntensity = photometricLight.getMaxIntensity() * multiplier;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [photometricLightIntensity]: 在 CPU 上计算光度学光源的强度]

[^xarrowIntensity]: XArrow 分布文件声明了一个1,750 lm的发光强度，但蒙特卡洛积分显示其强度仅为350 lm。

### 面光源

[TODO]

### 光源参数化

与标准材质模型的参数化类似，我们的目标是使光源参数化对美术师和开发人员都直观且易于使用。基于这一理念，我们决定将光的颜色（或色调）与光的强度分开。因此，光的颜色将定义为线性 RGB 颜色（为了方便，在工具 UI 中为 sRGB）。

光源参数的完整列表如表 [lightParameters] 所示。

| 参数 | 定义 |
|------------------:|:---------------------|
| **类型** | 方向光、点光源、聚光灯或面光源 |
| **方向** | 用于方向光、聚光灯、光度学点光源以及线性和管状面光源（方向） |
| **颜色** | 发射光的颜色，为线性 RGB 颜色。在工具中可以指定为 sRGB 颜色或色温 |
| **强度** | 光源的亮度。单位取决于光源类型 |
| **衰减半径** | 最大影响距离 |
| **内角** | 聚光灯内锥角，以度为单位 |
| **外角** | 聚光灯外锥角，以度为单位 |
| **长度** | 面光源的长度，用于创建线性或管状光源 |
| **半径** | 面光源的半径，用于创建球面或管状光源 |
| **光度学分布文件** | 表示光度学光源分布文件的纹理，仅对点光源有效 |
| **掩码分布文件** | 布尔值，指示 IES 分布文件是否作为掩码使用。作为掩码使用时，光源的亮度将乘以用户指定强度与集成后的 IES 分布文件强度之间的比值。不作为掩码使用时，用户指定的强度被忽略，转而使用 IES 乘数 |
| **光度学乘数** | 光度学光源的亮度乘数（如果 IES 作为掩码关闭） |
[表 [lightParameters]: 光源类型参数]

**注意**：为简化实现，所有光功率在发送到着色器之前都将转换为发光强度（$cd$）。此转换取决于光源类型，并在前面的章节中进行了说明。

**注意**：光源类型可以从其他参数推断出来（例如，点光源的长度、半径、内角和外角均为0）。

#### 色温

然而，真实世界的人造光源通常由其色温来定义，以开尔文（K）为单位。光源的色温是一个理想黑体辐射体在辐射出与光源色调相当的光时所处的温度。为方便起见，工具应允许美术师以色温的形式指定光源的色调（有意义的范围是1,000 K到12,500 K）。

为了从色温计算 RGB 值，我们可以使用普朗克轨迹，如图 [planckianLocus] 所示。这条轨迹是炽热黑体的颜色在色度空间中随温度变化而变化的路径。

<div align="center"><img src="https://google.github.io/filament/images/diagram_planckian_locus.png"/></div>

从这条轨迹计算 RGB 值的最简单方法是使用 [#Krystek85] 中描述的公式。Krystek 算法（公式 $\ref{krystek}$）在 CIE 1960（UCS）空间中工作，使用以下公式，其中 $T$ 是所需温度，$u$ 和 $v$ 是 UCS 空间中的坐标。

$$\begin{equation}\label{krystek}
u(T) = \frac{0.860117757 + 1.54118254 \times 10^{-4}T + 1.28641212 \times 10^{-7}T^2}{1 + 8.42420235 \times 10^{-4}T + 7.08145163 \times 10^{-7}T^2} \\
v(T) = \frac{0.317398726 + 4.22806245
 \times 10^{-5}T + 4.20481691 \times 10^{-8}T^2}{1 - 2.89741816
 \times 10^{-5}T + 1.61456053 \times 10^{-7}T^2}
\end{equation}$$

该近似在范围1,000K至15,000K内的精度约为 $ 9 \times 10^{-5} $。从 CIE 1960 空间，我们可以使用公式 $\ref{cieToxyY}$ 中的公式计算 xyY 空间（CIES 1931）中的坐标。

$$\begin{equation}\label{cieToxyY}
x = \frac{3u}{2u - 8v + 4} \\
y = \frac{2v}{2u - 8v + 4}
\end{equation}$$

上述公式适用于黑体色温，因此也适用于标准照明体的相关色温。如果我们希望计算 D 系列标准 CIE 照明体的精确色度坐标，可以使用公式 $\ref{seriesDtoxyY}$。

$$\begin{equation}\label{seriesDtoxyY}
x = \begin{cases} 0.244063 + 0.09911 \frac{10^3}{T} + 2.9678 \frac{10^6}{T^2} - 4.6070 \frac{10^9}{T^3} & 4,000K \le T \le 7,000K \\
0.237040 + 0.24748 \frac{10^3}{T} + 1.9018 \frac{10^6}{T^2} - 2.0064 \frac{10^9}{T^3} & 7,000K \le T \le 25,000K \end{cases} \\
y = -3x^2 + 2.87 x - 0.275
\end{equation}$$

从 xyY 空间，我们可以转换到 CIE XYZ 空间（公式 $\ref{xyYtoXYZ}$）。

$$\begin{equation}\label{xyYtoXYZ}
X = \frac{xY}{y} \\
Z = \frac{(1 - x - y)Y}{y}
\end{equation}$$

对于我们的需求，我们将固定 $Y = 1$。这使我们能够通过一个简单的3x3矩阵从 XYZ 空间转换到线性 RGB 空间，如公式 $\ref{XYZtoRGB}$ 所示。

$$\begin{equation}\label{XYZtoRGB}
\left[ \begin{matrix} R \\ G \\ B \end{matrix} \right] = M^{-1} \left[ \begin{matrix} X \\ Y \\ Z \end{matrix} \right]
\end{equation}$$

变换矩阵 M 根据目标 RGB 颜色空间的原色计算得出。公式 $\ref{XYZtoRGBValues}$ 展示了使用 sRGB 颜色空间的逆矩阵进行转换。

$$\begin{equation}\label{XYZtoRGBValues}
\left[ \begin{matrix} R \\ G \\ B \end{matrix} \right] = \left[ \begin{matrix} 3.2404542 & -1.5371385 & -0.4985314 \\ -0.9692660 & 1.8760108 & 0.0415560 \\ 0.0556434 & -0.2040259 & 1.0572252 \end{matrix} \right] \left[ \begin{matrix} X \\ Y \\ Z \end{matrix} \right]
\end{equation}$$

这些操作的结果是 sRGB 颜色空间中的一个线性 RGB 三元组。由于我们关心结果的色度，我们必须应用一个归一化步骤，以避免钳位大于1.0的值并扭曲结果颜色：

$$\begin{equation}\label{normalizedRGB}
\hat{C}_{linear} = \frac{C_{linear}}{max(C_{linear})}
\end{equation}$$

我们最终必须应用 sRGB 光电转换函数（OECF，如公式 $\ref{OECFsRGB}$ 所示）来获得一个可显示的值（如果传递给渲染器进行着色，该值应保持线性）。

$$\begin{equation}\label{OECFsRGB}
C_{sRGB} = \begin{cases} 12.92 \times \hat{C}_{linear} & \hat{C}_{linear} \le 0.0031308 \\
1.055 \times \hat{C}_{linear}^{\frac{1}{2.4}} - 0.055 & \hat{C}_{linear} \gt 0.0031308 \end{cases}
\end{equation}$$

为方便起见，图 [colorTemperatureScaleCCT] 显示了从1,000K到12,500K的相关色温范围。下面使用的所有颜色均假设 CIE $ D_{65} $ 为白点（正如 sRGB 颜色空间中的情况）。

<div align="center"><img src="https://google.github.io/filament/images/diagram_color_temperature_cct.png"/></div>

类似地，图 [colorTemperatureScaleCIE] 显示了从1,000K到12,500K的 D 系列 CIE 标准照明体范围。

<div align="center"><img src="https://google.github.io/filament/images/diagram_color_temperature_cie.png"/></div>

作为参考，图 [colorTemperatureScaleCCTClamped] 显示了未经公式 $\ref{normalizedRGB}$ 中归一化步骤的相关色温范围。

<div align="center"><img src="https://google.github.io/filament/images/diagram_color_temperature_cct_clamped.png"/></div>

表 [colorTemperatureSamples] 以 sRGB 色样形式展示了各种常见光源的相关色温。这些颜色是相对于 $ D_{65} $ 白点的，因此其感知色调可能会根据显示器的白点而变化。更多信息请参见 [太阳是什么颜色？](http://jila.colorado.edu/~ajsh/colour/Tspectrum.html)。

| 温度 (K) | 光源 | 颜色 |
|------------------:|:-----------------------------|:-------------------------------------------------------|
| 1,700-1,800 | 火柴火焰 | <div style="background-color: #ff7d00; width: 60px">&nbsp;</div> |
| 1,850-1,930 | 蜡烛火焰 | <div style="background-color: #ff8701; width: 60px">&nbsp;</div> |
| 2,000-3,000 | 日出/日落时的太阳 | <div style="background-color: #ffa64c; width: 60px">&nbsp;</div> |
| 2,500-2,900 | 家用钨丝灯泡 | <div style="background-color: #ffb05e; width: 60px">&nbsp;</div> |
| 3,000 | 钨丝灯1K | <div style="background-color: #ffb86f; width: 60px">&nbsp;</div> |
| 3,200-3,500 | 石英灯 | <div style="background-color: #ffbf7b; width: 60px">&nbsp;</div> |
| 3,200-3,700 | 荧光灯 | <div style="background-color: #ffbf7b; width: 60px">&nbsp;</div> |
| 3,275 | 钨丝灯2K | <div style="background-color: #ffc180; width: 60px">&nbsp;</div> |
| 3,380 | 钨丝灯5K, 10K | <div style="background-color: #ffc486; width: 60px">&nbsp;</div> |
| 5,000-5,400 | 正午太阳 | <div style="background-color: #ffe9d7; width: 60px">&nbsp;</div> |
| 5,500-6,500 | 日光（太阳+天空） | <div style="background-color: #fff3f1; width: 60px">&nbsp;</div> |
| 5,500-6,500 | 穿过云层/薄雾的太阳 | <div style="background-color: #fff3f1; width: 60px">&nbsp;</div> |
| 6,000-7,500 | 阴天天空 | <div style="background-color: #faf6ff; width: 60px">&nbsp;</div> |
| 6,500 | RGB 显示器白点 | <div style="background-color: #fff8fe; width: 60px">&nbsp;</div> |
| 7,000-8,000 | 室外阴影区域 | <div style="background-color: #ebecff; width: 60px">&nbsp;</div> |
| 8,000-10,000 | 多云天空 | <div style="background-color: #d6e0ff; width: 60px">&nbsp;</div> |
[表 [colorTemperatureSamples]: 常见光源的归一化相关色温]

### 预曝光光源

基于物理的渲染和物理光单位带来了一个有趣的挑战：如何存储和处理照明代码产生的大范围值？假设着色器中使用全精度进行计算，我们仍然希望能够将光照通道的线性输出存储在合理大小的缓冲区中（`RGB16F` 或等效格式）。实现这一目标最明显且最简单的方法是在写入光照通道结果之前，简单地应用相机曝光（更多信息请参见基于物理的相机部分）。这个简单的步骤如代码清单 [preexposedLighting] 所示：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
fragColor = luminance * camera.exposure;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [preexposedLighting]: 光照通道的输出被预曝光，以适应半浮点缓冲区]

这种解决方案解决了存储问题，但要求中间计算使用单精度浮点数进行。我们更希望使用半精度浮点数来执行所有（或至少大部分）光照计算。这样做可以极大地提高性能并降低功耗，特别是在移动设备上。然而，半精度浮点数不适合此类工作，因为常见的照度和亮度值（例如太阳的）可能超出其范围。解决方案是简单地对光源本身进行预曝光，而不是对光照通道的结果进行预曝光。如果在更新光源的常量缓冲区时成本较低，这可以在 CPU 上高效完成。也可以在 GPU 上完成，如代码清单 [preexposedLights] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// The inputs must be highp/single precision,
// both for range (intensity) and precision (exposure)
// The output is mediump/half precision
float computePreExposedIntensity(highp float intensity, highp float exposure) {
    return intensity * exposure;
}

Light getPointLight(uint index) {
    Light light;
    uint lightIndex = // fetch light index;

    // the intensity must be highp/single precision
    highp vec4 colorIntensity  = lightsUniforms.lights[lightIndex][1];

    // pre-expose the light
    light.colorIntensity.w = computePreExposedIntensity(
            colorIntensity.w, frameUniforms.exposure);

    return light;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [preexposedLights]: 对光源进行预曝光，使整个着色管线能够使用半精度浮点数]

在实践中，我们对以下光源进行预曝光：
- 点光源（点光和聚光）：在 GPU 上
- 方向光：在 CPU 上
- IBL：在 CPU 上
- 材质自发光：在 GPU 上
## 基于图像的光照

在真实世界中，光线从各个方向射来，要么直接来自光源，要么在环境中经过物体反射后被部分吸收。从某种意义上说，物体周围的整个环境都可以被视为一个光源。图像，特别是立方体贴图，是编码这种"环境光"的绝佳方式。这被称为基于图像的光照（Image Based Lighting, IBL），有时也称为间接光照（Indirect Lighting）。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_ball_ibl.png"/></div>

基于图像的光照存在一些局限性。显然，环境图像必须以某种方式获取，并且如下文所述，在使用前需要经过预处理。通常情况下，环境图像是在真实世界中离线获取的，或者由引擎在离线或运行时生成；无论采用哪种方式，都需要使用局部或远距离探测器（probes）。

这些探测器可用于获取远距离或局部环境。在本文档中，我们专注于远距离环境探测器，此时假设光线来自无穷远（这意味着物体表面上的每个点都使用相同的环境贴图）。

整个环境都对物体表面上的某个给定点贡献光照，这被称为**辐照度**（irradiance, $E$）。从物体表面反射出去的光被称为**出射辐射度**（radiance, $L_{out}$）。入射光照必须一致地应用于BRDF的漫反射和镜面反射部分。

由基于图像的光照（IBL）的辐照度与材质模型（BRDF）$f(\Theta)$[^ibl1]相互作用产生的出射辐射度 $L_{out}$ 按以下公式计算：

$$\begin{equation}
L_{out}(n, v, \Theta) = \int_\Omega  f(l, v, \Theta) L_{\bot}(l) \left< \NoL \right> dl 
\end{equation}$$ 

需要注意的是，这里我们考察的是表面在**宏观**层面的行为（不要与微观层面的方程混淆），因此它仅依赖于 $\vec n$ 和 $\vec v$。本质上，我们是在将BRDF应用于来自各个方向并编码在IBL中的"点光源"。

### IBL类型 ###

现代渲染引擎中使用四种常见的IBL类型：

- **远距离光照探测器（Distant light probes）**，用于捕捉"无穷远"处的光照信息，此时视差可以忽略。远距离探测器通常包含天空、远处的景观特征或建筑物等。它们由引擎捕获或从相机获取为高动态范围图像（HDRI）。

- **局部光照探测器（Local light probes）**，用于从特定视角捕捉世界的某个区域。根据周围几何体的不同，捕捉结果投影到立方体或球体上。局部探测器比远距离探测器更精确，特别适用于为材质添加局部反射。

- **平面反射（Planar reflections）**，通过将场景沿一个平面镜像渲染来捕捉反射。此技术仅适用于平坦表面，如建筑物地面、道路和水面。

- **屏幕空间反射（Screen space reflection, SSR）**，基于已渲染的场景（例如利用前一帧）通过深度缓冲中的光线步进（ray-marching）来捕捉反射。SSR效果出色，但计算成本可能非常高。

此外，我们必须区分静态IBL和动态IBL。例如，实现完全动态的昼夜循环需要动态地重新计算远距离光照探测器[^iblTypes1]。平面反射和屏幕空间反射本质上都是动态的。

### IBL单位 ###

正如之前在直接光照部分所讨论的，我们所有的光源必须使用物理单位。因此，我们的IBL将使用亮度单位 $\frac{cd}{m^2}$，这也是我们所有直接光照方程的输出单位。对于由引擎（动态或静态离线）捕获的光照探测器，使用亮度单位是直接的。

然而，高动态范围图像的处理则更为微妙。相机记录的不是测量的亮度，而是一个依赖于设备的数值，该数值仅与原始场景亮度_相关_。因此，我们必须为美术人员提供一个乘数，使他们能够恢复，或至少是近似地恢复原始的绝对亮度。

要正确重建IBL的HDRI亮度，美术人员需要做的不仅仅是拍摄环境照片和记录额外信息：

- **颜色校准**：使用灰卡或[MacBeth ColorChecker](http://en.wikipedia.org/wiki/ColorChecker)

- **相机设置**：光圈、快门和ISO

- **亮度采样**：使用点测光/亮度计

[TODO] 测量并列出常见的亮度值（晴朗天空、室内等）

### 处理光照探测器 ###

我们之前已经看到，IBL的出射辐射度是通过在表面的半球上积分来计算的。由于这显然在实时渲染中成本过高，我们必须首先对光照探测器进行预处理，将其转换为更适合实时交互的格式。

以下各节将讨论用于加速光照探测器评估的技术：

- **镜面反射（Specular reflectance）**：预滤波重要性采样和拆分和近似（split-sum approximation）

- **漫反射（Diffuse reflectance）**：辐照度贴图和球谐函数

### 远距离光照探测器 ###

#### 漫反射BRDF积分 ####

使用朗伯BRDF[^iblDiffuse1]，我们得到出射辐射度：

$$
\begin{align*}
   f_d(\sigma) &= \frac{\sigma}{\pi} \\
L_d(n, \sigma) &= \int_{\Omega} f_d(\sigma) L_{\bot}(l) \left< \NoL \right> dl \\
               &= \frac{\sigma}{\pi} \int_{\Omega} L_{\bot}(l) \left< \NoL \right> dl \\
               &= \frac{\sigma}{\pi} E_d(n) \quad \text{其中辐照度} \; 
        E_d(n) = \int_{\Omega} L_{\bot}(l) \left< \NoL \right> dl
\end{align*}
$$

或者在离散域中：

$$ E_d(n) \equiv \sum_{\forall \, i \in image} L_{\bot}(s_i) \left< n \cdot s_i \right> \Omega_s $$

$\Omega_s$ 是与采样点 $i$ 相关联的立体角[^iblDiffuse2]。

辐照度积分 $\Ed$ 可以简单（尽管较慢[^iblDiffuse3]）地预先计算并存储到立方体贴图中，以便在运行时高效访问。通常，*image*（图像）是一个立方体贴图或等距柱状投影图。项 $ \frac{\sigma}{\pi} $ 独立于IBL，在运行时添加以得到_出射辐射度_。

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m0.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_irradiance.png"/></div>


[^ibl1]: $\Theta$ 表示材质模型 $f$ 的参数，即：粗糙度、反照率等...

[^iblTypes1]: 这可以通过静态探测器的混合或随时间分摊工作负载来实现

[^iblDiffuse1]: 朗伯BRDF不依赖于 $\vec l$、$\vec v$ 或 $\theta$，因此 $L_d(n,v,\theta) \equiv L_d(n,\sigma)$

[^iblDiffuse2]: 对于立方体贴图，$\Omega_s$ 可近似为 $\frac{2\pi}{6 \cdot width \cdot height}$

[^iblDiffuse3]: $O(12\,n^2\,m^2)$，其中 $n$ 和 $m$ 分别是环境和预计算立方体贴图的尺寸


然而，辐照度也可以通过球谐函数（SH，在球谐函数部分中有更详细的描述）分解来非常近似地逼近，并在运行时廉价地计算。在移动设备上，通常最好避免纹理采样，以释放一个纹理单元。即使将辐照度存储到立方体贴图中，使用SH分解后再进行渲染来预计算积分也要快几个数量级。

SH分解在概念上类似于傅里叶变换，它在频域中将信号表示在一个正交基上。我们最感兴趣的特性是：

- 编码 $\cosTheta$ 只需要很少的系数

- 与具有_圆对称性_的核进行卷积非常廉价，在SH空间中变为乘积形式

实际上，对于 $\cosTheta$，只需要4个或9个系数（即：2个或3个波段）就足够了，这意味着我们也不需要更多的系数来表示 $\Lt$。

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_irradiance_sh3.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_irradiance_sh2.png"/></div>


在实践中，我们预先将 $\Lt$ 与 $\cosTheta$ 进行卷积，并预先将这些系数乘以基缩放因子 $K_l^m$，以使着色器中的重建代码尽可能简单：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 irradianceSH(vec3 n) {
    // uniform vec3 sphericalHarmonics[9]
    // We can use only the first 2 bands for better performance
    return
          sphericalHarmonics[0]
        + sphericalHarmonics[1] * (n.y)
        + sphericalHarmonics[2] * (n.z)
        + sphericalHarmonics[3] * (n.x)
        + sphericalHarmonics[4] * (n.y * n.x)
        + sphericalHarmonics[5] * (n.y * n.z)
        + sphericalHarmonics[6] * (3.0 * n.z * n.z - 1.0)
        + sphericalHarmonics[7] * (n.z * n.x)
        + sphericalHarmonics[8] * (n.x * n.x - n.y * n.y);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [irradianceSH]: 从预缩放SH系数重建辐照度的GLSL代码]

注意，使用2个波段时，上述计算变为一个简单的 $4 \times 4$ 矩阵乘以向量。

此外，由于预先乘以了 $K_l^m$，SH系数可以被视为颜色，特别是 `sphericalHarmonics[0]` 直接就是平均辐照度。


#### 镜面BRDF积分 ####

如上所述，由IBL的辐照度与BRDF相互作用产生的出射辐射度 $\Lout$ 为：

$$\begin{equation}\label{specularBRDFIntegration}
\Lout(n, v, \Theta) = \int_\Omega f(l, v, \Theta) \Lt(l) \left< \NoL \right> \partial l 
\end{equation}$$ 

我们认识到这是 $\Lt$ 与 $f(l, v, \Theta) \left< \NoL \right>$ 的卷积，即：环境以BRDF作为核被*滤波*。实际上，在较高的粗糙度下，镜面反射看起来更*模糊*。

将 $f$ 的表达式代入方程 $\ref{specularBRDFIntegration}$，我们得到：

$$\begin{equation}
\Lout(n,v,\Theta) = \int_\Omega D(l, v, \alpha) F(l, v, f_0, f_{90}) V(l, v, \alpha) \left< \NoL \right> \Lt(l) \partial l
\end{equation}$$ 

这个表达式在积分内部依赖于 $v$、$\alpha$、$f_0$ 和 $f_{90}$，这使得其评估极其昂贵，不适合在移动设备上实时计算（即使使用预滤波重要性采样）。

##### 简化BRDF积分 #####

由于不存在闭式解或简便的方法来计算 $\Lout$ 积分，我们使用一个简化的方程来代替：$\hat{I}$，其中我们假设 $v = n$，即视线方向 $v$ 始终等于表面法线 $n$。显然，这个假设将破坏卷积的所有视角相关效应，例如靠近观察者的反射中增加的模糊（即所谓的拉伸反射（stretchy reflections））。

这种简化也会对恒定环境（如白色熔炉测试）产生严重影响，因为它会影响结果的常数（即DC）项的大小。我们至少可以通过在我们的简化积分中使用缩放因子 $K$ 来对此进行校正，该因子在选择适当时能确保平均辐照度保持正确。

- $I$ 是我们的原始积分，即：$I(g) = \int_\Omega g(l) \left< \NoL \right> \partial l$
- $\hat{I}$ 是简化后的积分，其中 $v = n$
- $K$ 是一个缩放因子，确保平均辐照度在 $\hat{I}$ 下不变
- $\tilde{I}$ 是我们对 $I$ 的最终近似，$\tilde{I} = \hat{I} \times K$


由于 $I$ 是一个积分，乘法可以分配到其上，即：$I(g()f()) = I(g())I(f())$。

在此基础上，

$$\begin{equation}
I( f(\Theta) \Lt ) \approx \tilde{I}( f(\Theta) \Lt )                       \\
\tilde{I}( f(\Theta) \Lt ) = K \times \hat{I}( f(\Theta) \Lt )              \\
K = \frac{I(f(\Theta))}{\hat{I}(f(\Theta))}
\end{equation}$$ 


从上式可以看出，当 $\Lt$ 为常数时，$\tilde{I}$ 等价于 $I$，并产生正确的结果：

$$\begin{align*}
\tilde{I}(f(\Theta)\Lt^{constant}) &= \Lt^{constant} \hat{I}(f(\Theta)) \frac{I(f(\Theta))}{\hat{I}(f(\Theta))} \\
                                   &= \Lt^{constant} I(f(\Theta))                                               \\
                                   &= I(f(\Theta)\Lt^{constant})
\end{align*}$$


类似地，我们也可以证明当 $v = n$ 时结果是正确的，因为此时 $I = \hat{I}$：

$$\begin{align*}
\tilde{I}(f(\Theta)\Lt) &= I(f(\Theta)\Lt) \frac{I(f(\Theta))}{I(f(\Theta))}    \\
                        &= I(f(\Theta)\Lt)
\end{align*}$$


最后，我们可以通过将 $\Lt = \bar{\Lt} + (\Lt - \bar{\Lt}) = \bar{\Lt} + \Delta\Lt$ 代入 $\tilde{I}$ 来证明缩放因子 $K$ 满足我们的平均辐照度（$\bar{\Lt}$）要求：

$$\begin{align*}
\tilde{I}(f(\Theta)\Lt) &= \tilde{I}\left[f\left(\Theta\right) \left(\bar{\Lt} + \Delta\Lt\right)\right] \\
                        &= K \times \hat{I}\left[f\left(\Theta\right) \left(\bar{\Lt} + \Delta\Lt\right)\right] \\
                        &= K \times \left[\hat{I}\left(f\left(\Theta\right)\bar{\Lt}\right) + \hat{I}\left(f\left(\Theta\right)\Delta\Lt\right)\right] \\ 
                        &= K \times \hat{I}\left(f\left(\Theta\right)\bar{\Lt}\right) + K \times \hat{I}\left(f\left(\Theta\right) \Delta\Lt\right) \\
                        &= \tilde{I}\left(f\left(\Theta\right)\bar{\Lt}\right) + \tilde{I}\left(f\left(\Theta\right) \Delta\Lt\right) \\
                        &= I\left(f\left(\Theta\right)\bar{\Lt}\right) + \tilde{I}\left(f\left(\Theta\right) \Delta\Lt\right)
\end{align*}$$

上述结果表明平均辐照度被正确计算，即：$I(f(\Theta)\bar{\Lt})$。

理解这种近似的一种方式是，它将出射辐射度 $\Lt$ 分为两部分：平均值 $\bar{\Lt}$ 和相对于平均值的偏差 $\Delta\Lt$，并对平均值部分进行正确积分，然后加上偏差部分的简化积分：

$$\begin{equation}
approximation(\Lt) = correct(\bar{\Lt}) + simplified(\Lt - \bar{\Lt})
\end{equation}$$ 



现在，让我们来看每一项：

$$\begin{equation}\label{iblPartialEquations}
\hat{I}(f(n, \alpha) \Lt) = \int_\Omega f(l, n, \alpha) \Lt(l) \left< \NoL \right> \partial l   \\
\hat{I}(f(n, \alpha))     = \int_\Omega f(l, n, \alpha)        \left< \NoL \right> \partial l   \\
I(f(n, v, \alpha))        = \int_\Omega f(l, n, v, \alpha)     \left< \NoL \right> \partial l
\end{equation}$$


以上三个方程都可以轻松地预先计算并存储在查找表中，如下所述。


##### 离散域 #####

在离散域中，\ref{iblPartialEquations} 中的方程变为：

$$\begin{equation}
\hat{I}(f(n, \alpha) \Lt) \equiv \frac{1}{N}\sum_{\forall \, i \in image} f(l_i, n, \alpha) \Lt(l_i) \left<\NoL\right>  \\
\hat{I}(f(n, \alpha))     \equiv \frac{1}{N}\sum_{\forall \, i \in image} f(l_i, n, \alpha)          \left<\NoL\right>  \\
I(f(n, v, \alpha))        \equiv \frac{1}{N}\sum_{\forall \, i \in image} f(l_i, n, v, \alpha)       \left<\NoL\right>
\end{equation}$$

然而，在实践中我们使用的是_重要性采样_，这需要考虑分布的 $pdf$，并增加一项 $\frac{4\left<\VoH\right>}{D(h_i, \alpha)\left<\NoH\right>}$。请参阅"IBL的重要性采样"部分：

$$\begin{equation}\label{iblImportanceSampling}
\hat{I}(f(n, \alpha) \Lt) \equiv \frac{4}{N}\sum_i^N f(l_i, n, \alpha)    \frac{\left<\VoH\right>}{D(h_i, \alpha)\left<\NoH\right>} \Lt(l_i) \left<\NoL\right>  \\
\hat{I}(f(n, \alpha))     \equiv \frac{4}{N}\sum_i^N f(l_i, n, \alpha)    \frac{\left<\VoH\right>}{D(h_i, \alpha)\left<\NoH\right>}          \left<\NoL\right>  \\
I(f(n, v, \alpha))        \equiv \frac{4}{N}\sum_i^N f(l_i, n, v, \alpha) \frac{\left<\VoH\right>}{D(h_i, \alpha)\left<\NoH\right>}          \left<\NoL\right>
\end{equation}$$


回顾对于 $\hat{I}$，我们假设 $v = n$，方程 \ref{iblImportanceSampling} 简化为：

$$\begin{equation}
\hat{I}(f(n, \alpha) \Lt) \equiv \frac{4}{N}\sum_i^N \frac{f(l_i, n,    \alpha)}{D(h_i, \alpha)} \Lt(l_i) \left<\NoL\right>  \\
\hat{I}(f(n, \alpha))     \equiv \frac{4}{N}\sum_i^N \frac{f(l_i, n,    \alpha)}{D(h_i, \alpha)}          \left<\NoL\right>  \\
I(f(n, v, \alpha))        \equiv \frac{4}{N}\sum_i^N \frac{f(l_i, n, v, \alpha)}{D(h_i, \alpha)} \frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right>
\end{equation}$$

然后，前两个方程可以合并，使得 $LD(n, \alpha) = \frac{\hat{I}(f(n, \alpha) \Lt)}{\hat{I}(f(n, \alpha))}$

$$\begin{equation}\label{iblLD}
LD(n, \alpha)       \equiv \frac{\sum_i^N \frac{f(l_i, n, \alpha)}{D(h_i, \alpha)} \Lt(l_i) \left<\NoL\right>}{\sum_i^N \frac{f(l_i, n, \alpha)}{D(h_i, \alpha)}\left<\NoL\right>}
\end{equation}$$
$$\begin{equation}\label{iblDFV}
I(f(n, v, \alpha))  \equiv \frac{4}{N}\sum_i^N \frac{f(l_i, n, v, \alpha)}{D(h_i, \alpha)} \frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right>
\end{equation}$$

请注意，此时我们几乎可以离线计算剩余的两个方程。唯一的困难在于，当我们预计算这些积分时，我们不知道 $f_0$ 或 $f_{90}$。下面我们将看到，对于方程 \ref{iblDFV}，我们可以在运行时将这些项合并进去，但遗憾的是，这对于方程 \ref{iblLD} 是不可能的，我们必须假设 $f_0 = f_{90} = 1$（即：菲涅耳项始终取值为1）。

我们还必须处理BRDF的可见性项（visibility term），在实践中保留它会导致与ground truth相比略差的结果，因此我们也设 $V = 1$。

让我们将 $f$ 代入方程 \ref{iblLD} 和 \ref{iblDFV}：

$$\begin{equation}
f(l_i, n, \alpha) = D(h_i, \alpha)F(f_0, f_{90}, \left<\VoH\right>)V(l_i, v, \alpha)
\end{equation}$$

第一个简化是，BRDF中的 $D(h_i, \alpha)$ 项与分母（来自重要性采样的 $pdf$）相抵消，而F和V由于我们假设其值为1而消失。

$$\begin{equation}
LD(n, \alpha)       \equiv \frac{\sum_i^N V(l_i, v, \alpha)\left<\NoL\right>\Lt(l_i) }{\sum_i^N \left<\NoL\right>}
\end{equation}$$
$$\begin{equation}\label{iblFV}
I(f(n, v, \alpha))  \equiv \frac{4}{N}\sum_i^N \color{green}{F(f_0, f_{90}, \left<\VoH\right>)} V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right>
\end{equation}$$

现在，让我们将菲涅耳项代入方程 \ref{iblFV}：

$$\begin{equation}
F(f_0, f_{90}, \left<\VoH\right>) = f_0 (1 - F_c(\left<\VoH\right>)) + f_{90} F_c(\left<\VoH\right>) \\
F_c(\left<\VoH\right>) = (1 - \left<\VoH\right>)^5
\end{equation}$$


$$\begin{equation}
I(f(n, v, \alpha))  \equiv \frac{4}{N}\sum_i^N \left[\color{green}{f_0 (1 - F_c(\left<\VoH\right>)) + f_{90} F_c(\left<\VoH\right>)}\right] V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
\end{equation}$$

$$
\begin{align*}
I(f(n, v, \alpha))  \equiv & \color{green}{f_0   } \frac{4}{N}\sum_i^N  \color{green}{(1 - F_c(\left<\VoH\right>))} V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
                    +      & \color{green}{f_{90}} \frac{4}{N}\sum_i^N  \color{green}{     F_c(\left<\VoH\right>) } V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right>
\end{align*}
$$


最后，我们提取可以离线计算的方程（即：不依赖于运行时参数 $f_0$ 和 $f_{90}$ 的部分）：

$$\begin{equation}\label{iblAllEquations}
DFG_1(\alpha, \left<\NoV\right>) = \frac{4}{N}\sum_i^N  \color{green}{(1 - F_c(\left<\VoH\right>))} V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
DFG_2(\alpha, \left<\NoV\right>) = \frac{4}{N}\sum_i^N  \color{green}{     F_c(\left<\VoH\right>) } V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
I(f(n, v, \alpha))  \equiv   \color{green}{f_0} \color{red}{DFG_1(\alpha, \left<\NoV\right>)} + \color{green}{f_{90}} \color{red}{DFG_2(\alpha, \left<\NoV\right>)}
\end{equation}$$


注意，$DFG_1$ 和 $DFG_2$ 仅依赖于 $\NoV$，即法线 $n$ 与视线方向 $v$ 之间的夹角。这是因为积分相对于 $n$ 是对称的。在积分时，我们可以选择任意满足 $\NoV$ 的 $v$（例如：在计算 $\VoH$ 时）。


将所有内容重新组合：

$$
\begin{align*}
\Lout(n,v,\alpha,f_0,f_{90})     &\simeq \big[ f_0 \color{red}{DFG_1(\NoV, \alpha)} + f_{90} \color{red}{DFG_2(\NoV, \alpha)} \big] \times LD(n, \alpha) \\
DFG_1(\alpha, \left<\NoV\right>) &=      \frac{4}{N}\sum_i^N  \color{green}{(1 - F_c(\left<\VoH\right>))} V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
DFG_2(\alpha, \left<\NoV\right>) &=      \frac{4}{N}\sum_i^N  \color{green}{     F_c(\left<\VoH\right>) } V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
LD(n, \alpha)                    &=      \frac{\sum_i^N V(l_i, n, \alpha)\left<\NoL\right>\Lt(l_i) }{\sum_i^N \left<\NoL\right>}
\end{align*}     
$$

#### $DFG_1$ 和 $DFG_2$ 项的可视化 ####

$DFG_1$ 和 $DFG_2$ 都可以预先计算到由 $(\NoV, \alpha)$ 索引的常规2D纹理中并进行双线性采样，也可以在运行时使用这些曲面的解析近似来计算。参见附录中的示例代码。预计算纹理如表[textureDFG]所示。预计算的C++实现可以在[Precomputing L for image-based lighting]部分找到。

$DFG_1$                  | $DFG_2$                  | ${ DFG_1, DFG_2, 0 }$
-------------------------|--------------------------|----------------------
<div align="center"><img src="https://google.github.io/filament/images/ibl/dfg1.png"/></div> | <div align="center"><img src="https://google.github.io/filament/images/ibl/dfg2.png"/></div> | <div align="center"><img src="https://google.github.io/filament/images/ibl/dfg.png"/></div>
[Table [textureDFG]: Y轴: $\alpha$。X轴: $cos \theta$]


$DFG_1$ 和 $DFG_2$ 恰好位于 $[0, 1]$ 范围内，但8位纹理没有足够的精度，会导致问题。不幸的是，在移动设备上，16位或浮点纹理并不普及，而且采样器数量有限。尽管使用纹理的着色器代码具有简洁性，但使用解析近似可能更好。不过请注意，由于我们只需要存储两个项，OpenGL ES 3.0的RG16F纹理格式是一个不错的选择。

这种解析近似在[#Karis14]中有所描述，其本身基于[#Lazarov13]。[#Narkowicz14]是另一个有趣的近似。请注意，这两种近似与[Pre-integration for multiscattering]部分中介绍的能量补偿项不兼容。表[textureApproxDFG]展示了这些近似的可视化表示。

$DFG_1$                         | $DFG_2$                         | ${ DFG_1, DFG_2, 0 }$
--------------------------------|---------------------------------|----------------------
<div align="center"><img src="https://google.github.io/filament/images/ibl/dfg1_approx.png"/></div> | <div align="center"><img src="https://google.github.io/filament/images/ibl/dfg2_approx.png"/></div> | <div align="center"><img src="https://google.github.io/filament/images/ibl/dfg_approx.png"/></div>
[Table [textureApproxDFG]: Y轴: $\alpha$。X轴: $cos \theta$]


#### $LD$ 项的可视化 ####

$LD$ 是环境与仅依赖于 $\alpha$ 参数（其本身与粗糙度相关，参见[Roughness remapping and clamping]部分）的函数的卷积。$LD$ 可以方便地存储在具有mipmap的立方体贴图中，其中递增的LOD级别存储了以递增粗糙度预滤波的环境。这很有效，因为这种卷积是一种强大的低通滤波器。为了充分利用每个mipmap级别，有必要对 $\alpha$ 进行重映射；我们发现使用 $\gamma = 2$ 的幂次重映射效果良好且方便。

$$
\begin{align*}
    \alpha       &= perceptualRoughness^2                        \\
    lod_{\alpha} &= \alpha^{\frac{1}{2}} = perceptualRoughness   \\
\end{align*}
$$

示例如下：


<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m0.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m1.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m2.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m3.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_river_roughness_m4.png"/></div>

#### 间接镜面反射和间接漫反射分量的可视化 ####

图[iblVisualized]展示了间接光照如何与电介质和导体相互作用。为便于说明，去除了直接光照。

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_visualization.jpg"/></div>

#### IBL评估实现 ####

清单[iblEvaluation]展示了一个用于评估IBL的GLSL实现，使用了前面各节描述的各种纹理。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 ibl(vec3 n, vec3 v, vec3 diffuseColor, vec3 f0, vec3 f90,
        float perceptualRoughness) {
    vec3 r = reflect(n);
    vec3 Ld = textureCube(irradianceEnvMap, r) * diffuseColor;
    float lod = computeLODFromRoughness(perceptualRoughness);
    vec3 Lld = textureCube(prefilteredEnvMap, r, lod);
    vec2 Ldfg = textureLod(dfgLut, vec2(dot(n, v), perceptualRoughness), 0.0).xy;
    vec3 Lr =  (f0 * Ldfg.x + f90 * Ldfg.y) * Lld;
    return Ld + Lr;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [iblEvaluation]: 基于图像光照评估的GLSL实现]

然而，我们可以使用球谐函数代替辐照度立方体贴图，以及使用 $DFG$ LUT的解析近似，来节省几次纹理查找，如清单[optimizedIblEvaluation]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 irradianceSH(vec3 n) {
    // uniform vec3 sphericalHarmonics[9]
    // We can use only the first 2 bands for better performance
    return
          sphericalHarmonics[0]
        + sphericalHarmonics[1] * (n.y)
        + sphericalHarmonics[2] * (n.z)
        + sphericalHarmonics[3] * (n.x)
        + sphericalHarmonics[4] * (n.y * n.x)
        + sphericalHarmonics[5] * (n.y * n.z)
        + sphericalHarmonics[6] * (3.0 * n.z * n.z - 1.0)
        + sphericalHarmonics[7] * (n.z * n.x)
        + sphericalHarmonics[8] * (n.x * n.x - n.y * n.y);
}

// NOTE: this is the DFG LUT implementation of the function above
vec2 prefilteredDFG_LUT(float coord, float NoV) {
    // coord = sqrt(roughness), which is the mapping used by the
    // IBL prefiltering code when computing the mipmaps
    return textureLod(dfgLut, vec2(NoV, coord), 0.0).rg;
}

vec3 evaluateSpecularIBL(vec3 r, float perceptualRoughness) {
    // This assumes a 256x256 cubemap, with 9 mip levels
    float lod = 8.0 * perceptualRoughness;
    // decodeEnvironmentMap() either decodes RGBM or is a no-op if the
    // cubemap is stored in a float texture
    return decodeEnvironmentMap(textureCubeLodEXT(environmentMap, r, lod));
}

vec3 evaluateIBL(vec3 n, vec3 v, vec3 diffuseColor, vec3 f0, vec3 f90, float perceptualRoughness) {
    float NoV = max(dot(n, v), 0.0);
    vec3 r = reflect(-v, n);

    // Specular indirect
    vec3 indirectSpecular = evaluateSpecularIBL(r, perceptualRoughness);
    vec2 env = prefilteredDFG_LUT(perceptualRoughness, NoV);
    vec3 specularColor = f0 * env.x + f90 * env.y;

    // Diffuse indirect
    // We multiply by the Lambertian BRDF to compute radiance from irradiance
    // With the Disney BRDF we would have to remove the Fresnel term that
    // depends on NoL (it would be rolled into the SH). The Lambertian BRDF
    // can be baked directly in the SH to save a multiplication here
    vec3 indirectDiffuse = max(irradianceSH(n), 0.0) * Fd_Lambert();

    // Indirect contribution
    return diffuseColor * indirectDiffuse + indirectSpecular * specularColor;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [optimizedIblEvaluation]: 基于图像光照评估的GLSL实现]


#### 多重散射的预积分 ####

在[Energy loss in specular reflectance]部分中，我们讨论了如何使用第二个缩放的镜面反射瓣来补偿由于BRDF中仅考虑单次散射事件而导致的能量损失。这个能量补偿瓣由一个依赖于 $r$ 的项进行缩放，$r$ 定义如下：

$$\begin{equation}
r = \int_{\Omega} D(l,v) V(l,v) \left< \NoL \right> \partial l
\end{equation}$$

或者，使用重要性采样进行评估（参见"IBL的重要性采样"部分）：

$$\begin{equation}
r \equiv  \frac{4}{N}\sum_i^N  V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right>
\end{equation}$$

这个等式与方程 $\ref{iblAllEquations}$ 中的 $DFG_1$ 和 $DFG_2$ 项非常相似。实际上，除了没有菲涅耳项外，它们是相同的。

通过进一步假设 $f_{90} = 1$，我们可以重写 $DFG_1$、$DFG_2$ 以及 $\Lout$ 的重建：

$$
\begin{align*}
\Lout(n,v,\alpha,f_0)                           &\simeq \big[ (1 - f_0) \color{red}{DFG_1^{multiscatter}(\NoV, \alpha)} + f_0 \color{red}{DFG_2^{multiscatter}(\NoV, \alpha)} \big] \times LD(n, \alpha) \\
DFG_1^{multiscatter}(\alpha, \left<\NoV\right>) &=      \frac{4}{N}\sum_i^N  \color{green}{F_c(\left<\VoH\right>)} V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
DFG_2^{multiscatter}(\alpha, \left<\NoV\right>) &=      \frac{4}{N}\sum_i^N                                        V(l_i, v, \alpha)\frac{\left<\VoH\right>}{\left<\NoH\right>} \left<\NoL\right> \\
LD(n, \alpha)                                   &=      \frac{\sum_i^N V(l_i, n, \alpha)\left<\NoL\right>\Lt(l_i) }{\sum_i^N V(l_i, n, \alpha)\left<\NoL\right>}
\end{align*}     
$$

这两个新的 $DFG$ 项只需替换[Precomputing L for image-based lighting]部分所示实现中使用的项：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float Fc = pow(1 - VoH, 5.0f);
r.x += Gv * Fc;
r.y += Gv;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [multiscatterIBLPreintegration]: 多重散射 $L_{DFG}$ 项的C++实现]

为了执行重建，我们需要稍微修改清单[multiscatterIBLEvaluation]：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec2 dfg = textureLod(dfgLut, vec2(dot(n, v), perceptualRoughness), 0.0).xy;
// (1 - f0) * dfg.x + f0 * dfg.y
vec3 specularColor = mix(dfg.xxx, dfg.yyy, f0);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [multiscatterIBLEvaluation]: 使用多重散射LUT的基于图像光照评估的GLSL实现]


#### 总结 ####

为了计算远距离基于图像的光照的镜面反射贡献，我们不得不做了一些近似和折衷：

- $v = n$，这是在积分IBL的非恒定部分时导致最大误差的假设。这导致粗糙度各向异性相对于视点完全丢失。

- IBL非恒定部分的粗糙度贡献被量化，并使用三线性滤波在这些级别之间插值。这在低粗糙度时最为明显（例如，对于一个9级LOD的立方体贴图，约为0.0625）。

- 由于mipmap级别被用来存储预积分后的环境，它们无法按其本意用于纹理缩小。这可能会在高频区域或环境中在低粗糙度和/或远处或小物体上引起走样或摩尔纹伪影。这也可能由于导致较差的缓存访问模式而影响性能。

- IBL非恒定部分没有菲涅耳效应。

- IBL非恒定部分的可见性设为1。

- Schlick菲涅耳近似

- 在多重散射情况下 $f_{90} = 1$。


<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_prefilter_vs_reference.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_stretchy_reflections_error.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_trilinear_0.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_trilinear_1.png"/></div>

<div align="center"><img src="https://google.github.io/filament/images/ibl/ibl_no_mipmaping.png"/></div>


### 清漆层 ###

在对IBL进行采样时，清漆层（clear coat）作为第二个镜面反射瓣进行计算。这个镜面反射瓣沿视线方向定向，因为我们无法合理地积分整个半球。清单[clearCoatIBL]展示了这种近似的实际应用。它还展示了能量守恒步骤。需要注意的是，第二个镜面反射瓣的计算方式与主镜面瓣完全相同，使用相同的DFG近似。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// clearCoat_NoV == shading_NoV if the clear coat layer doesn't have its own normal map
float Fc = F_Schlick(0.04, 1.0, clearCoat_NoV) * clearCoat;
// base layer attenuation for energy compensation
iblDiffuse  *= 1.0 - Fc;
iblSpecular *= sq(1.0 - Fc);
iblSpecular += specularIBL(r, clearCoatPerceptualRoughness) * Fc;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clearCoatIBL]: 基于图像光照中清漆层镜面瓣的GLSL实现]

### 各向异性 ###

[#McAuley15] 描述了一种称为"弯曲反射向量"（bent reflection vector）的技术，基于[#Revie12]。弯曲反射向量是各向异性光照的一种粗略近似，但替代方案是使用重要性采样。这种近似计算成本足够低，并且提供了良好的结果，如图[anisotropicIBL1]和图[anisotropicIBL2]所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_anisotropic_ibl1.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_anisotropic_ibl2.jpg"/></div>

该技术的实现非常直接，如清单[bentReflectionVector]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 anisotropicTangent = cross(bitangent, v);
vec3 anisotropicNormal = cross(anisotropicTangent, bitangent);
vec3 bentNormal = normalize(mix(n, anisotropicNormal, anisotropy));
vec3 r = reflect(-v, bentNormal);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [bentReflectionVector]: 弯曲反射向量的GLSL实现]

通过接受负的 `anisotropy` 值，这种技术可以变得更加有用，如清单[bentReflectionVectorDirection]所示。当各向异性为负值时，高光不在切线的方向上，而是在副切线（bitangent）的方向上。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 anisotropicDirection = anisotropy >= 0.0 ? bitangent : tangent;
vec3 anisotropicTangent = cross(anisotropicDirection, v);
vec3 anisotropicNormal = cross(anisotropicTangent, anisotropicDirection);
vec3 bentNormal = normalize(mix(n, anisotropicNormal, anisotropy));
vec3 r = reflect(-v, bentNormal);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [bentReflectionVectorDirection]: 弯曲反射向量的GLSL实现]

图[anisotropicDirection]展示了这种修改后实现的实际效果。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_anisotropy_direction.png"/></div>

### 次表面散射 ###

[TODO] 解释次表面散射和IBL

### 织物 ###

织物材质模型的IBL实现比其他材质模型更为复杂。主要区别在于使用了不同的NDF（"Charlie" NDF vs 高度相关的Smith GGX）。如本节所述，在计算IBL时，我们使用拆分和近似来计算BRDF的DFG项。这个DFG项是为不同的BRDF设计的，不能用于织物BRDF。由于我们设计的织物BRDF不需要菲涅耳项，我们可以在DFG LUT的第三个通道中生成一个单一的DG项。结果如图[dfgClothLUT]所示。

DG项使用[#Estevez17]中推荐的均匀采样生成。使用均匀采样时，$pdf$ 简单地是 $\frac{1}{2\pi}$，并且我们仍然必须使用雅可比行列式 $\frac{1}{4\left< \VoH \right>}$。

<div align="center"><img src="https://google.github.io/filament/images/ibl/dfg_cloth.png"/></div>

基于图像光照实现的其余部分遵循与常规光照相同的步骤，包括可选的次表面散射项及其包裹漫反射分量。正如清漆层IBL实现一样，我们无法对半球进行积分，而是使用视线方向作为主导光方向来计算包裹漫反射分量。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float diffuse = Fd_Lambert() * ambientOcclusion;
#if defined(SHADING_MODEL_CLOTH)
#if defined(MATERIAL_HAS_SUBSURFACE_COLOR)
diffuse *= saturate((NoV + 0.5) / 2.25);
#endif
#endif

vec3 indirectDiffuse = irradianceIBL(n) * diffuse;
#if defined(SHADING_MODEL_CLOTH) && defined(MATERIAL_HAS_SUBSURFACE_COLOR)
indirectDiffuse *= saturate(subsurfaceColor + NoV);
#endif

vec3 ibl = diffuseColor * indirectDiffuse + indirectSpecular * specularColor;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [clothApprox]: 织物NDF的DFG近似GLSL实现]

需要注意的是，这只解决了IBL问题的一部分。前面描述的预滤波镜面反射环境贴图是使用标准着色模型的BRDF进行卷积的，这与织物BRDF不同。为了获得准确的结果，理论上我们应该为引擎中使用的每个BRDF提供一组IBL。然而，为我们的使用场景提供第二组IBL并不实际，因此我们决定依赖现有的IBL。

## 静态光照

[TODO] 球谐函数或球面高斯光照贴图、辐照度体（irradiance volumes）、PRT等...

## 透明与半透明光照

透明和半透明材质对于增加场景的真实感和正确性至关重要。因此，Filament必须为这两种材质类型提供光照模型，以使美术人员能够恰当地重建逼真的场景。半透明也可以有效地用于一些非真实感设置中。

### 透明度

要正确地对透明表面进行光照，我们首先必须理解材质的不透明度是如何应用的。观察一扇窗户，你会发现漫反射是透明的。另一方面，镜面反射越亮，窗户看起来就越不透明。这种效果可以在图[cameraTransparency]中看到：场景被正确地反射到玻璃表面上，但太阳的镜面高光足够亮，看起来是不透明的。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_camera_transparency.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_car.jpg"/></div>

为了正确实现不透明度，我们将使用预乘Alpha（premultiplied alpha）格式。给定所需的不透明度 $ \alpha_{opacity} $ 和漫反射颜色 $ \sigma $（线性、非预乘），我们可以计算片元的有效不透明度。

$$\begin{align*}
color &= \sigma * \alpha_{opacity} \\
opacity &= \alpha_{opacity}
\end{align*}$$

其物理含义是，源颜色的RGB分量定义了该像素发射了多少光，而Alpha分量定义了该像素阻挡了多少其背后的光。因此，我们必须使用以下混合函数：

$$\begin{align*}
Blend_{src} &= 1 \\
Blend_{dst} &= 1 - src_{\alpha}
\end{align*}$$

这些方程的GLSL实现如清单[surfaceTransparency]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// baseColor has already been premultiplied
vec4 shadeSurface(vec4 baseColor) {
    float alpha = baseColor.a;

    vec3 diffuseColor = evaluateDiffuseLighting();
    vec3 specularColor = evaluateSpecularLighting();    

    return vec4(diffuseColor + specularColor, alpha);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [surfaceTransparency]: GLSL中带光照表面透明度的实现]

### 半透明度

半透明材质可以分为两类：
- 表面半透明（Surface translucency）
- 体积半透明（Volume translucency）

体积半透明对于光照粒子系统很有用，例如云或烟雾。表面半透明可用于模拟具有透射散射的材质，如蜡、大理石、皮肤等。

[TODO] 表面半透明（BRDF+BTDF, BSSRDF）

<div align="center"><img src="https://google.github.io/filament/images/screenshot_translucency.png"/></div>

## 遮挡

遮挡是一个重要的变暗因子，用于在不同尺度上重建阴影效果：

小尺度
:   微遮挡（Micro-occlusion），用于处理折痕、裂缝和孔洞。

中尺度
:   宏遮挡（Macro-occlusion），用于处理物体自身几何体或烘焙到法线贴图中的几何体（砖块等）造成的遮挡。

大尺度
:   来自物体间接触或物体自身几何体的遮挡。

我们目前忽略微遮挡，这在工具和引擎中通常以"凹陷贴图"（cavity map）的形式提供。Sébastien Lagarde 在[#Lagarde14]中提供了一个关于Frostbite中如何处理微遮挡的有趣讨论：漫反射微遮挡预烘焙在漫反射贴图中，镜面微遮挡预烘焙在反射纹理中。在我们的系统中，微遮挡可以简单地烘焙到基础颜色贴图中。这样做时必须知道，镜面光将不受微遮挡影响。

中尺度环境光遮蔽预烘焙在环境光遮蔽贴图中，作为材质参数暴露，如前文材质参数化部分所述。

大尺度环境光遮蔽通常使用屏幕空间技术计算，如 *SSAO*（屏幕空间环境光遮蔽）、*HBAO*（基于水平线的环境光遮蔽）等。请注意，当相机足够靠近表面时，这些技术也可以贡献于中尺度环境光遮蔽。

**注意**：为防止同时使用中尺度和大尺度遮挡时过度变暗，Lagarde 建议使用 $min({AO}_{medium}, {AO}_{large})$。

### 漫反射遮挡

Morgan McGuire 在[#McGuire10]中在基于物理渲染的背景下形式化了环境光遮蔽。在他的表述中，McGuire 定义了一个环境照明函数 $ L_a $，在我们的情况下，该函数用球谐函数编码。他还定义了一个可见性函数 $V$，其中 $V(l)=1$ 表示从表面沿 $l$ 方向存在无遮挡的视线，否则为0。

使用这两个函数，渲染方程的环境项可以表示为方程 $\ref{diffuseAO}$ 所示。

$$\begin{equation}\label{diffuseAO}
L(l,v) = \int_{\Omega} f(l,v) L_a(l) V(l) \left< \NoL \right> dl
\end{equation}$$

这个表达式可以通过将可见性项与照明函数分离来近似，如方程 $\ref{diffuseAOApprox}$ 所示。

$$\begin{equation}\label{diffuseAOApprox}
L(l,v) \approx \left( \pi \int_{\Omega} f(l,v) L_a(l) dl \right) \left( \frac{1}{\pi} \int_{\Omega} V(l) \left< \NoL \right> dl \right)
\end{equation}$$

这种近似仅在远距离光 $ L_a $ 是常数且 $f$ 是朗伯项时才是精确的。然而，McGuire 指出，如果两个函数在球面的大部分区域都相对平滑，这种近似是合理的。远距离光照探测器（IBL）恰好满足这一条件。

该近似的左侧项是我们IBL的预计算漫反射分量。右侧项是一个介于0和1之间的标量因子，表示某点的可到达性分数（fractional accessibility）。其相反面是漫反射环境光遮蔽项，如方程 $\ref{diffuseAOTerm}$ 所示。

$$\begin{equation}\label{diffuseAOTerm}
{AO} = 1 - \frac{1}{\pi} \int_{\Omega} V(l) \left< \NoL \right> dl
\end{equation}$$

由于我们使用预计算的漫反射项，无法在运行时计算着色点的精确可到达性。为了补偿预计算项中缺乏此信息，我们通过应用一个特定于着色点处表面材质的环境光遮蔽因子来部分重建入射光照。

在实践中，烘焙的环境光遮蔽存储为灰度纹理，其分辨率通常可以比其他纹理（例如基础颜色或法线）更低。需要注意的是，我们材质模型的环境光遮蔽属性旨在重现宏观级别的漫反射环境光遮蔽。虽然这种近似在物理上并不完全正确，但它构成了一个可接受的质量与性能权衡。

图[aoComparison]展示了两种不同的材质，分别在不带和带有漫反射环境光遮蔽的情况下的对比。注意材质的环境光遮蔽如何用于重建不同瓷砖之间发生的自然阴影。没有环境光遮蔽时，两种材质都显得过于平坦。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_ao.jpg"/></div>

在GLSL着色器中应用烘焙的漫反射环境光遮蔽非常直接，如清单[bakedDiffuseAO]所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// diffuse indirect
vec3 indirectDiffuse = max(irradianceSH(n), 0.0) * Fd_Lambert();
// ambient occlusion
indirectDiffuse *= texture2D(aoMap, outUV).r;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [bakedDiffuseAO]: GLSL中烘焙漫反射环境光遮蔽的实现]

请注意，环境光遮蔽项仅应用于间接光照。

### 镜面遮挡

镜面微遮挡可以从 $\fNormal$ 推导出来，而 $\fNormal$ 本身又从漫反射颜色推导出来。该推导基于一个认识：没有真实世界的材质具有低于2%的反射率。因此，0-2%范围内的值可以被视为预烘焙的镜面遮挡，用于平滑地熄灭菲涅耳项。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float f90 = clamp(dot(f0, 50.0 * 0.33), 0.0, 1.0);
// cheap luminance approximation
float f90 = clamp(50.0 * f0.g, 0.0, 1.0);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularMicroOcclusion]: GLSL中预烘焙的镜面遮挡]

前述环境光遮蔽的推导假设朗伯表面，且仅对间接漫反射光照有效。表面可到达性信息的缺乏对间接镜面光照的重建尤其有害，通常表现为漏光（light leaks）。

Sébastien Lagarde 在[#Lagarde14]中提出了一种经验性方法，从漫反射遮挡项推导出镜面遮挡项。结果没有物理基础，但产生了视觉上令人满意的效果。他的公式的目标是：对于粗糙表面，保持漫反射遮挡项不变；对于光滑表面，该公式（在清单[specularOcclusion]中实现）降低了法线入射时的遮挡影响，并在掠射角时增加遮挡。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float computeSpecularAO(float NoV, float ao, float roughness) {
    return clamp(pow(NoV + ao, exp2(-16.0 * roughness - 1.0)) - 1.0 + ao, 0.0, 1.0);
}

// specular indirect
vec3 indirectSpecular = evaluateSpecularIBL(r, perceptualRoughness);
// ambient occlusion
float ao = texture2D(aoMap, outUV).r;
indirectSpecular *= computeSpecularAO(NoV, ao, roughness);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [specularOcclusion]: GLSL中Lagarde镜面遮挡因子的实现]

请注意，镜面遮挡因子仅应用于间接光照。

#### 水平线镜面遮挡

当为使用法线贴图的表面计算镜面IBL贡献时，反射向量可能指向表面内部。如果直接使用此反射向量进行着色，表面将在不应被照亮的位置被照亮（假设是不透明表面）。这是漏光的另一种表现，可以使用Jeff Russell [ #Russell15]描述的简单技术轻松缓解。

核心思想是遮挡来自表面背后的光。这很容易实现，因为反射向量与表面法线的点积为负值就表明反射向量指向表面内部。我们的实现如清单[horizonOcclusion]所示，与Russell的类似，只是没有美术师控制的地平线衰减因子。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// specular indirect
vec3 indirectSpecular = evaluateSpecularIBL(r, perceptualRoughness);

// horizon occlusion with falloff, should be computed for direct specular too
float horizon = min(1.0 + dot(r, n), 1.0);
indirectSpecular *= horizon * horizon;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [horizonOcclusion]: GLSL中水平线镜面遮挡的实现]

水平线镜面遮挡衰减成本低廉，但可以根据性能需要轻松省略。

## 法线贴图

法线贴图有两个常见用途：用低多边形网格替换高多边形网格（使用基础贴图（base map））和添加表面细节（使用细节贴图（detail map））。

假设我们想要渲染一件覆盖有簇绒皮革的家具。对几何体进行建模以准确表现簇绒图案需要太多三角形，因此我们将高多边形网格烘焙到法线贴图中。一旦将基础贴图应用于简化网格（此处为一个四边形），我们得到图[normalMapped]中的结果。用于创建此效果的基础贴图如图[baseNormalMap]所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_normal_mapping.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_normal_map.jpg"/></div>

如果我们现在想将此基础贴图与第二个法线贴图结合，就会出现一个简单的问题。例如，让我们使用图[detailNormalMap]中所示的细节贴图来添加皮革上的裂纹。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_normal_map_detail.jpg"/></div>

鉴于法线贴图的特性（XYZ分量存储在切线空间中），显然线性或叠加混合等朴素方法无法奏效。我们将使用两种更高级的技术：一种数学上正确的方法和一种适用于实时着色的近似方法。

### 重定向法线贴图（Reoriented normal mapping）

Colin Barré-Brisebois 和 Stephen Hill 在[#Hill12]中提出了一种数学上合理的解决方案，称为*重定向法线贴图*（Reoriented Normal Mapping），该方法将细节贴图的基旋转到基础贴图的法线上。该技术依靠最短弧四元数来应用旋转，由于切线空间的属性，这大大简化了。

根据[#Hill12]中描述的简化，我们可以得到清单[reorientedNormalMapping]中所示的GLSL实现。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 t = texture(baseMap,   uv).xyz * vec3( 2.0,  2.0, 2.0) + vec3(-1.0, -1.0,  0.0);
vec3 u = texture(detailMap, uv).xyz * vec3(-2.0, -2.0, 2.0) + vec3( 1.0,  1.0, -1.0);
vec3 r = normalize(t * dot(t, u) - u * t.z);
return r;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [reorientedNormalMapping]: GLSL中重定向法线贴图的实现]

请注意，此实现假设法线在源纹理中以未压缩形式存储在[0..1]范围内。

归一化步骤并非严格必要，如果在运行时使用该技术可以跳过。如果跳过，`r`的计算变为 `t * dot(t, u) / t.z - u`。

由于此技术比下面描述的方法略贵，我们主要在离线时使用它。因此，我们提供了一个简单的离线工具来合并两个法线贴图。图[blendedNormalMaps]展示了该工具的输出，使用了前面展示的基础贴图和细节贴图。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_normal_map_blended.jpg"/></div>

### UDN混合

[#Hill12]中描述的UDN混合技术是偏导数混合技术的一种变体。其主要优点是需要较少的着色器指令（见清单[udnBlending]）。虽然在平坦区域上会导致细节减少，但如果必须在运行时进行混合，UDN混合是值得考虑的方案。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 t = texture(baseMap,   uv).xyz * 2.0 - 1.0;
vec3 u = texture(detailMap, uv).xyz * 2.0 - 1.0;
vec3 r = normalize(t.xy + u.xy, t.z);
return r;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [udnBlending]: GLSL中UDN混合的实现]

结果在视觉上接近重定向法线贴图，但对数据的仔细比较表明UDN的准确性确实较低。图[blendedNormalMapsUDN]展示了使用与前述示例相同源数据的UDN混合方法的结果。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_normal_map_blended_udn.jpg"/></div>

# 体积效果

## 指数高度雾

<div align="center"><img src="https://google.github.io/filament/images/screenshot_fog1.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_fog2.jpg"/></div>

# 抗锯齿

[待办] MSAA、几何抗锯齿（法线和粗糙度）、着色器抗锯齿（物体空间着色？）

# 成像管线

本文档的光照部分描述了光线如何以基于物理的方式与场景中的表面交互。为了获得逼真的结果，我们必须更进一步，考虑将场景亮度（通过光照方程计算得出）转换为可显示像素值所需的变换。

我们将使用的一系列变换构成了以下成像管线：

*************************************************************************************
* .-------------.      .--------------.      .---------------.                      *
* |    Scene    |      |  Normalized  |      |               |                      *
* |  luminance  +----->|  luminance   +----->| White balance |                      *
* |             |      |    (HDR)     |      |               |                      *
* '-------------'      '--------------'      '-------+-------'                      *
*                                                    |                              *
*                                                    v                              *
*                                            .---------------.                      *
*                                            |               |                      *
*                                            | Color grading |                      *
*                                            |               |                      *
*                                            '-------+-------'                      *
*                                                    |                              *
*                                                    v                              *
*                                            .---------------.                      *
*                                            |               |                      *
*                                            | Tone mapping  |                      *
*                                            |               |                      *
*                                            '-------+-------'                      *
*                                                    |                              *
*                                                    v                              *
*                                            .---------------.      .-------------. *
*                                            |               |      |    Pixel    | *
*                                            |     OETF      +----->|    value    | *
*                                            |               |      |    (LDR)    | *
*                                            '---------------'      '-------------' *
*************************************************************************************

**注意**：*OETF*步骤是目标色彩空间的光电传递函数（opto-electronic transfer function）的应用。为清晰起见，此图未包含渐晕、泛光等后处理效果。这些效果将单独讨论。

[待办] 色彩空间（ACES、sRGB、Rec. 709、Rec. 2020等）、伽马/线性等。

## 基于物理的相机

图像变换过程的第一步是使用基于物理的相机来正确曝光场景的出射亮度。

### 曝光设置

由于我们在光照管线中始终使用光度学单位，到达相机的光是以亮度 $L$ 表示的辐能量，单位为 $cd.m^{-2}$。入射到相机传感器的光可能覆盖很大的范围，从星光的 $10^{-5}cd.m^{-2}$ 到太阳的 $10^{9}cd.m^{-2}$。显然我们无法操控更不用说记录如此大范围的值了，因此需要对其进行重新映射。

这种范围重映射在相机中是通过在一定时间内对传感器进行曝光来实现的。为了最大化传感器有限范围的使用，场景的光照范围以"中间灰"（介于黑白之间的值）为中心。因此，曝光是通过手动或自动调节以下3个参数来实现的：

- 光圈（Aperture）
- 快门速度（Shutter speed）
- 感光度（Sensitivity，也称为增益）

光圈
:   记为 $N$，以f制光圈 ƒ 表示，该参数控制相机系统的光圈开合程度。由于f制光圈表示镜头焦距与入瞳直径的比值，高值（ƒ/16）表示小光圈，低值（ƒ/1.4）表示大光圈。除了控制曝光外，光圈还控制景深。

快门速度
:   记为 $t$，以秒 $s$ 为单位，该参数控制光圈保持开启的时间（同时也控制传感器快门(s)的时序，无论是电子快门还是机械快门）。除了控制曝光外，快门速度还控制运动模糊。

感光度
:   记为 $S$，以ISO为单位，该参数控制到达传感器的光的量化方式。由于其单位特性，该参数通常简称为"ISO"或"ISO设置"。除了控制曝光外，感光度还控制噪点数量。

### 曝光值

由于在我们的方程中引用这3个参数会显得笨拙，我们将"曝光三角"总结为一个曝光值，记为EV[^reciprocity]。

EV以2为底的对数刻度表示，1 EV的差值称为一档（stop）。正一档（+1 EV）对应亮度加倍，负一档（-1 EV）对应亮度减半。

公式 $\ref{ev}$ 展示了EV的[正式定义](https://en.wikipedia.org/wiki/Exposure_value)。

$$\begin{equation}\label{ev}
EV = log_2(\frac{N^2}{t})
\end{equation}$$

注意，该定义仅与光圈和快门速度有关，与感光度无关。曝光值按惯例定义为ISO 100下的值，即 $EV_{100}$。由于我们希望采用此惯例，我们需要能够将 $EV_{100}$ 表示为感光度的函数。

由于我们知道EV是以2为底的对数刻度，每档增加或减少亮度2倍，我们可以正式定义 $EV_{S}$，即给定感光度下的曝光值（公式$\ref{evS}$）。

$$\begin{equation}\label{evS}
{EV}_S = EV_{100} + log_2(\frac{S}{100})
\end{equation}$$

计算 $EV_{100}$ 作为3个相机参数的函数是简单的，如$\ref{ev100}$所示。

$$\begin{equation}\label{ev100}
{EV}_{100} = EV_{S} - log_2(\frac{S}{100}) = log_2(\frac{N^2}{t}) - log_2(\frac{S}{100})
\end{equation}$$

注意，操作者（摄影师等）可以通过多种光圈、快门速度和感光度的组合来实现相同的曝光（从而得到相同的EV）。这在此过程中提供了一定的艺术控制空间（景深 vs 运动模糊 vs 颗粒感）。

[^reciprocity]: 我们假设使用数字传感器，这意味着我们不需要考虑倒易律失效（reciprocity failure）问题。

#### 曝光值与亮度

相机类似于点测光表（spot meter），能够测量场景的平均亮度并将其转换为EV，以实现自动曝光，或至少为用户提供曝光指导。

可以定义EV作为场景亮度 $L$ 的函数，其中需要一个设备相关的校准常数 $K$（公式 $\ref{evK}$）。

$$\begin{equation}\label{evK}
EV = log_2(\frac{L \times S}{K})
\end{equation}$$

该常数 $K$ 是反射光测光常数（reflected-light meter constant），因制造商而异。该常数有两个常见值：12.5（佳能、尼康和世光使用）和14（宾得和美能达使用）。鉴于佳能和尼康相机的广泛使用，以及我们自身使用世光测光表，我们选择使用 $K = 12.5$。

由于我们希望使用 $EV_{100}$，我们可以在公式 $\ref{evK}$ 中代入 $K$ 和 $S$ 得到公式 $\ref{ev100L}$。

$$\begin{equation}\label{ev100L}
EV = log_2(L \frac{100}{12.5})
\end{equation}$$

根据这种关系，我们可以通过首先测量一帧的平均亮度，在我们的引擎中实现自动曝光。一个简单的方法是将亮度缓冲逐级降采样到1像素并读取剩余值。不幸的是，这种技术很少稳定，且容易受到极端值的影响。许多游戏采用另一种方法，即使用亮度直方图（luminance histogram）来去除极端值。

为了验证和测试，可以从给定的EV计算亮度：

$$\begin{equation}
L = 2^{EV_{100}} \times \frac{12.5}{100} = 2^{EV_{100} - 3}
\end{equation}$$

#### 曝光值与照度

可以定义EV作为照度 $E$ 的函数，其中需要一个设备相关的校准常数 $C$：

$$\begin{equation}\label{evC}
EV = log_2(\frac{E \times S}{C})
\end{equation}$$

常数 $C$ 是入射光测光常数（incident-light meter constant），因制造商和/或传感器类型而异。有两种常见的传感器类型：平面型（flat）和半球型（hemispherical）。对于平面传感器，常用值为250。对于半球型传感器，有两个常见值：320（美能达使用）和340（世光使用）。

由于我们希望使用 $EV_{100}$，我们可以在公式 $\ref{evC}$ 中代入 $S$ 得到公式 $\ref{ev100C}$。

$$\begin{equation}\label{ev100C}
EV = log_2(E \frac{100}{C})
\end{equation}$$

然后可以从给定的EV计算照度。对于 $C = 250$ 的平面传感器，我们得到公式 $\ref{eFlatSensor}$。

$$\begin{equation}\label{eFlatSensor}
E = 2^{EV_{100}} \times 2.5
\end{equation}$$

对于 $C = 340$ 的半球型传感器，我们得到公式 $\ref{eHemisphereSensor}$

$$\begin{equation}\label{eHemisphereSensor}
E = 2^{EV_{100}} \times 3.4
\end{equation}$$

#### 曝光补偿

尽管曝光值实际上表示相机设置的组合，但它常被摄影师用来描述光强度。这就是为什么相机允许摄影师应用曝光补偿来使图像过度曝光或曝光不足。该设置可用于艺术控制，也可用于实现正确曝光（例如，雪景将按18%中间灰曝光）。

应用曝光补偿 $EC$ 只需向曝光值添加一个偏移量，如公式 $\ref{ec}$ 所示。

$$\begin{equation}\label{ec}
EV_{100}' = EV_{100} - EC
\end{equation}$$

该公式使用负号，因为我们使用以f制光圈为单位的 $EC$ 来调整最终曝光。增加EV相当于缩小镜头光圈（或降低快门速度或降低感光度）。较高的EV会产生较暗的图像。

### 曝光

要将场景亮度转换为归一化亮度，我们必须使用[光曝光量](https://en.wikipedia.org/wiki/Exposure_value#Camera_settings_vs._photometric_exposure)（photometric exposure，或称亮度曝光量），即到达相机传感器的场景亮度值。光曝光量以勒克斯秒（lux seconds）表示，记为 $H$，由公式 $\ref{photometricExposure}$ 给出。

$$\begin{equation}\label{photometricExposure}
H = \frac{q \cdot t}{N^2} L
\end{equation}$$

其中 $L$ 是场景亮度，$t$ 是快门速度，$N$ 是光圈，$q$ 是镜头和渐晕衰减因子（通常 $q = 0.65$[^lensAttenuation]）。该定义未考虑传感器感光度。为此，我们必须使用三种关联光曝光量和感光度的方法之一：基于饱和速度（saturation-based speed）、基于噪声速度（noise-based speed）和标准输出灵敏度（standard output sensitivity）。

我们选择基于饱和速度的关系，该关系给出 $H_{sat}$，即不导致相机输出裁剪或泛光的最大可能曝光值（公式 $\ref{hSat}$）。

$$\begin{equation}\label{hSat}
H_{sat} = \frac{78}{S_{sat}}
\end{equation}$$

我们在公式 $\ref{lmax}$ 中结合公式 $\ref{hSat}$ 和 $\ref{photometricExposure}$，以计算在曝光设置 $S$、$N$ 和 $t$ 下会使传感器饱和的最大亮度 $L_{max}$。

$$\begin{equation}\label{lmax}
L_{max} = \frac{N^2}{q \cdot t} \frac{78}{S}
\end{equation}$$

然后可以使用该最大亮度将入射亮度 $L$ 归一化，如公式 $\ref{normalizedLuminance}$ 所示。

$$\begin{equation}\label{normalizedLuminance}
L' = L \frac{1}{L_{max}}
\end{equation}$$

$L_{max}$ 可以使用公式 $\ref{ev}$、$S = 100$ 和 $q = 0.65$ 进行简化：

$$\begin{align*}
L_{max} &= \frac{N^2}{t} \frac{78}{q \cdot S} \\
L_{max} &= 2^{EV_{100}} \frac{78}{q \cdot S} \\
L_{max} &= 2^{EV_{100}} \times 1.2
\end{align*}$$

代码清单 [fragmentExposure] 展示了如何将曝光项直接应用于片段着色器中计算的像素颜色。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// Computes the camera's EV100 from exposure settings
// aperture in f-stops
// shutterSpeed in seconds
// sensitivity in ISO
float exposureSettings(float aperture, float shutterSpeed, float sensitivity) {
    return log2((aperture * aperture) / shutterSpeed * 100.0 / sensitivity);
}

// Computes the exposure normalization factor from
// the camera's EV100
float exposure(float ev100) {
    return 1.0 / (pow(2.0, ev100) * 1.2);
}

float ev100 = exposureSettings(aperture, shutterSpeed, sensitivity);
float exposure = exposure(ev100);

vec4 color = evaluateLighting();
color.rgb *= exposure;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [fragmentExposure]: Implementation of exposure in GLSL]

在实际应用中，曝光因子可以在CPU上预先计算以节省着色器指令。

[^lensAttenuation]: 参见维基百科上的《Film Speed, Measurements and calculations》（https://en.wikipedia.org/wiki/Film_speed）

### 自动曝光

上述过程依赖于美术师手动设置相机曝光参数。这在实际操作中可能很繁琐，因为相机移动和/或动态效果会极大地影响场景亮度。由于我们知道如何根据给定亮度计算曝光值（参见章节[曝光值与亮度]），我们可以将相机转换为点测光表。为此，我们需要测量场景的亮度。

有两种常用的技术用于测量场景亮度：

- **亮度降采样（Luminance downsampling）**，通过对前一帧进行逐级降采样，直到获得一个1x1的对数亮度缓冲，可以在CPU上读取（也可以使用计算着色器实现）。结果是场景的平均对数亮度。第一次降采样必须首先提取每个像素的亮度。这种技术可能不稳定，其输出应随时间进行平滑处理。
- **使用亮度直方图（Luminance histogram）**，来找到平均对数亮度。这种技术相比前一种有一个优势，因为它可以忽略极端值并提供更稳定的结果。

注意，两种方法都是在乘以反照率（albedo）后找到平均亮度。这并不完全正确，但替代方案是保持一个包含每个像素在乘以表面反照率之前的亮度的亮度缓冲，这在计算和内存方面都很昂贵。

这两种技术也将测光系统限制为平均测光（average metering），其中每个像素对最终曝光具有相同的影响（或权重）。相机通常提供3种测光模式：

点测光（Spot metering）
:   只有图像中心的一个小圆形区域对最终曝光有贡献。该圆通常占图像总大小的1%到5%。

中心加权测光（Center-weighted metering）
:   对位于屏幕中心的场景亮度值赋予更大的影响。

多区域或矩阵测光（Multi-zone or matrix metering）
:   一种因制造商而异的测光模式。该模式的目标是优先考虑场景中最重要部分的曝光。通常通过将图像分割成网格并分类每个单元格（使用对焦信息、最小/最大亮度等）来实现。高级实现尝试将场景与已知数据集进行比较以实现正确曝光（逆光日落、阴天雪景等）。

#### 点测光

每个亮度值在计算场景亮度时所用的权重 $w$ 由公式 $\ref{spotMetering}$ 给出。

$$\begin{equation}\label{spotMetering}
w(x,y) = \begin{cases} 1 & \left| p_{x,y} - s_{x,y} \right| \le s_r \\ 0 & \left| p_{x,y} - s_{x,y} \right| \gt s_r \end{cases}
\end{equation}$$

其中 $p$ 是像素的位置，$s$ 是测光点中心，$s_r$ 是测光点半径。

#### 中心加权测光

$$\begin{equation}\label{centerMetering}
w(x,y) = smooth(\left| p_{x,y} - c \right| \times \frac{2}{width} )
\end{equation}$$

其中 $c$ 是屏幕中心，$smooth()$ 是类似于GLSL的 `smoothstep()` 的平滑函数。

#### 自适应

为了平滑测光结果，我们可以使用公式 $\ref{adaptation}$，这是 Pattanaik 等人在 [Pattanaik00] 中描述的指数反馈回路。

$$\begin{equation}\label{adaptation}
L_{avg} = L_{avg} + (L - L_{avg}) \times (1 - e^{-\Delta t \cdot \tau})
\end{equation}$$

其中 $\Delta t$ 是上一帧的增量时间，$\tau$ 是控制自适应速率的常数。

### 泛光（Bloom）

由于EV刻度几乎是对数感知线性的，曝光值也常被用作光照单位。这意味着我们可以让美术师使用曝光补偿作为单位来指定光源或自发光表面的强度。发射光的强度因此将相对于曝光设置。应尽可能避免使用曝光补偿作为光照单位，但在某些情况下，它可以用于独立于相机设置强制（或取消）自发光表面周围的泛光效果（例如，游戏中的光剑应始终产生泛光）。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_bloom.jpg"/></div>

使用 $c$ 表示泛光颜色，$EV_{100}$ 表示当前曝光值，我们可以轻松计算泛光值的亮度，如公式 $\ref{bloomEV}$ 所示。

$$\begin{equation}\label{bloomEV}
EV_{bloom} = EV_{100} + EC \\
L_{bloom} = c \times 2^{EV_{bloom} - 3}
\end{equation}$$

公式 $\ref{bloomEV}$ 可以在片段着色器中用于实现自发光泛光，如代码清单 [fragmentEmissive] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec4 surfaceShading() {
    vec4 color = evaluateLights();
    // rgb = color, w = exposure compensation
    vec4 emissive = getEmissive();
    color.rgb += emissive.rgb * pow(2.0, ev100 + emissive.w - 3.0);
    color.rgb *= exposure;
    return color;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [fragmentEmissive]: Implementation of emissive bloom in GLSL]

## 光学后处理

### 色差（Color fringing）

[待办]

<div align="center"><img src="https://google.github.io/filament/images/screenshot_fringing.jpg"/></div>

### 镜头光晕（Lens flares）

[待办] 注：有一种基于物理的方法通过追踪光线穿过镜头光学组件来生成镜头光晕，但我们将使用基于图像的方法。这种方法成本更低，且有一些有益的优点，如自发光的自动遮挡和无限制的光源支持。

## 电影级后处理

[待办] 尽可能在场景参照数据（scene referred data，线性空间，色调映射之前）上进行后处理。

提供色彩校正工具以赋予美术师对最终图像更大的艺术控制力非常重要。这些工具可以在每个照片或视频处理应用程序中找到，如Adobe Photoshop或Adobe After Effects。

### 对比度（Contrast）

### 曲线（Curves）

### 色阶（Levels）

### 色彩分级（Color grading）

## 光照路径

引擎使用的光照路径（light path），或称渲染方法，可能会对性能产生严重影响，并可能对场景中可以使用多少光源施加严格限制。传统上，3D引擎使用两种不同的渲染方法：前向渲染（forward rendering）和延迟渲染（deferred rendering）。

我们的目标是使用一种满足以下约束的渲染方法：

- 低带宽需求
- 每像素多个动态光源

此外，我们希望轻松支持：

- MSAA
- 透明度（Transparency）
- 多种材质模型

延迟渲染被许多现代3D渲染引擎用来轻松支持数十、数百甚至数千个光源（以及其他优点）。不幸的是，这种方法在带宽方面非常昂贵。使用我们的默认PBR材质模型，G缓冲（G-buffer）将每像素使用160到192位，这将直接转化为相当高的带宽需求。

另一方面，前向渲染方法在历史上一直不擅长处理多光源。一种常见的实现是多次渲染场景，每次针对一个可见光源，然后混合（相加）结果。另一种技术是为场景中的每个对象分配固定的最大光源数。然而，当对象占据世界中大量空间（建筑、道路等）时，这是不切实际的。

瓦片着色（Tiled shading）可以应用于前向渲染和延迟渲染方法。其思想是将屏幕分割成瓦片网格，并为每个瓦片找到影响该瓦片内像素的光源列表。这具有减少过度绘制（在延迟渲染中）和大对象的着色计算（在前向渲染中）的优点。然而，这种技术存在深度不连续性问题，可能导致大量额外工作。

图 [sponza] 中显示的场景是使用聚类前向渲染（clustered forward rendering）渲染的。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_sponza.jpg"/></div>

图 [sponzaTiles] 显示了同一场景分割成瓦片（本例中为1280x720渲染目标，80x80像素瓦片）。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_sponza_tiles.jpg"/></div>

### 聚类前向渲染（Clustered Forward Rendering）

我们决定探索另一种称为聚类着色（Clustered Shading）的方法，采用其前向变体。聚类着色扩展了瓦片渲染的思想，但增加了第三维的分割。"聚类"在视图空间（view space）中进行，通过将视锥体（frustum）分割成3D网格。

首先在深度轴上对视锥体进行切片，如图 [sponzaSlices] 所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_sponza_slices.jpg"/></div>

然后将深度切片与屏幕瓦片结合以"体素化"视锥体。我们称每个聚类为一个froxel，因为这清晰地表明了它们所代表的内容（视锥体空间中的体素，voxel in frustum space）。"froxel化"过程的结果如图 [froxel1] 和图 [froxel2] 所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_sponza_froxels1.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_sponza_froxels2.jpg"/></div>

在渲染一帧之前，场景中的每个光源被分配到与其相交的任何froxel。光源分配过程的结果是每个froxel的光源列表。在渲染过程中，我们可以计算片段所属的froxel的ID，从而获得可能影响该片段的光源列表。

深度切片不是线性的，而是指数的。在典型场景中，靠近近平面（near plane）的像素比靠近远平面（far plane）的像素更多。因此，froxel的指数网格将改善在最重要区域的光源分配。

图 [froxelDistribution] 显示了在使用指数切片时，每个深度切片使用了多少世界空间单位。

<div align="center"><img src="https://google.github.io/filament/images/diagram_froxels1.png"/></div>

简单的指数体素化是不够的。上图清晰显示了世界空间如何在切片间分布，但它未能显示近平面附近发生的情况。如果我们检查较小范围（0.1m到7m）内的相同分布，我们可以看到出现了一个有趣的问题，如图 [froxelDistributionClose] 所示。

<div align="center"><img src="https://google.github.io/filament/images/diagram_froxels2.png"/></div>

该图显示，简单的指数分布在非常靠近相机的地方使用了半数切片。在这个特定例子中，我们在前5米内使用了16个切片中的8个。由于动态世界光源要么是点光源（球体），要么是聚光灯（锥体），如此精细的分辨率在如此靠近近平面的地方完全没有必要。

我们的解决方案是根据场景以及近平面和远平面手动调整第一个froxel的大小。通过这样做，我们可以更好地在视锥体中分布剩余的froxel。图 [froxelDistributionExp] 显示了例如当我们使用0.1m到5m之间的特殊froxel时发生的情况。

<div align="center"><img src="https://google.github.io/filament/images/diagram_froxels3.png"/></div>

这种新分布效率更高，允许在整个视锥体中更好地分配光源。

### 实现说明

光源分配可以通过两种不同方式完成：在GPU上或在CPU上。

#### GPU光源分配

该实现需要OpenGL ES 3.1和对计算着色器（compute shaders）的支持。光源存储在着色器存储缓冲对象（Shader Storage Buffer Objects, SSBO）中，并传递给计算着色器，计算着色器将每个光源分配到相应的froxel。

视锥体体素化可以由第一个计算着色器仅执行一次（只要投影矩阵不变），光源分配可以每帧由另一个计算着色器执行。

计算着色器的线程模型特别适合此任务。我们只需调用与froxel数量相同的工作组（workgroups）（我们可以直接将X、Y和Z工作组计数映射到我们的froxel网格分辨率）。每个工作组将依次进行线程化并遍历所有要分配的光源。

相交测试涉及简单的球体/视锥体或锥体/视锥体测试。

有关GPU实现的源代码（仅点光源），请参见附录。

#### CPU光源分配

在非OpenGL ES 3.1设备上，光源分配可以在CPU上高效执行。算法与GPU实现不同。引擎不是为每个froxel遍历每个光源，而是将每个光源"光栅化"为froxel。例如，给定点光源的中心和半径，计算其相交的froxel列表是简单的。

这种技术还有一个额外的好处，即提供比GPU变体更紧密的剔除。CPU实现也可以更容易地生成紧凑的光源列表。

#### 着色

每个froxel的光源列表可以作为SSBO（OpenGL ES 3.1）或纹理传递给片段着色器。

#### 从深度到froxel

给定近平面 $n$、远平面 $f$、最大深度切片数 $m$ 以及范围 [0..1] 内的线性深度值 $z$，公式 $\ref{zToCluster}$ 可用于计算给定位置的聚类索引。

$$\begin{equation}\label{zToCluster}
zToCluster(z,n,f,m)=floor \left( max \left( log2(z) \frac{m}{-log2(\frac{n}{f})} + m, 0 \right) \right)
\end{equation}$$

然而，该公式存在前面提到的分辨率问题。我们可以通过引入 $sn$（一个特殊的近值，定义第一个froxel的范围）来解决此问题。第一个froxel占据范围 [n..sn]，剩余froxel占据 [sn..f]。

$$\begin{equation}\label{zToClusterFix}
zToCluster(z,n,sn,f,m)=floor \left( max \left( log2(z) \frac{m-1}{-log2(\frac{sn}{f})} + m, 0 \right) \right)
\end{equation}$$

公式 $\ref{linearZ}$ 可用于从 `gl_FragCoord.z` 计算线性深度值（假设使用标准的OpenGL投影矩阵）。

$$\begin{equation}\label{linearZ}
linearZ(z)=\frac{n}{f+z(n-f)}
\end{equation}$$

该公式可以通过预先计算两个项 $c0$ 和 $c1$ 进行简化，如公式 $\ref{linearZFix}$ 所示。

$$\begin{equation}\label{linearZFix}
c1 = \frac{f}{n} \\
c0 = 1 - c1 \\
linearZ(z)=\frac{1}{z \cdot c0 + c1}
\end{equation}$$

这个简化很重要，因为我们在 $\ref{zToClusterFix}$ 中将线性z值传递给 `log2`。由于除法在对数下变为取负，我们可以通过使用 $-log2(z \cdot c0 + c1)$ 来避免除法。

综合起来，计算给定片段的froxel索引可以相当容易地实现，如代码清单 [fragCoordToFroxel] 所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#define MAX_LIGHT_COUNT 16 // max number of lights per froxel

uniform uvec4 froxels; // res x, res y, count y, count y
uniform vec4 zParams;  // c0, c1, index scale, index bias

uint getDepthSlice() {
    return uint(max(0.0, log2(zParams.x * gl_FragCoord.z + zParams.y) *
            zParams.z + zParams.w));
}

uint getFroxelOffset(uint depthSlice) {
    uvec2 froxelCoord = uvec2(gl_FragCoord.xy) / froxels.xy;
    froxelCoord.y = (froxels.w - 1u) - froxelCoord.y;

    uint index = froxelCoord.x + froxelCoord.y * froxels.z +
            depthSlice * froxels.z * froxels.w;
    return index * MAX_FROXEL_LIGHT_COUNT;
}

uint slice = getDepthSlice();
uint offset = getFroxelOffset(slice);

// Compute lighting...
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [fragCoordToFroxel]: GLSL implementation to compute a froxel index from a fragment's screen coordinates]

必须预先计算几个uniform变量以高效执行索引计算。用于预先计算这些uniform变量的代码可以在代码清单 [froxelIndexPrecomputation] 中找到。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
froxels[0] = TILE_RESOLUTION_IN_PX;
froxels[1] = TILE_RESOLUTION_IN_PX;
froxels[2] = numberOfTilesInX;
froxels[3] = numberOfTilesInY;

zParams[0] = 1.0f - Z_FAR / Z_NEAR;
zParams[1] = Z_FAR / Z_NEAR;
zParams[2] = (MAX_DEPTH_SLICES - 1) / log2(Z_SPECIAL_NEAR / Z_FAR);
zParams[3] = MAX_DEPTH_SLICES;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [froxelIndexPrecomputation]]

#### 从froxel到深度

给定froxel索引 $i$、特殊近平面 $sn$、远平面 $f$ 以及最大深度切片数 $m$，公式 $\ref{clusterToZ}$ 计算给定froxel的最小深度。

$$\begin{equation}\label{clusterToZ}
clusterToZ(i \ge 1,sn,f,m)=2^{(i-m) \frac{-log2(\frac{sn}{f})}{m-1}}
\end{equation}$$

对于 $i=0$，z值为0。该公式的结果在 [0..1] 范围内，应乘以 $f$ 以获得世界单位中的距离。

计算着色器实现应使用 `exp2` 而不是 `pow`。除法可以预先计算并作为uniform变量传递。

## 验证

鉴于我们光照系统的复杂性，验证我们的实现非常重要。我们将通过多种方式进行验证：使用参考渲染（reference renderings）、光照测量和数据可视化。

[待办] 解释光照测量验证（从渲染目标读取EV并与测光表/相机等测量值进行比较）

### 场景参照可视化

快速简便地验证场景光照的一种方法是修改着色器以输出颜色，提供到相关数据的直观映射。这可以通过使用输出伪颜色的自定义调试色调映射算子轻松实现。

#### 亮度光阑（Luminance stops）

使用自发光材质和IBL，很容易获得镜面高光比其明显来源更亮的场景。这类问题在色调映射和量化后可能难以观察，但在场景参照空间（scene-referred space）中相当明显。图 [luminanceViz] 显示了如何使用代码清单 [tonemapLuminanceViz] 中描述的自定义算子来显示场景的曝光亮度。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_luminance_debug.png"/></div>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec3 Tonemap_DisplayRange(const vec3 x) {
    // The 5th color in the array (cyan) represents middle gray (18%)
    // Every stop above or below middle gray causes a color shift
    float v = log2(luminance(x) / 0.18);
    v = clamp(v + 5.0, 0.0, 15.0);
    int index = int(floor(v));
    return mix(debugColors[index], debugColors[min(15, index + 1)], fract(v));
}

const vec3 debugColors[16] = vec3[](
     vec3(0.0, 0.0, 0.0),         // black
     vec3(0.0, 0.0, 0.1647),      // darkest blue
     vec3(0.0, 0.0, 0.3647),      // darker blue
     vec3(0.0, 0.0, 0.6647),      // dark blue
     vec3(0.0, 0.0, 0.9647),      // blue
     vec3(0.0, 0.9255, 0.9255),   // cyan
     vec3(0.0, 0.5647, 0.0),      // dark green
     vec3(0.0, 0.7843, 0.0),      // green
     vec3(1.0, 1.0, 0.0),         // yellow
     vec3(0.90588, 0.75294, 0.0), // yellow-orange
     vec3(1.0, 0.5647, 0.0),      // orange
     vec3(1.0, 0.0, 0.0),         // bright red
     vec3(0.8392, 0.0, 0.0),      // red
     vec3(1.0, 0.0, 1.0),         // magenta
     vec3(0.6, 0.3333, 0.7882),   // purple
     vec3(1.0, 1.0, 1.0)          // white
);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Listing [tonemapLuminanceViz]: GLSL implementation of a custom debug tone-mapping operator for luminance visualization]

### 参考渲染

要验证我们的实现是否与参考渲染一致，我们将使用一个商业级开源基于物理的离线路径追踪器Mitsuba。Mitsuba提供许多不同的积分器（integrators）、采样器（samplers）和材质模型，这应该允许我们与实时渲染器进行公平比较。该路径追踪器还依赖于简单的XML场景描述格式，该格式应易于从我们自己的场景描述中自动生成。

图 [mitsubaReference] 和图 [filamentReference] 显示了一个简单场景——一个完全光滑的介电球体（perfectly smooth dielectric sphere），分别使用Mitsuba和Filament渲染。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_ref_mitsuba.jpg"/></div>

<div align="center"><img src="https://google.github.io/filament/images/screenshot_ref_filament.jpg"/></div>

用于渲染两个场景的参数如下：

**Filament**

- 材质
  - 基色（Base color）：sRGB 0.81, 0, 0
  - 金属度（Metallic）：0
  - 粗糙度（Roughness）：0
  - 反射率（Reflectance）：0.5
- 间接光照：IBL
  - 由cmgen从office.exr生成的256x256立方体贴图
  - 倍率：35,000
- 直接光照：方向光
  - 线性颜色：1.0, 0.96, 0.95
  - 强度：120,000 lux
- 曝光
  - 光圈：f/16
  - 快门速度：1/125s
  - ISO：100

**Mitsuba**

- BSDF: roughplastic
  - 分布（Distribution）：GGX
  - Alpha: 0
  - 漫反射率（Diffuse reflectance）：sRGB 0.81, 0, 0
- 发射器（Emitter）：环境贴图
  - 源：office.exr
  - 缩放：35,000
- 发射器（Emitter）：方向光
  - 辐照度（Irradiance）：线性 RGB 120,000 115,200 114,000
- 胶片（Film）：LDR
  - 曝光：-15.23，由 log2(filamentExposure) 计算
- 积分器（Integrator）：路径
- 采样器（Sampler）：ldsampler
  - 采样数：256

完整的Mitsuba场景可作为附录找到。两个场景均以相同分辨率（2048x1440）渲染。

#### 比较

两个渲染之间的细微差异来自于Filament使用的各种近似：RGBM 256x256反射探针、RGBM 1024x1024背景贴图、朗伯漫反射（Lambert diffuse）、拆分求和近似（split-sum approximation）、DFG项的解析近似等。

图 [referenceComparison] 显示了两个引擎生成图像的亮度梯度。比较是在LDR图像上进行的。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_ref_comparison.png"/></div>

最大的差异出现在掠射角（grazing angles）处，这很可能是由Filament使用朗伯漫反射项造成的。Disney漫反射项及其掠射逆反射（grazing retro-reflections）将使Filament更接近Mitsuba。

## 坐标系

### 世界坐标系

Filament使用Y轴朝上、右手坐标系。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_coordinates.jpg"/></div>

### 相机坐标系

Filament的相机朝向其局部 -Z 轴方向。也就是说，当在场景中放置一个未应用任何变换的相机时，相机朝向世界的 -Z 轴方向。

### 立方体贴图坐标系

Filament中使用的所有立方体贴图都遵循OpenGL的面排列约定，如图 [cubemapCoordinates] 所示。

<div align="center"><img src="https://google.github.io/filament/images/screenshot_cubemap_coordinates.png"/></div>

注意，环境背景和反射探针是镜像对称的（参见章节[镜像对称]）。

#### 镜像对称

为简化反射的渲染，IBL立方体贴图在X轴上存储为镜像对称。这是 `cmgen` 工具的默认行为。这意味着用作环境背景的IBL立方体贴图需要在运行时再次镜像。对于天空盒，一种简单的实现方法是使用纹理化的背面。Filament默认采用此方式。

#### 等距柱状投影环境贴图

要将等距柱状投影环境贴图转换为水平/垂直交叉立方体贴图，我们将 +Z 面放置在源直线型环境贴图的中心。

#### 环境贴图和天空盒的世界空间朝向

在Filament中指定天空盒或IBL时，指定的立方体贴图的方向使其 -Z 面指向世界的 +Z 轴（这是因为Filament假定为镜像对称的立方体贴图，参见章节[镜像对称]）。然而，由于环境和天空盒预期是预先镜像对称的，它们的 -Z（背面）面按预期指向世界的 -Z 轴（并且相机默认朝向该方向，参见章节[相机坐标系]）。

# 附录

## 镜面反射颜色（Specular color）

金属表面的镜面反射颜色（即$\fNormal$）可直接从实测光谱数据计算得出。诸如 [Refractive Index](https://refractiveindex.info/?shelf=3d&book=metals&page=brass) 等在线数据库提供了各种材料在不同波长下测量的复数折射率数据表。

本文前面介绍了方程 $\ref{fresnelEquation}$，用于计算给定折射率下电介质表面在法线入射时的菲涅耳反射率。通过使用复数表示表面折射率，可将同一方程改写为适用于导体的形式：

$$\begin{equation}
c_{ior} = n_{ior} + ik
\end{equation}$$

方程 $\ref{fresnelComplexIOR}$ 给出了最终的菲涅耳公式，其中 $c^*$ 为复数 $c$ 的共轭：

$$\begin{equation}\label{fresnelComplexIOR}
\fNormal(c_{ior}) = \frac{(c_{ior} - 1)(c_{ior}^* - 1)}{(c_{ior} + 1)(c_{ior}^* + 1)}
\end{equation}$$

要计算材料的镜面反射颜色，我们需要在可见光谱范围内，对每个复数折射率的光谱样本计算复数菲涅耳方程。每个光谱样本得到一个光谱反射率样本。为求得法线入射下的RGB颜色，需要将每个样本与CIE XYZ颜色匹配函数以及目标照明体的光谱功率分布相乘。我们选择标准照明体D65，因为我们希望计算sRGB色彩空间中的颜色。

然后对所有样本求和（积分）并归一化，得到XYZ色彩空间中的$\fNormal$。从该空间出发，经过简单的色彩空间转换即可得到线性sRGB颜色，或者应用光电传递函数（OETF，通常称为"伽马"曲线）后得到非线性sRGB颜色。需要注意的是，对于某些材料（如金），最终的sRGB颜色可能会超出色域。我们使用简单的归一化步骤作为一种低成本的色域重映射方法，但考虑在更宽色域的色彩空间（例如BT.2020）中进行计算将是一个有趣的方向。

为实现所需结果，我们使用了ICE 1931 2度颜色匹配函数（范围360nm至830nm，1nm间隔，[来源](http://cvrl.ioo.ucl.ac.uk/cmfs.htm)）和CIE标准照明体D65相对光谱功率分布（范围300nm至830nm，5nm间隔，[来源](http://files.cie.co.at/204.xls)）。

我们的实现见代码清单 [specularColorImpl]，为简洁起见省略了实际数据。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
// CIE 1931 2-deg color matching functions (CMFs), from 360nm to 830nm,
// at 1nm intervals
//
// Data source:
//     http://cvrl.ioo.ucl.ac.uk/cmfs.htm
//     http://cvrl.ioo.ucl.ac.uk/database/text/cmfs/ciexyz31.htm
const size_t CIE_XYZ_START = 360;
const size_t CIE_XYZ_COUNT = 471;
const float3 CIE_XYZ[CIE_XYZ_COUNT] = { ... };

// CIE Standard Illuminant D65 relative spectral power distribution,
// from 300nm to 830, at 5nm intervals
//
// Data source:
//     https://en.wikipedia.org/wiki/Illuminant_D65
//     https://cielab.xyz/pdf/CIE_sel_colorimetric_tables.xls
const size_t CIE_D65_INTERVAL = 5;
const size_t CIE_D65_START = 300;
const size_t CIE_D65_END = 830;
const size_t CIE_D65_COUNT = 107;
const float CIE_D65[CIE_D65_COUNT] = { ... };

struct Sample {
    float w = 0.0f; // wavelength
    std::complex<float> ior; // complex IOR, n + ik
};

static float illuminantD65(float w) {
    auto i0 = size_t((w - CIE_D65_START) / CIE_D65_INTERVAL);
    uint2 indexBounds{i0, std::min(i0 + 1, CIE_D65_END)};

    float2 wavelengthBounds = CIE_D65_START + float2{indexBounds} * CIE_D65_INTERVAL;
    float t = (w - wavelengthBounds.x) / (wavelengthBounds.y - wavelengthBounds.x);
    return lerp(CIE_D65[indexBounds.x], CIE_D65[indexBounds.y], t);
}

// For std::lower_bound
bool operator<(const Sample& lhs, const Sample& rhs) {
    return lhs.w < rhs.w;
}

// The wavelength w must be between 360nm and 830nm
static std::complex<float> findSample(const std::vector<Sample>& samples, float w) {
    auto i1 = std::lower_bound(
		        samples.begin(), samples.end(), Sample{w, 0.0f + 0.0if});
    auto i0 = i1 - 1;

    // Interpolate the complex IORs
    float t = (w - i0->w) / (i1->w - i0->w);
    float n = lerp(i0->ior.real(), i1->ior.real(), t);
    float k = lerp(i0->ior.imag(), i1->ior.imag(), t);
    return { n, k };
}

static float fresnel(const std::complex<float>& sample) {
    return (((sample - (1.0f + 0if)) * (std::conj(sample) - (1.0f + 0if))) /
            ((sample + (1.0f + 0if)) * (std::conj(sample) + (1.0f + 0if)))).real();
}

static float3 XYZ_to_sRGB(const float3& v) {
    const mat3f XYZ_sRGB{
             3.2404542f, -0.9692660f,  0.0556434f,
            -1.5371385f,  1.8760108f, -0.2040259f,
            -0.4985314f,  0.0415560f,  1.0572252f
    };
    return XYZ_sRGB * v;
}

// Outputs a linear sRGB color
static float3 computeColor(const std::vector<Sample>& samples) {
    float3 xyz{0.0f};
    float y = 0.0f;

    for (size_t i = 0; i < CIE_XYZ_COUNT; i++) {
        // Current wavelength
        float w = CIE_XYZ_START + i;

        // Find most appropriate CIE XYZ sample for the wavelength
        auto sample = findSample(samples, w);
        // Compute Fresnel reflectance at normal incidence
        float f0 = fresnel(sample);

        // We need to multiply by the spectral power distribution of the illuminant
        float d65 = illuminantD65(w);

        xyz += f0 * CIE_XYZ[i] * d65;
        y += CIE_XYZ[i].y * d65;
    }

    // Normalize so that 100% reflectance at every wavelength yields Y=1
    xyz /= y;

    float3 linear = XYZ_to_sRGB(xyz);

    // Normalize out-of-gamut values
    if (any(greaterThan(linear, float3{1.0f}))) linear *= 1.0f / max(linear);

    return linear;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [specularColorImpl]: 从光谱数据计算金属表面基础颜色的C++实现]

特别感谢Naty Hoffman在本主题上提供的宝贵帮助。

## IBL中的重要性采样

在离散域中，积分可通过方程 $\ref{iblSampling}$ 定义的采样来近似。

$$\begin{equation}\label{iblSampling}
\Lout(n,v,\Theta) \equiv \frac{1}{N} \sum_{i}^{N} f(l_{i}^{uniform},v,\Theta) L_{\perp}(l_i) \left< n \cdot l_i^{uniform} \right>
\end{equation}$$

遗憾的是，评估该积分需要过多的样本。一种常用技术是更频繁地选择更"重要"的样本，这称为_重要性采样_。在我们的情况下，我们将使用微表面法线分布 $D_{ggx}$ 作为重要样本的分布。

使用重要性采样评估 $ \Lout(n,v,\Theta) $ 的方法见方程 $\ref{annexIblImportanceSampling}$。

$$\begin{equation}\label{annexIblImportanceSampling}
\Lout(n,v,\Theta) \equiv \frac{1}{N} \sum_{i}^{N} \frac{f(l_{i},v,\Theta)}{p(l_i,v,\Theta)} L_{\perp}(l_i) \left< n \cdot l_i \right>
\end{equation}$$

在方程 $\ref{annexIblImportanceSampling}$ 中，$p$ 是_重要方向样本_ $l_i$ 分布的概率密度函数（PDF）。这些样本依赖于 $h_i$、$v$ 和 $\alpha$。PDF的定义如方程 $\ref{iblPDF}$ 所示。

$h_i$ 由我们选择的分布给出，详见 [选择重要方向] 一节。

_重要方向样本_ $l_i$ 被计算为 $v$ 关于 $h_i$ 的反射，因此**不**具有与 $h_i$ 相同的PDF。变换后的分布的PDF由下式给出：

$$\begin{equation}
p(T_r(x)) = p(x) |J(T_r)|^{-1}
\end{equation}$$

其中 $|J(T_r)|$ 是变换的雅可比行列式。在我们的情况下，我们考虑从 $h_i$ 到 $l_i$ 的变换，其雅可比行列式在 \ref{iblPDF} 中给出。

$$\begin{equation}\label{iblPDF}
p(l,v,\Theta) = D(h,\alpha) \left< \NoH \right> |J_{h \rightarrow l}|^{-1} \\
|J_{h \rightarrow l}| = 4 \left< \VoH \right>
\end{equation}$$

### 选择重要方向

更多细节请参考 [选择用于采样BRDF的重要方向] 一节。给定均匀分布 $(\zeta_{\phi},\zeta_{\theta})$，重要方向 $l$ 由方程 $\ref{importantDirection}$ 定义。

$$\begin{equation}\label{importantDirection}
\phi = 2 \pi \zeta_{\phi} \\
\theta = cos^{-1} \sqrt{\frac{1 - \zeta_{\theta}}{(\alpha^2 - 1)\zeta_{\theta}+1}} \\
l = \{ cos \phi sin \theta, sin \phi sin \theta, cos \theta \}
\end{equation}$$

通常，$ (\zeta_{\phi},\zeta_{\theta}) $ 使用 [Hammersley序列] 一节中描述的Hammersley均匀分布算法选择。

### 预滤波重要性采样

重要性采样仅考虑PDF来生成重要方向；特别是，它不关注IBL的实际内容。如果IBL在样本稀疏的区域包含高频信息，积分将不准确。可以通过一种称为_预滤波重要性采样_的技术在一定程度上缓解此问题，此外该技术还允许用少得多的样本实现积分收敛。

预滤波重要性采样使用对同一环境场景逐渐增加低通滤波的多张图像。这通常通过mipmap和盒式滤波器高效实现。LOD根据样本重要性选择，即低概率样本使用更高的LOD级别（更多滤波）。

该技术的详细描述见 [#Krivanek08]。

立方体贴图的LOD按以下方式确定：

$$\begin{align*}
lod &= log_4 \left( K\frac{\Omega_s}{\Omega_p} \right) \\
K &= 4.0 \\
\Omega_s &= \frac{1}{N \cdot p(l_i)} \\
\Omega_p &\approx \frac{4\pi}{6 \cdot width \cdot height}
\end{align*}$$

其中 $K$ 是由经验确定的常数，$p$ 是BRDF的PDF，$ \Omega_{s} $ 是与样本关联的立体角，$\Omega_p$ 是与立方体贴图中的纹理像素关联的立体角。

立方体贴图采样使用无缝三线性滤波完成。使用OpenGL的无缝采样功能或其他避免/减少接缝的技术，在跨面正确采样立方体贴图是极其重要的。

表 [importanceSamplingViz] 展示了重要性采样与预滤波重要性采样应用于图 [importanceSamplingRef] 时的对比。

<div align="center"><img src="https://google.github.io/filament/images/image_is_original.png" alt="Figure [importanceSamplingRef]: Importance sampling image reference"/></div>

 样本数 |      重要性采样                |    预滤波重要性采样
---------|-------------------------------|---------------------------------------
  4096   | <div align="center"><img src="https://google.github.io/filament/images/image_is_4096.png"/></div> | &nbsp;
  1024   | <div align="center"><img src="https://google.github.io/filament/images/image_is_1024.png"/></div> | <div align="center"><img src="https://google.github.io/filament/images/image_fis_1024.png"/></div>
  32     | <div align="center"><img src="https://google.github.io/filament/images/image_is_32.png"/></div>   | <div align="center"><img src="https://google.github.io/filament/images/image_fis_32.png"/></div>
[表 [importanceSamplingViz]: 重要性采样与预滤波重要性采样的对比，$\alpha = 0.4$]

下面比较中使用的参考渲染器不做任何近似。特别地，它不假设 $v = n$，也不执行拆分求和近似。预滤波渲染器使用了本节讨论的所有技术：预滤波立方体贴图、DFG项的解析形式，当然也包括拆分求和近似。

左：参考渲染器，右：预滤波重要性采样。

<div align="center"><img src="https://google.github.io/filament/images/image_is_ref_1.png"/></div> <div align="center"><img src="https://google.github.io/filament/images/image_filtered_1.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/image_is_ref_2.png"/></div> <div align="center"><img src="https://google.github.io/filament/images/image_filtered_2.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/image_is_ref_3.png"/></div> <div align="center"><img src="https://google.github.io/filament/images/image_filtered_3.png"/></div>
<div align="center"><img src="https://google.github.io/filament/images/image_is_ref_4.png"/></div> <div align="center"><img src="https://google.github.io/filament/images/image_filtered_4.png"/></div>

## 选择用于采样BRDF的重要方向

为简单起见，我们使用BRDF的 $ D $ 项作为PDF，但PDF必须归一化，使得半球上的积分为1：

$$\begin{equation}
\int_{\Omega}p(m)dm = 1 \\
\int_{\Omega}D(m)(n \cdot m)dm = 1 \\
\int_{\phi=0}^{2\pi}\int_{\theta=0}^{\frac{\pi}{2}}D(\theta,\phi) cos \theta sin \theta d\theta d\phi = 1 \\
\end{equation}$$

因此BRDF的PDF可以表示为方程 $\ref{importantPDF}$ 的形式：

$$\begin{equation}\label{importantPDF}
p(\theta,\phi) = \frac{\alpha^2}{\pi(cos^2\theta (\alpha^2-1) + 1)^2} cos\theta sin\theta
\end{equation}$$

由于我们在球面上积分，$sin\theta$ 项来自于微分立体角 $sin\theta d\phi d\theta$。我们对 $\theta$ 和 $\phi$ 独立采样：

$$\begin{align*}
p(\theta) &= \int_0^{2\pi} p(\theta,\phi) d\phi = \frac{2\alpha^2}{(cos^2\theta (\alpha^2-1) + 1)^2} cos\theta sin\theta \\
p(\phi) &= \frac{p(\theta,\phi)}{p(\phi)} = \frac{1}{2\pi}
\end{align*}$$

$ p(\phi) $ 的表达式对于法线的各向同性分布成立。

然后计算每个变量的累积分布函数（CDF）：

$$\begin{align*}
P(s_{\phi}) &= \int_{0}^{s_{\phi}} p(\phi) d\phi = \frac{s_{\phi}}{2\pi} \\
P(s_{\theta}) &= \int_{0}^{s_{\theta}} p(\theta) d\theta = 2 \alpha^2 \left( \frac{1}{(2\alpha^4-4\alpha^2+2) cos(s_{\theta})^2 + 2\alpha^2 - 2} - \frac{1}{2\alpha^4-2\alpha^2} \right)
\end{align*}$$

将 $ P(s_{\phi}) $ 和 $ P(s_{\theta}) $ 设为随机变量 $ \zeta_{\phi} $ 和 $ \zeta_{\theta} $，分别解出 $ s_{\phi} $ 和 $ s_{\theta} $：

$$\begin{align*}
P(s_{\phi}) &= \zeta_{\phi} \rightarrow s_{\phi} = 2\pi\zeta_{\phi} \\
P(s_{\theta}) &= \zeta_{\theta} \rightarrow s_{\theta} = cos^{-1} \sqrt{\frac{1-\zeta_{\theta}}{(\alpha^2-1)\zeta_{\theta}+1}}
\end{align*}$$

因此，给定均匀分布 $ (\zeta_{\phi},\zeta_{\theta}) $，我们的重要方向 $l$ 定义为：

$$\begin{align*}
\phi &= 2\pi\zeta_{\phi} \\
\theta &= cos^{-1} \sqrt{\frac{1-\zeta_{\theta}}{(\alpha^2-1)\zeta_{\theta}+1}} \\
l &= \{ cos\phi sin\theta,sin\phi sin\theta,cos\theta \}
\end{align*}$$

## Hammersley序列

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
vec2f hammersley(uint i, float numSamples) {
    uint bits = i;
    bits = (bits << 16) | (bits >> 16);
    bits = ((bits & 0x55555555) << 1) | ((bits & 0xAAAAAAAA) >> 1);
    bits = ((bits & 0x33333333) << 2) | ((bits & 0xCCCCCCCC) >> 2);
    bits = ((bits & 0x0F0F0F0F) << 4) | ((bits & 0xF0F0F0F0) >> 4);
    bits = ((bits & 0x00FF00FF) << 8) | ((bits & 0xFF00FF00) >> 8);
    return vec2f(i / numSamples, bits / exp2(32));
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[Hammersley序列生成器的C++实现]

## 为基于图像照明预计算L

$ L_{DFG} $ 项仅依赖于 $ \NoV $。下面，法线任意设置为 $ n=\left[0, 0, 1\right] $，并选择 $v$ 以满足 $ \NoV $。向量 $ h_i $ 是 $ D_{GGX}(\alpha) $ 重要方向样本 $i$。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
float GDFG(float NoV, float NoL, float a) {
    float a2 = a * a;
    float GGXL = NoV * sqrt((-NoL * a2 + NoL) * NoL + a2);
    float GGXV = NoL * sqrt((-NoV * a2 + NoV) * NoV + a2);
    return (2 * NoL) / (GGXV + GGXL);
}

float2 DFG(float NoV, float a) {
    float3 V;
    V.x = sqrt(1.0f - NoV*NoV);
    V.y = 0.0f;
    V.z = NoV;

    float2 r = 0.0f;
    for (uint i = 0; i < sampleCount; i++) {
        float2 Xi = hammersley(i, sampleCount);
        float3 H = importanceSampleGGX(Xi, a, N);
        float3 L = 2.0f * dot(V, H) * H - V;

        float VoH = saturate(dot(V, H));
        float NoL = saturate(L.z);
        float NoH = saturate(H.z);

        if (NoL > 0.0f) {
            float G = GDFG(NoV, NoL, a);
            float Gv = G * VoH / NoH;
            float Fc = pow(1 - VoH, 5.0f);
            r.x += Gv * (1 - Fc);
            r.y += Gv * Fc;
        }
    }
    return r * (1.0f / sampleCount);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[$ L_{DFG} $ 项的C++实现]

## 球谐函数

          符号             |           定义
:---------------------------:|:---------------------------|
$K^m_l$                      | 归一化因子
$P^m_l(x)$                   | 连带勒让德多项式
$y^m_l$                      | 球谐函数基
$L^m_l$                      | 定义在单位球面上的 $L(s)$ 函数的SH系数
[表 [shSymbols]: 球谐函数符号定义]

### 基函数

单位球面上点的球面参数化：

$$\begin{equation}
\{ x, y, z \} = \{ cos \phi sin \theta, sin \phi sin \theta, cos \theta \}
\end{equation}$$

复数球谐函数基由下式给出：

$$\begin{equation}
Y^m_l(\theta, \phi) = K^m_l e^{im\theta} P^{|m|}_l(cos \theta), l \in N, -l <= m <= l
\end{equation}$$

然而我们只需要实数基：

$$\begin{align*}
y^{m > 0}_l &= \sqrt{2} K^m_l cos(m \phi) P^m_l(cos \theta) \\
y^{m < 0}_l &= \sqrt{2} K^m_l sin(|m| \phi) P^{|m|}_l(cos \theta) \\
y^0_l &= K^0_l P^0_l(cos \theta)
\end{align*}$$

归一化因子由下式给出：

$$\begin{equation}
K^m_l = \sqrt{\frac{(2l + 1)(l - |m|)!}{4 \pi (l + |m|)!}}
\end{equation}$$

连带勒让德多项式 $P^{|m|}_l$ 可通过以下递推关系计算：

$$\begin{equation}\label{shRecursions}
P^0_0(x) = 1 \\
P^0_1(x) = x \\
P^l_l(x) = (-1)^l (2l - 1)!! (1 - x^2)^{\frac{l}{2}} \\
P^m_l(x) = \frac{((2l - 1) x P^m_{l - 1} - (l + m - 1) P^m_{l - 2})}{l - m} \\
\end{equation}$$

计算 $y^{|m|}_l$ 需要先计算 $P^{|m|}_l(z)$。使用方程 $\ref{shRecursions}$ 中的递推关系可以相当容易地完成。第三个递推关系可用于在表 [basisFunctions] 中"对角移动"，即计算 $y^0_0$、$y^1_1$、$y^2_2$ 等。然后第四个递推关系可用于垂直移动。

  频带索引 |  基函数 $-l <= m <= l$
:-----------:|:---------------------------------:|
$l = 0$      | $y^0_0$
$l = 1$      | $y^{-1}_1$ $y^0_1$ $y^1_1$
$l = 2$      | $y^{-2}_2$ $y^{-1}_2$ $y^0_2$ $y^1_2$ $y^2_2$
[表 [basisFunctions]: 各频带的基函数]

递归计算三角函数项也相当简单：

$$\begin{align*}
C_m &\equiv cos(m \phi)sin(\theta)^m \\
S_m &\equiv sin(m \phi)sin(\theta)^m \\
\{ x, y, z \} &= \{ cos \phi sin \theta, sin \phi sin \theta, cos \theta \}
\end{align*}$$

使用三角函数的和角恒等式：

$$\begin{align*}
cos(m \phi + \phi) &= cos(m \phi) cos(\phi) - sin(m \phi) sin(\phi) \Leftrightarrow C_{m + 1} = x C_m - y S_m \\
sin(m \phi + \phi) &= sin(m \phi) cos(\phi) + cos(m \phi) sin(\phi) \Leftrightarrow S_{m + 1} = x S_m - y C_m
\end{align*}$$

代码清单 [nonNormalizedSHBasis] 显示了计算非归一化SH基 $\frac{y^m_l(s)}{\sqrt{2} K^m_l}$ 的C++代码：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
static inline size_t SHindex(ssize_t m, size_t l) {
    return l * (l + 1) + m;
}

void computeShBasis(
        double* const SHb,
        size_t numBands,
        const vec3& s)
{
    // handle m=0 separately, since it produces only one coefficient
    double Pml_2 = 0;
    double Pml_1 = 1;
    SHb[0] =  Pml_1;
    for (ssize_t l = 1; l < numBands; l++) {
        double Pml = ((2 * l - 1) * Pml_1 * s.z - (l - 1) * Pml_2) / l;
        Pml_2 = Pml_1;
        Pml_1 = Pml;
        SHb[SHindex(0, l)] = Pml;
    }
    double Pmm = 1;
    for (ssize_t m = 1; m < numBands ; m++) {
        Pmm = (1 - 2 * m) * Pmm;
        double Pml_2 = Pmm;
        double Pml_1 = (2 * m + 1)*Pmm*s.z;
        // l == m
        SHb[SHindex(-m, m)] = Pml_2;
        SHb[SHindex( m, m)] = Pml_2;
        if (m + 1 < numBands) {
            // l == m+1
            SHb[SHindex(-m, m + 1)] = Pml_1;
            SHb[SHindex( m, m + 1)] = Pml_1;
            for (ssize_t l = m + 2; l < numBands; l++) {
                double Pml = ((2 * l - 1) * Pml_1 * s.z - (l + m - 1) * Pml_2)
                        / (l - m);
                Pml_2 = Pml_1;
                Pml_1 = Pml;
                SHb[SHindex(-m, l)] = Pml;
                SHb[SHindex( m, l)] = Pml;
            }
        }
    }
    double Cm = s.x;
    double Sm = s.y;
    for (ssize_t m = 1; m <= numBands ; m++) {
        for (ssize_t l = m; l < numBands ; l++) {
            SHb[SHindex(-m, l)] *= Sm;
            SHb[SHindex( m, l)] *= Cm;
        }
        double Cm1 = Cm * s.x - Sm * s.y;
        double Sm1 = Sm * s.x + Cm * s.y;
        Cm = Cm1;
        Sm = Sm1;
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [nonNormalizedSHBasis]: 计算非归一化SH基的C++实现]

前3个频带的归一化SH基函数 $y^m_l(s)$：

   频带   |               $m = -2$               |                $m = -1$               |                      $m = 0$                        |                $m = 1$                |                    $m = 2$                    |
:-------:|:------------------------------------:|:-------------------------------------:|:---------------------------------------------------:|:-------------------------------------:|:---------------------------------------------:|
$l = 0$  |                                      |                                       | $\frac{1}{2}\sqrt{\frac{1}{\pi}}$                   |                                       |                                               |
$l = 1$  |                                      | $-\frac{1}{2}\sqrt{\frac{3}{\pi}}y$   | $\frac{1}{2}\sqrt{\frac{3}{\pi}}z$                  | $-\frac{1}{2}\sqrt{\frac{3}{\pi}}x$   |                                               |
$l = 2$  | $\frac{1}{2}\sqrt{\frac{15}{\pi}}xy$ | $-\frac{1}{2}\sqrt{\frac{15}{\pi}}yz$ | $\frac{1}{4}\sqrt{\frac{5}{\pi}}(2z^2 - x^2 - y^2)$ | $-\frac{1}{2}\sqrt{\frac{15}{\pi}}xz$ | $\frac{1}{4}\sqrt{\frac{15}{\pi}}(x^2 - y^2)$ |
[表 [basisFunctions]: 各频带的归一化基函数]

### 分解与重建

定义在球面上的函数 $L(s)$ 按如下方式投影到SH基上：

$$\begin{equation}
L^m_l = \int_\Omega L(s) y^m_l(s) ds \\
L^m_l = \int_{\theta = 0}^{\pi} \int_{\phi = 0}^{2\pi} L(\theta, \phi) y^m_l(\theta, \phi) sin \theta d\theta d\phi
\end{equation}$$

注意，每个 $L^m_l$ 是一个包含3个值的向量，分别对应RGB三个颜色通道。

从SH系数进行的逆变换（即重建或渲染）由下式给出：

$$\begin{equation}
\hat{L}(s) = \sum_l \sum_{m = -l}^l L^m_l y^m_l(s)
\end{equation}$$

### $\left< cos \theta \right>$ 的分解

由于 $\left< cos \theta \right>$ 不依赖于 $\phi$（方位角独立性），积分简化为：

$$\begin{align*}
C^0_l &= 2\pi \int_0^{\pi} \left< cos \theta \right> y^0_l(\theta) sin \theta d\theta \\
C^0_l &= 2\pi K^m_l \int_0^{\frac{\pi}{2}} P^0_l(cos \theta) cos \theta sin \theta d\theta \\
C^m_l &= 0, m != 0
\end{align*}$$

[#Ramamoorthi01] 中描述了该积分的解析解：

$$\begin{align*}
C_1 &= \sqrt{\frac{\pi}{3}} \\
C_{odd} &= 0 \\
C_{l, even} &= 2\pi \sqrt{\frac{2l + 1}{4\pi}} \frac{(-1)^{\frac{l}{2} - 1}}{(l + 2)(l - 1)} \frac{l!}{2^l (\frac{l!}{2})^2}
\end{align*}$$

前几个系数为：

$$\begin{align*}
C_0 &= +0.88623 \\
C_1 &= +1.02333 \\
C_2 &= +0.49542 \\
C_3 &= +0.00000 \\
C_4 &= -0.11078
\end{align*}$$

只需要很少的系数就能合理逼近 $\left< cos \theta \right>$，如图 [shCosThetaApprox] 所示。

<div align="center"><img src="https://google.github.io/filament/images/chart_sh_cos_thera_approx.png" alt="Figure [shCosThetaApprox]: Approximation of $cos \theta$ with SH coefficients"/></div>

### 卷积

具有圆对称性的核 $h$ 的卷积可以直接且方便地在SH空间中进行：

$$\begin{equation}
(h * f)^m_l = \sqrt{\frac{4\pi}{2l + 1}} h^0_l(s) f^m_l(s)
\end{equation}$$

方便的是，$\sqrt{\frac{4\pi}{2l + 1}} = \frac{1}{K^0_l}$，因此在实际中我们将 $C_l$ 预乘 $\frac{1}{K^0_l}$，得到更简洁的表达式：

$$\begin{equation}
\hat{C}_{l, even} = 2\pi \frac{(-1)^{\frac{l}{2} - 1}}{(l + 2)(l - 1)} \frac{l!}{2^l (\frac{l!}{2})^2} \\
\hat{C}_1 = \frac{2\pi}{3}
\end{equation}$$

以下是计算 $\hat{C}_l$ 的C++代码：

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
static double factorial(size_t n, size_t d = 1);

// < cos(theta) > SH coefficients pre-multiplied by 1 / K(0,l)
double computeTruncatedCosSh(size_t l) {
    if (l == 0) {
        return M_PI;
    } else if (l == 1) {
        return 2 * M_PI / 3;
    } else if (l & 1) {
        return 0;
    }
    const size_t l_2 = l / 2;
    double A0 = ((l_2 & 1) ? 1.0 : -1.0) / ((l + 2) * (l - 1));
    double A1 = factorial(l, l_2) / (factorial(l_2) * (1 << l));
    return 2 * M_PI * A0 * A1;
}

// returns n! / d!
double factorial(size_t n, size_t d ) {
   d = std::max(size_t(1), d);
   n = std::max(size_t(1), n);
   double r = 1.0;
   if (n == d) {
       // intentionally left blank
   } else if (n > d) {
       for ( ; n>d ; n--) {
           r *= n;
       }
   } else {
       for ( ; d>n ; d--) {
           r *= d;
       }
       r = 1.0 / r;
   }
   return r;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## Mitsuba样本验证场景

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
&lt;scene version="0.5.0"&gt;
    &lt;integrator type="path"/&gt;

    &lt;shape type="serialized" id="sphere_mesh"&gt;
        &lt;string name="filename" value="plastic_sphere.serialized"/&gt;
        &lt;integer name="shapeIndex" value="0"/&gt;

        &lt;bsdf type="roughplastic"&gt;
            &lt;string name="distribution" value="ggx"/&gt;
            &lt;float name="alpha" value="0.0"/&gt;
            &lt;srgb name="diffuseReflectance" value="0.81, 0.0, 0.0"/&gt;
        &lt;/bsdf&gt;
    &lt;/shape&gt;

    &lt;emitter type="envmap"&gt;
        &lt;string name="filename" value="../../environments/office/office.exr"/&gt;
        &lt;float name="scale" value="35000.0" /&gt;
        &lt;boolean name="cache" value="false" /&gt;
    &lt;/emitter&gt;

    &lt;emitter type="directional"&gt;
        &lt;vector name="direction" x="-1" y="-1" z="1" /&gt;
        &lt;rgb name="irradiance" value="120000.0, 115200.0, 114000.0" /&gt;
    &lt;/emitter&gt;

    &lt;sensor type="perspective"&gt;
        &lt;float name="farClip" value="12.0"/&gt;
        &lt;float name="focusDistance" value="4.1"/&gt;
        &lt;float name="fov" value="45"/&gt;
        &lt;string name="fovAxis" value="y"/&gt;
        &lt;float name="nearClip" value="0.01"/&gt;
        &lt;transform name="toWorld"&gt;

            &lt;lookat target="0, 0, 0" origin="0, 0, -3.1" up="0, 1, 0"/&gt;
        &lt;/transform&gt;

        &lt;sampler type="ldsampler"&gt;
            &lt;integer name="sampleCount" value="256"/&gt;
        &lt;/sampler&gt;

        &lt;film type="ldrfilm"&gt;
            &lt;integer name="height" value="1440"/&gt;
            &lt;integer name="width" value="2048"/&gt;
            &lt;float name="exposure" value="-15.23" /&gt;
            &lt;rfilter type="gaussian"/&gt;
        &lt;/film&gt;
    &lt;/sensor&gt;
&lt;/scene&gt;
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## 基于视锥体素的光源分配

使用两个计算着色器可以在GPU上实现光源到视锥体素的分配。第一个（如代码清单 [froxelGeneration] 所示）在SSBO中创建视锥体素数据（每个视锥体素包含4个平面以及最小Z和最大Z值），只需运行一次。该着色器需要以下uniform变量：

投影矩阵
:   用于渲染场景的投影矩阵（视图空间到裁剪空间的变换）。

逆投影矩阵
:   用于渲染场景的投影矩阵的逆矩阵（裁剪空间到视图空间的变换）。

深度参数
:   $-log2(\frac{z_{lighnear}}{z_{far}}) \frac{1}{maxSlices-1}$，最大深度切片数，近裁剪面和远裁剪面。

裁剪空间大小
:   $\frac{F_x \times F_r}{w} \times 2$，其中 $F_x$ 为X轴上的瓦片数，$F_r$ 为瓦片的像素分辨率，w 为渲染目标的像素宽度。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#version 310 es

precision highp float;
precision highp int;


#define FROXEL_RESOLUTION 80u

layout(local_size_x = 1, local_size_y = 1, local_size_z = 1) in;

layout(location = 0) uniform mat4 projectionMatrix;
layout(location = 1) uniform mat4 projectionInverseMatrix;
layout(location = 2) uniform vec4 depthParams; // index scale, index bias, near, far
layout(location = 3) uniform float clipSpaceSize;

struct Froxel {
    // NOTE: the planes should be stored in vec4[4] but the
    // Adreno shader compiler has a bug that causes the data
    // to not be read properly inside the loop
    vec4 plane0;
    vec4 plane1;
    vec4 plane2;
    vec4 plane3;
    vec2 minMaxZ;
};

layout(binding = 0, std140) writeonly restrict buffer FroxelBuffer {
    Froxel data[];
} froxels;

shared vec4 corners[4];
shared vec2 minMaxZ;

vec4 projectionToView(vec4 p) {
    p = projectionInverseMatrix * p;
    return p / p.w;
}

vec4 createPlane(vec4 b, vec4 c) {
    // standard plane equation, with a at (0, 0, 0)
    return vec4(normalize(cross(c.xyz, b.xyz)), 1.0);
}

void main() {
    uint index = gl_WorkGroupID.x + gl_WorkGroupID.y * gl_NumWorkGroups.x +
            gl_WorkGroupID.z * gl_NumWorkGroups.x * gl_NumWorkGroups.y;

    if (gl_LocalInvocationIndex == 0u) {
        // first tile the screen and build the frustum for the current tile
        vec2 renderTargetSize = vec2(FROXEL_RESOLUTION * gl_NumWorkGroups.xy);
        vec2 frustumMin = vec2(FROXEL_RESOLUTION * gl_WorkGroupID.xy);
        vec2 frustumMax = vec2(FROXEL_RESOLUTION * (gl_WorkGroupID.xy + 1u));

        corners[0] = vec4(
            frustumMin.x / renderTargetSize.x * clipSpaceSize - 1.0,
            (renderTargetSize.y - frustumMin.y) / renderTargetSize.y
				    * clipSpaceSize - 1.0,
            1.0,
            1.0
        );
        corners[1] = vec4(
            frustumMax.x / renderTargetSize.x * clipSpaceSize - 1.0,
            (renderTargetSize.y - frustumMin.y) / renderTargetSize.y
				    * clipSpaceSize - 1.0,
            1.0,
            1.0
        );
        corners[2] = vec4(
            frustumMax.x / renderTargetSize.x * clipSpaceSize - 1.0,
            (renderTargetSize.y - frustumMax.y) / renderTargetSize.y
				    * clipSpaceSize - 1.0,
            1.0,
            1.0
        );
        corners[3] = vec4(
            frustumMin.x / renderTargetSize.x * clipSpaceSize - 1.0,
            (renderTargetSize.y - frustumMax.y) / renderTargetSize.y
				    * clipSpaceSize - 1.0,
            1.0,
            1.0
        );

        uint froxelSlice = gl_WorkGroupID.z;
        minMaxZ = vec2(0.0, 0.0);
        if (froxelSlice > 0u) {
            minMaxZ.x = exp2((float(froxelSlice) - depthParams.y) * depthParams.x)
                    * depthParams.w;
        }
        minMaxZ.y = exp2((float(froxelSlice + 1u) - depthParams.y) * depthParams.x)
                * depthParams.w;
    }

    if (gl_LocalInvocationIndex == 0u) {
        vec4 frustum[4];
        frustum[0] = projectionToView(corners[0]);
        frustum[1] = projectionToView(corners[1]);
        frustum[2] = projectionToView(corners[2]);
        frustum[3] = projectionToView(corners[3]);

        froxels.data[index].plane0 = createPlane(frustum[0], frustum[1]);
        froxels.data[index].plane1 = createPlane(frustum[1], frustum[2]);
        froxels.data[index].plane2 = createPlane(frustum[2], frustum[3]);
        froxels.data[index].plane3 = createPlane(frustum[3], frustum[0]);
        froxels.data[index].minMaxZ = minMaxZ;
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [froxelGeneration]: 视锥体素数据生成的GLSL实现（计算着色器）]

第二个计算着色器（如代码清单 [froxelEvaluation] 所示）每帧运行（如果相机和/或光源发生变化），将所有光源分配到各自的视锥体素。该着色器仅依赖几个uniform变量（点光源/聚光灯数量和视图矩阵）和四个SSBO：

光源索引缓冲
:   对于每个视锥体素，记录影响该视锥体素的每个光源的索引。点光源的索引优先写入，如果剩余空间足够，再写入聚光灯的索引。值为 0x7fffffffu 的哨兵值用于分隔点光源和聚光灯，和/或标记视锥体素光源列表的结束。每个视锥体素有一个最大光源数（点光源 + 聚光灯）。

点光源缓冲
:   描述场景中点光源的结构体数组。

聚光灯缓冲
:   描述场景中聚光灯的结构体数组。

视锥体素缓冲
:   由上一个计算着色器创建的视锥体素列表，以平面表示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#version 310 es
precision highp float;
precision highp int;

#define LIGHT_BUFFER_SENTINEL 0x7fffffffu
#define MAX_FROXEL_LIGHT_COUNT 32u

#define THREADS_PER_FROXEL_X 8u
#define THREADS_PER_FROXEL_Y 8u
#define THREADS_PER_FROXEL_Z 1u
#define THREADS_PER_FROXEL (THREADS_PER_FROXEL_X * \
        THREADS_PER_FROXEL_Y * THREADS_PER_FROXEL_Z)

layout(local_size_x = THREADS_PER_FROXEL_X,
       local_size_y = THREADS_PER_FROXEL_Y,
       local_size_z = THREADS_PER_FROXEL_Z) in;

// x = point lights, y = spot lights
layout(location = 0) uniform uvec2 totalLightCount;
layout(location = 1) uniform mat4 viewMatrix;

layout(binding = 0, packed) writeonly restrict buffer LightIndexBuffer {
    uint index[];
} lightIndexBuffer;

struct PointLight {
    vec4 positionFalloff; // x, y, z, falloff
    vec4 colorIntensity;  // r, g, b, intensity
    vec4 directionIES;    // dir x, dir y, dir z, IES profile index
};

layout(binding = 1, std140) readonly restrict buffer PointLightBuffer {
    PointLight lights[];
} pointLights;

struct SpotLight {
    vec4 positionFalloff; // x, y, z, falloff
    vec4 colorIntensity;  // r, g, b, intensity
    vec4 directionIES;    // dir x, dir y, dir z, IES profile index
    vec4 angle;           // angle scale, angle offset, unused, unused
};

layout(binding = 2, std140) readonly restrict buffer SpotLightBuffer {
    SpotLight lights[];
} spotLights;

struct Froxel {
    // NOTE: the planes should be stored in vec4[4] but the
    // Adreno shader compiler has a bug that causes the data
    // to not be read properly inside the loop
    vec4 plane0;
    vec4 plane1;
    vec4 plane2;
    vec4 plane3;
    vec2 minMaxZ;
};

layout(binding = 3, std140) readonly restrict buffer FroxelBuffer {
    Froxel data[];
} froxels;

shared uint groupLightCounter;
shared uint groupLightIndexBuffer[MAX_FROXEL_LIGHT_COUNT];

float signedDistanceFromPlane(vec4 p, vec4 plane) {
    // plane.w == 0.0, simplify computation
    return dot(plane.xyz, p.xyz);
}

void synchronize() {
    memoryBarrierShared();
    barrier();
}

void main() {
    if (gl_LocalInvocationIndex == 0u) {
        groupLightCounter = 0u;
    }
    memoryBarrierShared();

    uint froxelIndex = gl_WorkGroupID.x + gl_WorkGroupID.y * gl_NumWorkGroups.x +
            gl_WorkGroupID.z * gl_NumWorkGroups.x * gl_NumWorkGroups.y;
    Froxel current = froxels.data[froxelIndex];

    uint offset = gl_LocalInvocationID.x +
		        gl_LocalInvocationID.y * THREADS_PER_FROXEL_X;
    for (uint i = 0u; i < totalLightCount.x &&
			    groupLightCounter < MAX_FROXEL_LIGHT_COUNT &&
            offset + i < totalLightCount.x; i += THREADS_PER_FROXEL) {

        uint currentLight = offset + i;

        vec4 center = pointLights.lights[currentLight].positionFalloff;
        center.xyz = (viewMatrix * vec4(center.xyz, 1.0)).xyz;
        float r = inversesqrt(center.w);

        if (-center.z + r > current.minMaxZ.x &&
                -center.z - r <= current.minMaxZ.y) {
            if (signedDistanceFromPlane(center, current.plane0) < r &&
                signedDistanceFromPlane(center, current.plane1) < r &&
                signedDistanceFromPlane(center, current.plane2) < r &&
                signedDistanceFromPlane(center, current.plane3) < r) {

                uint index = atomicAdd(groupLightCounter, 1u);
                groupLightIndexBuffer[index] = currentLight;
            }
        }
    }

    synchronize();

    uint pointLightCount = groupLightCounter;
    offset = froxelIndex * MAX_FROXEL_LIGHT_COUNT;

    for (uint i = gl_LocalInvocationIndex; i < pointLightCount;
            i += THREADS_PER_FROXEL) {
        lightIndexBuffer.index[offset + i] = groupLightIndexBuffer[i];
    }

    if (gl_LocalInvocationIndex == 0u) {
        if (pointLightCount < MAX_FROXEL_LIGHT_COUNT) {
            lightIndexBuffer.index[offset + pointLightCount] = LIGHT_BUFFER_SENTINEL;
        }
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[代码清单 [froxelEvaluation]: 光源分配到视锥体素的GLSL实现（计算着色器）]

# 修订历史

2019年2月20日：布料着色
 - 从布料BRDF中移除了菲涅耳项
 - 移除了布料DFG近似，替换为DFG查找表中的新通道

2018年8月21日：多次散射
 - 新增关于如何补偿单次散射BRDF中能量损失的 [Energy loss in specular reflectance] 章节

2018年8月17日：镜面反射颜色
 - 新增 [Specular color] 章节，解释各种金属的基础颜色如何计算

2018年8月15日：菲涅耳
 - 在 [Fresnel (specular F)] 章节中增加了菲涅耳效应的描述

2018年8月9日：照明
 - 增加了关于预曝光光源的说明

2018年8月7日：布料模型
 - 增加了"Charlie"NDF的描述

2018年8月3日：首个公开版本

# 参考文献

[#Ashdown98]: Ian Ashdown. 1998. Parsing the IESNA LM-63 photometric data file. http://lumen.iee.put.poznan.pl/kw/iesna.txt

[#Ashikhmin00]: Michael Ashikhmin, Simon Premoze and Peter Shirley. A Microfacet-based BRDF Generator. *SIGGRAPH '00 Proceedings*, 65-74.

[#Ashikhmin07]: Michael Ashikhmin and Simon Premoze. 2007. Distribution-based BRDFs.

[#Burley12]: Brent Burley. 2012. Physically Based Shading at Disney. *Physically Based Shading in Film and Game Production, ACM SIGGRAPH 2012 Courses*.

[#Estevez17]: Alejandro Conty Estevez and Christopher Kulla. 2017. Production Friendly Microfacet Sheen BRDF. *ACM SIGGRAPH 2017*.

[#Hammon17]: Earl Hammon. 217. PBR Diffuse Lighting for GGX+Smith Microsurfaces. *GDC 2017*.

[#Heitz14]: Eric Heitz. 2014. Understanding the Masking-Shadowing Function
in Microfacet-Based BRDFs. *Journal of Computer Graphics Techniques*, 3 (2).

[#Heitz16]: Eric Heitz et al. 2016. Multiple-Scattering Microfacet BSDFs with the Smith Model. *ACM SIGGRAPH 2016*.

[#Hill12]: Colin Barré-Brisebois and Stephen Hill. 2012. Blending in Detail. http://blog.selfshadow.com/publications/blending-in-detail/

[#Karis13a]: Brian Karis. 2013. Specular BRDF Reference. http://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html

[#Karis13b]: Brian Karis, 2013. Real Shading in Unreal Engine 4. https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf

[#Karis14]: Brian Karis. 2014. Physically Based Shading on Mobile. https://www.unrealengine.com/blog/physically-based-shading-on-mobile

[#Kelemen01]: Csaba Kelemen et al. 2001. A Microfacet Based Coupled Specular-Matte BRDF Model with Importance Sampling. *Eurographics Short Presentations*.

[#Krystek85]: M. Krystek. 1985. An algorithm to calculate correlated color temperature. *Color Research & Application*, 10 (1), 38–40.

[#Krivanek08]: Jaroslave Krivànek and Mark Colbert. 2008. Real-time Shading with Filtered Importance Sampling. *Eurographics Symposium on Rendering 2008*, Volume 27, Number 4.

[#Kulla17]: Christopher Kulla and Alejandro Conty. 2017. Revisiting Physically Based Shading at Imageworks. *ACM SIGGRAPH 2017*

[#Lagarde14]: Sébastien Lagarde and Charles de Rousiers. 2014. Moving Frostbite to PBR. *Physically Based Shading in Theory and Practice, ACM SIGGRAPH 2014 Courses*.

[#Lagarde18]: Sébastien Lagarde and Evgenii Golubev. 2018. The road toward unified rendering with Unity's high definition rendering pipeline. *Advances in Real-Time Rendering in Games, ACM SIGGRAPH 2018 Courses*.

[#Lazarov13]: Dimitar Lazarov. 2013. Physically-Based Shading in Call of Duty: Black Ops. *Physically Based Shading in Theory and Practice, ACM SIGGRAPH 2013 Courses*.

[#McAuley15]: Stephen McAuley. 2015. Rendering the World of Far Cry 4. *GDC 2015*.

[#McGuire10]: Morgan McGuire. 2010. Ambient Occlusion Volumes. *High Performance Graphics*.

[#Narkowicz14]: Krzysztof Narkowicz. 2014. Analytical DFG Term for IBL. https://knarkowicz.wordpress.com/2014/12/27/analytical-dfg-term-for-ibl

[#Neubelt13]: David Neubelt and Matt Pettineo. 2013. Crafting a Next-Gen Material Pipeline for The Order: 1886. *Physically Based Shading in Theory and Practice, ACM SIGGRAPH 2013 Courses*.

[#Oren94]: Michael Oren and Shree K. Nayar. 1994. Generalization of lambert's reflectance model. *SIGGRAPH*, 239–246. ACM.

[#Pattanaik00]: Sumanta Pattanaik00 et al. 2000. Time-Dependent Visual Adaptation
For Fast Realistic Image Display. *SIGGRAPH '00 Proceedings of the 27th annual conference on Computer graphics and interactive techniques*, 47-54.

[#Ramamoorthi01]: Ravi Ramamoorthi and Pat Hanrahan. 2001. On the relationship between radiance and irradiance: determining the illumination from images of a convex Lambertian object. *Journal of the Optical Society of America*, Volume 18, Number 10, October 2001.

[#Revie12]: Donald Revie. 2012.  Implementing Fur in Deferred Shading. *GPU Pro 2*, Chapter 2.

[#Russell15]: Jeff Russell. 2015. Horizon Occlusion for Normal Mapped Reflections. http://marmosetco.tumblr.com/post/81245981087

[#Schlick94]: Christophe Schlick. 1994. An Inexpensive BRDF Model for Physically-Based Rendering. *Computer Graphics Forum*, 13 (3), 233–246.

[#Walter07]: Bruce Walter et al. 2007. Microfacet Models for Refraction through Rough Surfaces. *Proceedings of the Eurographics Symposium on Rendering*.
