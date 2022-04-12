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

这个公式每一项如何得到，又为什么这么组合，感觉需要看下原论文，总是要去看的。

### Fresnel Term

菲涅尔项：描述材质在不同角度上反射出的能量的百分比，可以简单理解为反射率。

当入射方向近乎于法线垂直时，反射率较低；而当入射方向近乎是grazing angle的时候，反射率较高。生动的例子是看向近处的湖面，会看到水面下的物体，而看向远处的湖面，则会看到远方物体在湖面的倒影。这里考虑光线是可逆的。

![image-20220411230024709](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204112300810.png)

不同材质的Fresnel曲线是不同的，Fresnel项有一个简单的估计：Schlick估计：
$$
R(\theta) = R_0 + (1 - R_0)(1 - cos\theta)^5
$$
其中$R_0$是基础反射率，$\theta$是入射方向和半程向量的夹角。

### Normal Distribution Function

D项，法线分布函数。每一个微表面的反射都可以看作是镜面反射。所有微表面法线的分布情况决定了这个着色点的反射情况。如果法线分布较为集中在几何的法线附近，那么整体的反射就接近于specular。如果法线分布比较杂乱，那么整体的反射就好比是diffuse。中间情况是glossy。

法线分布函数的值，$D(h)$，表示对于一个入射方向和一个出射方向，分布在他们半程向量上的法线比例有多大，也就是有是多大比例的光会反射到出射方向上。

#### Beckmann NDF

形状接近高斯函数。
$$
D(h) = \frac{e^{-\frac{tan^2\theta_h}{\alpha^2}}}{\pi\alpha^2cos^4\theta_h}
$$
$\alpha$是粗糙程度，这个值越小，表示越不粗糙（越接近镜面）。

$\theta_h$是半程向量与法线间的夹角。

而分子上的tan表示Beckmann是定义在坡度空间上的（slope space），$\theta_h$范围是[0,90]，而$tan\theta$范围是[0，∞]，导致了定义域是无限大的，无论$tan\theta$多大，$\theta$都不会超过90，并且$D(h)$都是有值的。

#### GGX

王希老师说，他也不知道为什么叫GGX。

GGX相对于Beckmann，有更明显的长尾效应，即，即使$\theta$接近了90°，函数仍然有一个不会太小的值，反映在效果上就是，高光会更加的柔和，diffuse范围更大：

![image-20220412210401584](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204122104660.png)

![image-20220412210619090](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204122106154.png)

#### GTR（Generalized Trowbridge-Reitz）

可调节参数，生成不同的函数。例如接近Beckmann或GGX

![image-20220412210925162](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204122109190.png)

### Shadowing-Masking Term

G项，几何项。反映微表面之间的相互遮挡的情况。

![image-20220412211034063](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204122110104.png)

入射光可能由于微表面的遮挡而不会按原来的方向反射，同时反射光也可能由于微表面的遮挡不会按照原来方向反射出，但我们却计算了这一部分的能量，所以正常颜色应该比我们计算出的要暗，所以需要几何项让结果变得暗一点。

### Kulla-Conty Approximation

上面的微表面模型能够得到很好的效果，但有一个致命问题，能量不守恒。因为它只考虑到了单次反射，而有部分光线是多次反射之后才反射出去的，要加上这部分能量才合理。

表面越粗糙，多次反射越多，能量损失越多。

在不考虑菲涅尔项的情况下，就不会有能量损失，因此如果用一个白的环境光去照亮物体，物体应该处处反射出白光，即与环境融为一体。这就是白炉测试：

![image-20220412212112972](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202204122121015.png)

如何补充这部分多次弹射的能量？对于一个出射方向，可以按原来的BRDF计算出该方向得到的能量E，那么1-E就是损失的能量，再想办法将这部分能量用一个BRDF表示出来，加到原来的BRDF上就可以。

对于一个出射方向$w_o$，计算它得到的能量，需要对所有入射方向做积分，不考虑能量的吸收，所以不考虑菲涅尔项:
$$
E(w_o) = \int_0^{2\pi}\int_0^{\frac{\pi}{2}} f(\theta,\phi,w_o)\mathrm{d}\theta \mathrm{d}\phi
$$
 换元 ，令$\mu = sin\theta$，得：
$$
E(\mu_o) = \int_0^{2\pi}\int_0^1 f(\mu_o,\mu_i,\phi)\mu_i\mathrm{d}\mu_i \mathrm{d}\phi
$$
那么设我们补充的BRDF为$f_{ms}$，那么：
$$
1 - E(\mu_o) = \int_0^{2\pi}\int_0^1 f_{ms}\mu_i\mathrm{d}\mu_i \mathrm{d}\phi
$$
可以设计出一个表达式使上式成立：
$$
f_{ms}(\mu_o,\mu_i) = \frac{(1-E(\mu_o))(1 - E(\mu_i))}{\pi(1 - E_{avg})}, E_{avg} = 2\int_0^1E(\mu)\mu \mathrm{d}\mu
$$
其中$E(\mu)$和$E_{avg}$可以用预计算打表，这样可以实时得到。

$E(\mu)$依赖于$\mu$和roughness， $E_{avg}$只依赖于roughness。



考虑到有颜色的情况（有菲涅尔项），由于是多次反射，所以定义一个平均的反射率$F_{avg}$，而上面提到的$E_{avg}$表示平均情况下，给定一个入射方向，单次反射出去的能量。

$E_{avg}$计算公式为$E_\mu$的cos weighted平均数，首先光线是可逆的，前面提到的$E_\mu$是某个出射方向得到的来自所有入射方向的能量的比例，那么对于一个入射方向的光，它反射到各个出射方向上能量的比例之和也是$E_\mu$。（假设100条入射光，每条能量为1，每条入射光反射到这个出射方向的比例都是0.08，那么出射方向得到的总能量为0.8，也就是比例是0.8。那么假设一条入射光能量为1，能量会反射到100个出射方向上，每个比例也是0.08，加起来反射出去的能量也是0.8。二者数值相等）。

我们要计算平均情况下的单次反射出的能量，也就是对所有入射角度进行平均，可能需要加权平均，那么为什么这个加权平均的公式是$E_{avg} = 2\int_0^1E(\mu)\mu \mathrm{d}\mu$？

1.首先为什么是对$E_\mu$而不是对$E(\theta,\phi)$进行加权平均，$E(\theta,\phi)$才真正代表一个入射方向的单次反射能量，而$E_\mu$表示某个$\theta$下的$E(\theta,\phi)$，因为各向同性，所以一圈的$E(\theta,\phi)$都是相等的。

2.为什么是cos加权平均而不是简单平均。

抛开这个问题继续向下，有了平均反射率$F_{avg}$和平均单次反射的能量$E_{avg}$。那么单次反射出去的能量为$F_{avg}E_{avg}$，两次反射出去的能量为$F_{avg}(1 - E_{avg})F_{avg}E_{avg}$，继续向下递推，将所有的加起来就得到了有菲涅尔项（有能量损失）的情况下，我们能得到的能量的比例：
$$
f_{add} = \frac{F_{avg}E_{avg}}{1 - F_{avg}(1-E_{avg})}
$$
带有菲涅尔项的$f_{micro}$已经考虑了能量损失，而表示多次反射的$f_{ms}$没有考虑到能量损失，因此将其乘上我们得到的这个比例，再与$f_{micro}$相加，最终的BRDF: $f_{final} = f_{micro} + f_{add}f_{ms} $。



## Disney principled BRDF

### Shading with microfacet BRDFs under polygonal lighting

