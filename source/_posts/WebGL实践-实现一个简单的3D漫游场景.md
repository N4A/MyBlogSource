---
title: WebGL实践_实现一个简单的3D漫游场景
sticky: 1
date: 2016-12-13 16:57:46
tags: [WebGL]
---

[演示效果](/demo/webgl/)

### 运行和使用方法

1. 推荐使用webstorm运行程序，会自动开启服务器来运行项目。需注意不能使用chrome直接打开本地文件来运行。如需如此运行，请关闭相关安全检查。Firefox 50.1可以直接打开html来运行。
2. 运行后，默认是基本画面，小鸟飞行动画，阴影以及雾的效果。用以下按键使用相应功能
   1. W,A,S,D : 分别控制前后左右移动，漫游场景。
   2. J,L,I,K : 分别控制向左，向右，向上，向下转动视角。
   3. F : 打开眼睛所在位置的点光源，照亮眼前事物，同时会照亮模型和贴图。
   4. 以上按键部分同时按为部分组合效果。

<!-- more -->

### 目录结构说明

1. lib/: 项目提供的库文件
2. image/: 项目使用到的图片资源文件。
3. model/: 项目使用到的模型资源文件
4. animation.js: 该文件维护了所有做动画所需要的信息，包括场景漫游时的相机参数，小鸟飞行的小鸟位置。
5. scene.js: 包含项目提供的配置文件，同时包含图片、模型资源文件的路径。添加了雾的配置参数
6. keyEvent.js: 绑定所有按键事件的处理过程。包括场景漫游响应和点光源响应
7. webGL.js: 封装了webGL的基本操作。例如初始化program，初始化各种buffer等操作。
8. shadow.js: 实现绘制shadow map的功能。
9. objLoader: 加载obj模型文件的库
10. objDraw: 实现预先加载obj文件到buffer和obj模型的绘制
11. index.js: 实现程序的主控制流，通过调用上述文件的方法来实现。
12. index.html: 项目启动页面。

### 开发和运行环境

1. 本项目使用webstorm开发，使用 Firefox 50.1 进行测试。
2. chrome升级到55之后，打开样例代码和自己的项目都是显示黑屏，chrome54可以正常显示。
3. IE11可以正常显示。

### 项目亮点

1. 纹理和光照结合。当按F打开点光源时，模型和纹理贴图都会被照亮。
2. 雾的效果，随着远离物体，会渐渐看不清。
3. 纹理题图的雾的效果，由于技术有限这个真的挺坑的，见下一部分的第3个问题
4. 阴影效果，模型和纹理会产生阴影效果.
5. 镜面反射。镜面反射是根据姜老师ppt的公式来做的。见下一部分的第4个问题
6. phong shading. phong shading就是在每个点都独立计算shading，这个参考第八章PointLightedCube_perFragment来实现。见下一部分第5个问题。

### 开发过程的问题和解决方法

1. 2D到3D成像的转换。一开始直接照着样例代码就开始写了，但是调了一些参数得不到想要的效果。解决办法是认真看教程和ppt，弄懂成像原理，尤其是modelMatrix，viewMatrix和projectMatrix的用途和用法，理解之后自然也就有了正确的参数设定概念。

2. 工具包的modelmtrix的变换如setScale()和scale()等的区别，以及viewMatrix的lookAt和setLookAt()的区别。看样例代码一会有set，一会没set，以为有没有set是一样的。使用的时候，就出现了一些不合逻辑的错误。解决办法是看源码，看到其中的不同之处。如下：

   ```javascript
   //scale函数是将原有matrix经该scale变换后，返回该matrix
   Matrix4.prototype.scale = function(x, y, z) {
     var e = this.elements;
     e[0] *= x;  e[4] *= y;  e[8]  *= z;
     e[1] *= x;  e[5] *= y;  e[9]  *= z;
     e[2] *= x;  e[6] *= y;  e[10] *= z;
     e[3] *= x;  e[7] *= y;  e[11] *= z;
     return this;
   };
   
   //setScale是直接将原有matrix设置为该scale变换matrix
   Matrix4.prototype.setScale = function(x, y, z) {
     var e = this.elements;
     e[0] = x;  e[4] = 0;  e[8]  = 0;  e[12] = 0;
     e[1] = 0;  e[5] = y;  e[9]  = 0;  e[13] = 0;
     e[2] = 0;  e[6] = 0;  e[10] = z;  e[14] = 0;
     e[3] = 0;  e[7] = 0;  e[11] = 0;  e[15] = 1;
     return this;
   };
   ```

   所以modelMatrix在变换时只能第一次变换时加set，这种实现是符合我们平常使用习惯的。

   搞清楚这点之后，问题也就迎刃而解了。

1. 雾的效果问题，做贴图的时候，坐标只有顶点的坐标，贴图内部雾的颜色实际上是根据顶点雾的颜色算出来的varying color。这一点对于箱子这样的小物体没有什么影响，而模型的顶点是更密集的也没有问题，但是对于地板这样的大物体就出现了错误，当站在地板中间的时候，距离四个顶点都很远，所以四个顶点的雾的颜色都很浓，这就导致中间的varying color也很浓。这样就使得只有四个角才会有雾的变换效果，而中间就一直是大雾弥漫，站在中间也看不清眼前地板。

   一种解决办法是地板的雾的效果总是以眼睛到地板中心的距离来计算，这样会有远离地板的雾的效果，但是整块地板同步变换，难以看到地板内部的雾的变换。

   有一种巧妙的办法可以较好的利用已有的点和技术解决这个问题。看下图，

   ![FOG](/img/fog.png)

   如图，大的九宫格代表地板，地板中心再坐标(0,0,0)的位置。做texture shader的时候只会传入A,B,(C,D)四个顶点的坐标，如前面所述的一样，用这四个顶点计算与眼睛的距离会导致站在中间，地板就全会被雾遮盖。但是我们可以对四个顶点在计算雾的效果的时候，做一个缩放变换，使得顶点对应到A1,B1,(C1,D1)的位置，这样的话，四个点将地板分为九个区域，这时就要调整雾的视距，以及A1-D1的位置，使得玩家脚下及周围是雾较淡，远处较浓。如图，玩家站在五角星的位置，A1点对应的A点的雾的颜色最淡，而玩家脚下距离地板四个顶点中A的点也是最近的，所以在varying color的时候，脚下的雾会比远处的淡。更远处的接近C，D（未标出）的区域就会看不清。

   这样还有一点反常，就是玩家在五角星位置向C，D（未标出）方向看去是正常的，向A方向看去就发现远处的A附近雾更加的淡，毕竟A才是真正的着色点。但是经过测试，整体效果要比前两种情况要好得多，有了雾的渐变效果。这个效果在上传的旧版本看到。

   当然最好的方法就是贴图中每一个点都独立单独计算距离最好了，但是之前不会。在做phong shading的时候，参考代码可以使用v_Position，这和v_Color一样，顶点之外的点坐标也会被算出来，如下：

   ```java
     VSHADER_SOURCE = 'attribute vec4 a_Position;\n' +
      'uniform mat4 u_ModelMatrix;\n' +
      'varying vec3 v_Position;\n' +
      'void main() {\n' +
      '  gl_Position = u_MvpMatrix * a_Position;\n' +
      '  v_Position = vec3(u_ModelMatrix * a_Position);\n' +
      '}\n';
     FSHADER_SOURCE =
           'precision mediump float;\n' +
           'varying vec3 v_Position;\n' +
           'void main() {\n' +
           	//这里在使用v_Position计算，就是这一个小fragment的坐标了
           '}\n'
   ```

   这样做出来的效果也就是真正的雾的渐变效果了。

2. 镜面反射问题。镜面反射没有找到样例代码。但是姜老师ppt上给了公式。就可以照着实现以下，如下：

   ```java
          // Calculate the point light direction and make it 1.0 in length,also view direction
              '  vec3 lightDirection = normalize(u_LightPosition - v_Position);\n' + // 点光源方向，同时也是眼睛所在方向。
              '  vec3 specular_r = -1.0 * u_DirectionLight + 2.0 * dot(normal, u_DirectionLight) * normal;'+
              '  float nDotS = max(dot(specular_r, u_DirectionLight), 0.0);'+
              '  vec3 specular = v_Color.rgb * nDotS;' + //平行光
   
              '  vec3 specular_r_p = -1.0 * lightDirection + 2.0 * dot(normal, lightDirection) * normal;'+
              '  float nDotS_P = max(dot(specular_r_p, u_DirectionLight), 0.0);'+
              '  vec3 specular_p = u_LightColor * nDotS_P * 0.05;' + //点光源，这里降低了强度，应为点光源和眼睛通向，看眼前又基本垂直强度太强。
   ```

   未加镜面反射之前，效果如下：

   ![diffuse](/img/diffuse.PNG)

   加了镜面反射之后，如下，可以看出镜面反光的感觉

   ![specular](/img/fog_new.PNG)

3. phong shading要求在每个点都独立计算shading。第八章给出样例用v_Potistion。参考可做如下实现：

   ```java
   VSHADER_LIGHT2_SOURCE:
           'attribute vec4 a_Position;\n' +
           'attribute vec4 a_Color;\n' +
           'attribute vec4 a_Normal;\n' +
           'uniform mat4 u_ModelMatrix;\n' +
           'uniform mat4 u_MvpMatrix;\n' +
           'uniform mat4 u_NormalMatrix;\n' +
           'varying vec4 v_Color;\n' +
           'varying vec3 v_Position;\n' +
           'varying vec3 v_Normal;\n' +
           'void main() {\n' +
           '  gl_Position = u_MvpMatrix * a_Position;\n' +
               // Calculate the vertex position in the world coordinate
           '  v_Position = vec3(u_ModelMatrix * a_Position);\n' +
           '  v_Normal = normalize(vec3(u_NormalMatrix * a_Normal));\n' +
           '  v_Color = a_Color;\n' +
           '}\n',
   // 之后在下部分的shading中就可以用v_Position和v_Normal独立计算每一个fragment的shading效果
             '  vec3 normal = normalize(v_Normal);\n' +
           '  float nDotL = max(dot(normal, u_DirectionLight), 0.0);\n' +
           '  vec3 diffuse = v_Color.rgb * nDotL;\n' +
   
               // Calculate the point light direction and make it 1.0 in length,also view direction
           '  vec3 lightDirection = normalize(u_LightPosition - v_Position);\n' +
           '  vec3 specular_r = -1.0 * u_DirectionLight + 2.0 * dot(normal, u_DirectionLight) * normal;'+
           '  float nDotS = max(dot(specular_r, u_DirectionLight), 0.0);'+
           '  vec3 specular = v_Color.rgb * nDotS;' +
   
           '  vec3 specular_r_p = -1.0 * lightDirection + 2.0 * dot(normal, lightDirection) * normal;'+
           '  float nDotS_P = max(dot(specular_r_p, u_DirectionLight), 0.0);'+
           '  vec3 specular_p = u_LightColor * nDotS_P * 0.05;' +
   
               // The dot product of the light direction and the normal
           '  float nDotLPoint = max(dot(lightDirection, normal), 0.0);\n' +
               // Calculate the color due to diffuse reflection
           '  vec3 diffusePoint = u_LightColor * u_LightColor * nDotLPoint;\n' +
               // Calculate the color due to ambient reflection
           '  vec3 ambient = u_AmbientLight * v_Color.rgb;\n' +
           '  vec4 f_Color = vec4(specular + specular_p + diffuse + diffusePoint + ambient, v_Color.a);\n' +
   ```

   但是感觉效果不是很明显，上传的老版本中是没有加的效果。

4. 阴影呈现问题。对于阴影的呈现，光源的位置要调好。经过多番尝试，是放在了相机初始位置右边一点点，方向与相机初始方向相同。阴影是根据shadow map来实现的，所以背面的东西都是阴影。

   还有一点就是场景里小鸟是一直在移动的，所以最好小鸟的影子也是跟着移动。解决办法也很简单，就是每次在绘制场景前都重新绘制以下shadow map。这样性能会有点损耗。

5. 效果顺序问题。光照，阴影，雾都是通过颜色变换来实现的。不同的顺序处理效果也不一样，这里雾的效果计算要放在最后。

6. 性能优化问题，首先模型贴图都要预先加载好，不要每次都加载。其次小鸟飞行的动画已经按帧重绘，其它动画就不需要每次都调用重绘函数。

### 可能存在的问题及思考

1. 性能问题，项目的性能不好，频繁操作还会卡住，有待优化。
2. 阴影的问题，很诧异的是不知道为什么箱子可以接受阴影,但是不会产生阴影.