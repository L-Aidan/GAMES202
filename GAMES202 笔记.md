# GAMES202 笔记

## CG基础回顾

### 基础GPU渲染管线

（待补充）

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202110282255523.png" alt="image-20211028225552443" style="zoom: 67%;" />

### OpenGL

### OpenGL 着色语言(GLSL)

### 渲染方程

$$
L_0(p,w_0) = L_e(p,w_0) + \int_{H^2} f_r(p,w_i\rightarrow w_o) L_i(p,w_i) cos\theta_i dw_i
$$

实时渲染中的渲染方程通常写为：
$$
L_0(p,w_0) = L_e(p,w_0) + \int_{\Omega^+} L_i(p,w_i) f_r(p,w_i,w_o) cos\theta_i V(p,w_i) dw_i
$$
这里公式中的 $L_i( p ,w_i)$ 与 $V( p ,w_i)$ 相乘才等同于上一公式的 $L_i( p ,w_i)$，表示将①光源是否存在 和②光源能否打到 $ p$ 点 两项分开考虑

## Shadow Mapping

### 基本思想

这里只考虑点光源或者平行光源。

两趟算法：以点光源为例，第一趟，将摄像机放在点光源的位置，渲染出一张shadow map，其实是depth map，用来记录光源看到的最近的点的深度。第二趟，用实际的观察点渲染场景，对摄像机看到的每一个点 $p$ ，通过矩阵变换，将其投影到上一张shadow map中，查询投影坐标上的**depth值**， 和 **$p$ 点到光源的距离**做比较，如果depth值<距离，表明有物体挡在了光源和 $p$ 点之间，则 $p$ 点应该显示阴影。

需要注意：shadow map中保存的是场景中顶点的真实深度（点到光源的真实距离）还是在裁剪空间（视锥体frustum需要经过仿射变换变为正方体形状的裁剪空间）内的z值？进行深度比较时，要保证比较双方的意义是相同的。

平行光源和点光源的区别在于观察空间不同，平行光源是一个长方体，点光源是平截头体。

### 存在的问题

#### 自遮挡（Self occlusion）

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012230917.png" alt="image-20211101223001812" style="zoom: 50%;" />

由于shadow map本身是有分辨率的，所以它每个像素中用一个值表示一片区域的深度值。对地面上的一个点 $p$ ，如果它对应的shadow map中的深度值小于 $p$ 点到光源的距离，即 z < dis 时，就会被判定为遮挡。

解决方法：我们可以设定一个bias来抵消上面提到的两个距离的差值，当z + bias < dis 时才判定为遮挡。考虑到在 $p$ 点，如果光源是近乎垂直打在地面上的，那儿shadow map中的值和 $p$ 的深度值差异会很小，bias就可以很小。如果光束方向近乎平行于地面（grazing angle），那么该差异可能很大，bias也需要较大。因此可以用入射光的角度$\theta$来控制bias的大小。

但设置bias会带来另一个问题：detached shadow。

#### Detached shadow

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012248429.png" alt="image-20211101224834349" style="zoom: 50%;" />

设置bias后，有可能出现这种情况，即本来判定为遮挡的点，因为bias的存在，被判定为不遮挡。那么当bias较大时，就会出现上图的状况。

### Second Depth Shadow Mapping

(试图解决自遮挡和detached shadow的方法，但因耗费太高而没有人用)

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012256008.png" alt="image-20211101225651952" style="zoom:50%;" />

这个方法和最基础的Shadow Mapping方法的区别就是，它采用最小深度和次小深度的均值来作为shadow map中的值。因为不采用bias，所以不会出现detached shadow；如果最小深度和次小深度的值相差较大，也会一定程度减少自遮挡（个人理解）。但缺点就是①：需要模型是一个“盒子”类的东西（watertight）②：保存每个像素的最小深度和次小深度，尽管时间复杂度相同为$O(n)$，但这种方法前面的常数部分更大，耗费要更高。

<img src="https://raw.githubusercontent.com/L-Aidan/Images/main/img/202111012307928.png" alt="image-20211101230757888" style="zoom:67%;" />

<center>电子竞技不相信眼泪</center>

走样

两个积分不等式

PCF

PCSS
