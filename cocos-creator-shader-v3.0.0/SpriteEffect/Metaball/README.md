## CocosCreatorShader
#### Write by yeshao2069.
#### CocosCreatorShader intends to help users who do not know Shader quickly understand how to use Cocos Effect.
#### Cocos Creator V3.0.0+
&nbsp;

## 融球效果  Metaball
#### 感谢 异名 
#### 原仓库地址(2.4.x版本): https://github.com/ifengzp/cocos-awesome/tree/master/assets/Scene/Metaball
### 演示
#### 元球也叫融球，它能够让两个球体产生“黏糊糊”的效果，是流体，融合等效果的实现基础，这次实现的demo是一个固定的大圆，然后手指控制一个游离态的小圆
![image](https://gitee.com/yeshaohelpme/ShaderDemoImageLibrary/raw/master/image/Metaball.gif)
### 实现思路 (以2.4.x的shader代码为例，3.x的shader代码[请移步](https://gitee.com/yeshao2069/cocos-creator-shader/blob/v3.0.0/Metaball/assets/res/shader/Metaball.effect))
#### Metaballs在数学上是一个求等势面的公式，两个球体之间的等势面为`E = R² / (△x² + △y²)`，
```ts
float energy(float r, vec2 point1, vec2 point2) {
  return (r * r) / ((point1.x - point2.x) * (point1.x - point2.x) + (point1.y - point2.y) * (point1.y - point2.y));
}
```
#### demo的实现很简单，固定的圆处于中心的位置，加大一下半径，求出它的等势面`energy(u_radius + 0.1, v_uv0.xy, vec2(0.5))`，然后我们在手指的落足点再生成一个等势面`energy(u_radius, v_uv0.xy, u_point)`，然后叠加它们，让处于等势面上的点的色值透明度为1，不在该等势面上的透明度为0就可以达到视觉中的球体融合效果。
```ts
void main(){
  vec4 color = texture(texture, v_uv0);

  float fragEnergy = energy(u_radius + 0.1, v_uv0.xy, vec2(0.5)) + energy(u_radius, v_uv0.xy, u_point);
  color.a = smoothstep(0.95, 1.0, fragEnergy);
  gl_FragColor = color;
}
```