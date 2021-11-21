# Environment Lighting

## Approximation in RTR

本节用到的一个数学近似。

有很多不等式，在一定情况下可以看作等号成立。由于实时渲染只要求看着真实，而不必满足绝对的真实，因此这种近似可以应用在渲染方程中。
$$
\int_{\Omega}f(x)g(x)dx \approx \frac{\int_{\Omega}f(x)dx}{\int_{\Omega}dx} \cdot \int_{\Omega}g(x)dx \tag{3.3}
$$
(数学含义或者证明待补充)

当满足以下其中一个条件时，两边可近似相等

1. Small support 

   指的是公式中的积分域 $\Omega$ 尽可能小的时候。因此如果只考虑直接光照，在点光源和方向光源的情况下， $\Omega$ 就只表示光源方向的立体角，就会很小，如果考虑一个点的镜面反射，则渲染方程中的 $\Omega$ 就只是镜面反射方向上的很小的立体角。

2. Smooth integrand

   指的是被积函数 $g(x)$ 在 $\Omega$ 内的变化很小，即频率低，“光滑”的情况下。因此如果G(x)是一个漫反射的BRDF，那么它值的变化就不大。

当满足上面其中一个条件时，对于渲染方程来说，把 $V()$ 看作 $f(x)$ ,其他项看作 $g(x)$，渲染方程就可以写成：
$$
L_0(p,w_0) \approx \frac{\int_{\Omega^+}V(p,w_i) dw_i}{\int_{\Omega^+}dw_i} \cdot \int_{\Omega^+} L_i(p,w_i) f_r(p,w_i,w_o) cos\theta_i dw_i
$$
这个积分会在环境光遮蔽（Ambient Occlusions）章节用到。

## IBL(Image Based Lighting)

IBL，使用一张图像来，表示所有方向上的环境光。

一般有两种存储方式，spherical map 和 cube map

![image-20211121144410613](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111211444705.png)



## Shading from IBL

这里只考虑着色，不考虑阴影。渲染方程里也不考虑visibility项：
$$
L_0(p,w_0) = \int_{\Omega^+} L_i(p,w_i) f_r(p,w_i,w_o) cos\theta_i  dw_i \tag{1}
$$
解此方程可用蒙特卡洛积分采样，但实时渲染希望避免采样带来的耗费。
