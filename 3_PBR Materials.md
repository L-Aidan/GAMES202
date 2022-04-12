# GAMES202笔记

# PBR Materials

Physically-Based Rendering 基于物理的渲染

PBR材质包括表面材质和体积材质。

表面材质主要包括微表面模型和迪士尼原则的模型。

体积材质主要聚焦于快速近似的单词和多次散射（云，头发，皮肤等）

## Microfacet BRDF

$$
f(i,o) = \frac{F(i,h)G(i,o,h)D(h)}{4(n,i)(n,o)}
$$

### Fresnel Term

菲涅尔项：描述材质在不同角度上反射出的能量的百分比，可以简单理解为反射率。

当入射方向近乎于法线垂直时，反射率较低；而当入射方向近乎是grazing angle的时候，反射率较高。生动的例子是看向近处的湖面，会看到水面下的物体，而看向远处的湖面，则会看到远方物体在湖面的倒影。这里考虑光线是可逆的。

![image-20220411230024709](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204112300810.png)

不同材质的Fresnel曲线是不同的，Fresnel项有一个简单的估计：Schlick估计：
$$
R(\theta) = R_0 + (1 - R_0)(1 - cos\theta)^5
$$
其中$R_0$是基础反射率，$\theta$是入射方向和半程向量的夹角。

## Disney principled BRDF

### Shading with microfacet BRDFs under polygonal lighting

