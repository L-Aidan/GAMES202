# GAMES202 笔记

## CG基础回顾

### 基础GPU渲染管线

（待补充）

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202110282255523.png" alt="image-20211028225552443" style="zoom: 67%;" />

### OpenGL

### OpenGL 着色语言(GLSL)

### 渲染方程

$$
L_0(p,w_0) = L_e(p,w_0) + \int_{H^2} f_r(p,w_i\rightarrow w_o) L_i(p,w_i) cos\theta_i dw_i \tag{3.1}
$$

实时渲染中的渲染方程通常写为：
$$
L_0(p,w_0) = L_e(p,w_0) + \int_{\Omega^+} L_i(p,w_i) f_r(p,w_i,w_o) cos\theta_i V(p,w_i) dw_i \tag{3.2}
$$
这里公式中的 $L_i( p ,w_i)$ 与 $V( p ,w_i)$ 相乘才等同于上一公式的 $L_i( p ,w_i)$，表示将①光源是否存在 和②光源能否打到 $ p$ 点 两项分开考虑

## Shadow Mapping

### 基本思想

这里只考虑点光源或者平行光源。

两趟算法：以点光源为例，第一趟，将摄像机放在点光源的位置，渲染出一张shadow map，其实是depth map，用来记录光源看到的最近的点的深度。第二趟，用实际的观察点渲染场景，对摄像机看到的每一个点 $p$ ，通过矩阵变换，将其投影到上一张shadow map中，查询投影坐标上的**depth值**， 和 **$p$ 点到光源的距离**做比较，如果depth值<距离，表明有物体挡在了光源和 $p$ 点之间，则 $p$ 点应该显示阴影。

需要注意：shadow map中保存的是场景中顶点的真实深度（点到光源的真实距离）还是在裁剪空间（视锥体frustum需要经过仿射变换变为正方体形状的裁剪空间）内的z值？进行深度比较时，要保证比较双方的意义是相同的。

平行光源和点光源的区别在于观察空间不同，平行光源是一个长方体，点光源是平截头体。

### 存在的问题

#### 1. Self occlusion（自遮挡）

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012230917.png" alt="image-20211101223001812" style="zoom: 50%;" />

由于shadow map本身是有分辨率的，所以它每个像素中用一个值表示一片区域的深度值。对地面上的一个点 $p$ ，如果它对应的shadow map中的深度值小于 $p$ 点到光源的距离，即 z < dis 时，就会被判定为遮挡。

解决方法：我们可以设定一个bias来抵消上面提到的两个距离的差值，当z + bias < dis 时才判定为遮挡。考虑到在 $p$ 点，如果光源是近乎垂直打在地面上的，那儿shadow map中的值和 $p$ 的深度值差异会很小，bias就可以很小。如果光束方向近乎平行于地面（grazing angle），那么该差异可能很大，bias也需要较大。因此可以用入射光的角度$\theta$来控制bias的大小。

但设置bias会带来另一个问题：detached shadow。

#### 2. Detached shadow

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012248429.png" alt="image-20211101224834349" style="zoom: 50%;" />

设置bias后，有可能出现这种情况，即本来判定为遮挡的点，因为bias的存在，被判定为不遮挡。那么当bias较大时，就会出现上图的状况。

#### 3. Aliasing（走样）

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111022023700.png" alt="image-20211102202317556" style="zoom: 67%;" />

还是由于shadow map的每一个像素代表了一块区域，那么当遮挡物离 $p$ 点较远时，就会出现上图中左边的锯齿现象。

解决方法：cascaded shadow map等

### Second Depth Shadow Mapping

(试图解决自遮挡和detached shadow的方法，但因耗费太高而没有人用)

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012256008.png" alt="image-20211101225651952" style="zoom:50%;" />

这个方法和最基础的Shadow Mapping方法的区别就是，它采用最小深度和次小深度的均值来作为shadow map中的值。因为不采用bias，所以不会出现detached shadow；如果最小深度和次小深度的值相差较大，也会一定程度减少自遮挡（个人理解）。但缺点就是①：需要模型是一个“盒子”类的东西（watertight）②：保存每个像素的最小深度和次小深度，尽管时间复杂度相同为$O(n)$，但这种方法前面的常数部分更大，耗费要更高。

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012307928.png" alt="image-20211101230757888" style="zoom:67%;" />

<center>电子竞技不相信眼泪</center>

### Approximation in RTR

有很多不等式，在一定情况下可以看作等号成立。由于实时渲染只要求看着真实，而不必满足绝对的真实，因此这种近似可以应用在渲染方程中。
$$
\int_{\Omega}f(x)g(x)dx \approx \frac{\int_{\Omega}f(x)dx}{\int_{\Omega}dx} \cdot \int_{\Omega}g(x)dx \tag{3.3}
$$
(数学含义或者证明待补充)

当满足以下其中一个条件时，两边可近似相等

1. Small support 

   指的是公式中的积分域 $\Omega$ 尽可能小的时候。因此如果只考虑直接光照，在点光源和方向光源的情况下， $\Omega$ 就只表示光源方向的立体角，就会很小。

2. Smooth integrand

   指的是被积函数 $g(x)$ 在 $\Omega$ 内的变化很小，即频率低，“光滑”的情况下。因此如果G(x)是一个漫反射的BRDF，那么它值的变化就不大。

当满足上面其中一个条件时，对于渲染方程来说，把 $V()$ 看作 $f(x)$ ,其他项看作 $g(x)$，渲染方程就可以写成：
$$
L_0(p,w_0) \approx \frac{\int_{\Omega^+}V(p,w_i) dw_i}{\int_{\Omega^+}dw_i} \cdot \int_{\Omega^+} L_i(p,w_i) f_r(p,w_i,w_o) cos\theta_i dw_i 
$$
这个积分会在环境光遮蔽（Ambient Occlusions）章节用到。

### PCF（Percentage Closer Filtering）

PCF是抗锯齿，反走样的一种技术。

基本思想：对一个像素的某个属性，取它周围多个像素属性的平均值。

在Shadow Mapping中的应用：可用于实现面光源的软阴影的效果

对于一个点 $p$，它到光源的距离是dis，通过与shadow map中深度值z的比较，我们可以得到它的可见性，从而确定它是否显示阴影。而PCF则是将dis与shadow map上的一块区域的深度值作比较，得到一系列的可见性值（0或1），再求这些可见性的均值。用此值去确定阴影颜色的深浅。

这块filter的区域越大，平均的像素也就越多，可以实现越模糊，越软的阴影。

### PCSS（Percentage Closer Soft Shadows）

![image-20211102211307583](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111022113681.png)

通过现实中的景象，我们可以观察到阴影在不同位置的软度是不同的。虽然上图有景深导致的模糊，但还是能说明这个问题。

此时有一个问题：对于不同的着色点 $p$ ，采用多大的filter？

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111022120550.png" alt="image-20211102212011505" style="zoom: 67%;" />

图中 $w_{Penumbra}$  表示软阴影的范围大小，即软的程度。影响它的因素有：面光源的大小 $w_{Light}$、面光源到遮挡物的距离 $d_{Blocker}$、面光源到阴影接收物的距离 $d_{Receiver}$。根据相似三角性原理：
$$
w_{Penumbra} = (d_{Receiver} - d_{Blocker}) \cdot w_{Light} / d_{Blocker} \tag{3.4}
$$
假设 $w_{Light}$ 已知，着色点 $p$ 到光源的距离 $d_{Receiver}$ 也可求。而 $d_{Blocker}$ 就是shadow map中记录的深度值（因为面光源无法生成shadow map，所以应该将面光源看作点光源，生成一张shadow map，比如用面光源中心的点）。

因为是面光源，所以对于我们要着色的一个点 $p$，遮挡物上不同的位置都可能挡住了一些入射光，所以需要求一个平均的 $d_{Blocker}$，就是对shadow map上的一个区域中的 $d_{Blocker}$ 求平均值（需要注意，如果shaodow map中的点没有遮挡点 $p$，那就不算遮挡物，不参与平均值计算），那么又有了一个问题，这个区域的大小是多少？

![image-20211102221703551](https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111022217609.png)

有一个方法是，让点 $p$ 连向光源的四个顶点（假设是四个），看这个视锥在shadow map上覆盖了多大的区域，用这个区域来当作求均值的区域。这种方法是很符合直观想象的，我们想求的就是遮挡物在shadow map上覆盖的区域。

至此，就可以完成整个流程：

1. 让点 $p$ 连向光源的顶点，看这个视锥在shadow map上覆盖了多大的区域，在这个区域中计算平均  $d_{Blocker}$。
2. 用公式(3.4) 计算软阴影范围大小，确定PCF的滤波范围。
3. 使用PCF，求出点 $p$ 的阴影深浅程度。
