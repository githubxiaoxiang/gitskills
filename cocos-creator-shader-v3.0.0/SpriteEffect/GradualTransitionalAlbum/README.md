## CocosCreatorShader
#### Write by yeshao2069.
#### CocosCreatorShader intends to help users who do not know Shader quickly understand how to use Cocos Effect.
#### Cocos Creator V3.0.0+
&nbsp;

## 渐变过渡的相册效果  GradualTransitionalAlbum
#### 感谢 异名 
#### 原仓库地址(2.4.x版本): https://github.com/ifengzp/cocos-awesome/tree/master/assets/Scene/Photo_gallery
### 演示
#### 相册是一个大家比较熟悉的场景，一般我们是实现的都是那种跑马灯式的轮播相册，这里给大家提供一个利用shader实现图片渐变过渡的相册思路。
![image](https://gitee.com/yeshaohelpme/ShaderDemoImageLibrary/raw/master/image/GradualTransitionalAlbum.gif)
### 实现思路 (以2.4.x的shader代码为例，3.x的shader代码[请移步](https://gitee.com/yeshao2069/cocos-creator-shader/blob/v3.0.0/SpriteEffect/GradualTransitionalAlbum/assets/res/shader/pic_transform.effect))
#### 拆分一下功能点，主要有两个：一个是实现图片的渐变，一个是实现图片的切换。
#### 图片的渐变可以理解为随着时间的变化，在某一方向上的局部的像素点的透明度变化。demo中实现的效果是一个水平滚轴式的切换，水平平移在数学上的实现其实就是一个简单的关于时间变化的垂直直线x = time，我们只需要把每个像素点的x坐标和这个垂直直线做比较，在左边的透明度设为0，在右边的透明度设为1，然后再通过平滑取样就能够有渐变过渡的效果。
```ts
void main () {
  vec4 color = vec4(1, 1, 1, 1);
  color *= texture(texture, v_uv0);
  color *= v_color;

  #if USE_TRAMSFORM
    color.a = smoothstep(0.0, u_fade_radius, u_fade_radius + v_uv0.x - u_time);
  #endif

  gl_FragColor = color;
}
```
#### 实现了图片的渐变，接下来就是图片的切换，所有的图片其实都在一个队列当中，我们在渐变完成之后只需要把最上面的的那张图片放到最下面，就能够让这个相册一直在循环播放，在这个过程中我们再加上一些图片的状态处理就能够是实现demo中的渐变相册效果了。
```ts
isTransforming: boolean = false;
bgTramsform() {
  if (this.isTransforming) return;
  this.isTransforming = true;

  let time = 0.0;
  let node = this.switchNodeList[0];
  let material = node.getComponent(cc.Sprite).getMaterial(0);
  material.setProperty('u_fade_radius', this.fadeRadius);
  material.setProperty('u_time', time);
  material.define('USE_TRAMSFORM', true, 0, true);

  let timer = setInterval(() => {
    time += 0.03;
    material.setProperty('u_time', time);
    if (time > 1.0 + this.fadeRadius) {
      this.switchNodeList.shift();
      this.switchNodeList.push(node);
      this.switchNodeList.forEach((node, idx) => node.zIndex = this.switchNodeList.length - idx)
      material.define('USE_TRAMSFORM', false, 0, true);
      this.isTransforming = false;
      timer && clearInterval(timer);
    }
  }, 30);
}
```