---
title: Cropper Component Essay
date: 2020-03-06 16:01:00
tags: [component,Essay]
published: true
hideInList: false
feature: /post-images/cropper-component-essay.jpg
---
1. 如果我们想要实现一个裁剪组件，最核心的方法就是裁剪函数

	**裁剪函数需要用到 Html Canvas API** 

	-  创建画布

		```js
		const canvas = document.createElement("canvas");
		const ctx = canvas.getContext("2d");
		```

	-  绘制图像

		```js
		ctx.drawImage(img, dx, dy, width, height);
		```
				
	-  输出裁剪图像
	
		```js
		// 输出为 blob 格式
		canvas.toBlob(blob=>{}, type, encoderOptions)
		// 输出为 dataUrl 格式
		canvas.toDataURL(type, encoderOptions)
		```
				
	- 调整画布

		```js
		// 旋转
		ctx.rotate()
		// 拉伸
		ctx.translate()
		//缩放
		ctx.scale()
		// ! save 只是保存的 CanvasRenderingContext2D 对象的状态以及对象的所有属性
		ctx.save() / ctx.restore()
		```
		
2. 当然裁剪组件不可能这么简单，我们还得需要一个容器，让使用者在里面进行图片的裁剪

	-  在容器内放置用户上传的图片，给图片设置 scale tanslate rotate 属性
	
	```html
		<div
			class="cropper-box"
			:style="{
			transform: 'scale(' + scale + ',' + scale + ') ' + 'translate3d('+ x / scale + 'px,' + y / scale + 'px,' + '0)' +  'rotateZ('+ rotate * 90 +'deg)'
		}"
		>
			<img :src="imgs" />
		</div>
	```
		
	-  设置图片可拖拽区域，一般与容器大小相同，其中产生的x ， y 就是图片在容器左上角移动的距离

	```html
	    <div
      class="cropper-drag-box"
      :class="{'cropper-move': canMove,'cropper-modal': cropping}"
      @mousedown="startMove"
      @touchstart="startMove"
    ></div>
	```
	
3. 我们还需要一个裁剪框容器，裁剪框内出现的就是裁剪出的内容

	-  在裁剪框容器内同样放置用户上传的图片，给图片设置 scale tanslate rotate 属性
			
	```html
	<div class="cropper-view-box">
        <img
          :src="imgs"
          :style="{
						'transform': 'scale(' + scale + ',' + scale + ') ' + 'translate3d('+ (x - cropOffsertX) / scale  + 'px,' + (y - cropOffsertY) / scale + 'px,' + '0)'
						+ 'rotateZ('+ rotate * 90 +'deg)'
						}"
        />
	</div>
	```
	
	**这里的 x - cropOffsertX 与 y - cropOffsertY 就是裁剪框内图片相对于裁剪框左上角的位置**

	-  设置图片可拖拽区域，一般与容器大小相同，其中产生的changeX, changeY就是裁剪框容器距外层容器左上角移动的距离，注意裁剪框不要超出外层容器即可。

	```html
	<span class="cropper-face cropper-move" @mousedown="cropMove" @touchstart="cropMove"></span>
	```

	- 这里之所以还要放置一个与外面相同的图片，是为了让图片进入裁剪框时高亮显示，我们需要明白两张图片其实在视觉上在同一位置

		-  首先因为两张图片都是基于各自容器设置的绝对位置

		```css
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		```
	
		-  所以当 translate 时，外层裁剪容器根据自身移动时，移动为 x 与 y 那么内层裁剪框的根据自身移动 translate 就是 x- x - cropOffsertX 与 y - cropOffsertY，cropOffsertX 与 cropOffsertY 为裁剪框距离外层裁剪容器的距离

	-  还有一个逻辑较为复杂的地方是，裁剪框会被拉伸，随意改变大小

		- 首先判断改变的是 X 还是 Y

		- 在里层继续判断产生的 changeX 和 changeY 是正数还是负数，结合下层来判断裁剪框是增大还是减小

		- 继续在第三内层判断 change 与 crop 的大小， 因为存在两极反转的情况

		**比较绕，但我们可以这么想，首先第一层我们需要知道此时拖拽改变的是 X 还是 Y 轴，其次在 X 轴上裁剪框（正方形）存在两条边，那么我们就要第二层来知道产生的 change (拖拽距离) 为正还是为负，来分别处理两条边的情况，比如左边那条边 change 为负裁剪框增加，但是右边那条边却是减小。**
		
		**第三层 if 逻辑存在的理由是存在左边因为拖拽变成右边的情况（两极反转），在第三层里面，我们就去做 crop 大小和与外层容器距离的计算了（位置 + 大小 就能确定 crop 的情况了）**
		
		- 代码如下

		```js
		const changeW = nowX - this.cropX;
        const changeH = nowY - this.cropY;

        if (this.canChangeX) {
          if (this.changeCropTypeX === 1) {
            // 判断「两极反转」的情况
            if (this.cropOldW - changeW > 0) {
              this.cropW =
                this.containerWidth - this.cropPositionX - changeW <=
                this.containerWidth
                  ? this.cropOldW - changeW
                  : this.cropOldW + this.cropPositionX;
              this.cropOffsertX =
                this.containerWidth - this.cropPositionX - changeW <=
                this.containerWidth
                  ? this.cropPositionX + changeW
                  : 0;
            } else {
              // 左坐标线与右坐标线两极反转
              this.cropW =
                Math.abs(changeW) + this.cropPositionX <= this.containerWidth
                  ? Math.abs(changeW) - this.cropOldW
                  : this.containerWidth - this.cropW - this.cropPositionX;
              this.cropOffsertX = this.cropPositionX + this.cropOldW;
            }
          } else if (this.changeCropTypeX === 2) {
            if (this.cropOldW + changeW > 0) {
              this.cropW =
                this.cropOldW + changeW + this.cropPositionX <=
                this.containerWidth
                  ? this.cropOldW + changeW
                  : this.containerWidth - this.cropPositionX;
              this.cropOffsertX = this.cropPositionX;
            } else {
              // 左坐标线与右坐标线两极反转
              this.cropW =
                Math.abs(changeW + this.cropOldW) <= this.cropPositionX
                  ? Math.abs(changeW + this.cropOldW)
                  : this.cropPositionX;
              this.cropOffsertX =
                Math.abs(changeW + this.cropOldW) <= this.cropPositionX
                  ? this.cropPositionX - Math.abs(changeW + this.cropOldW)
                  : 0;
            }
          }
        }

        if (this.canChangeY) {
          if (this.changeCropTypeY === 1) {
            // 判断「两极反转」
            if (this.cropOldH - changeH > 0) {
              this.cropH =
                this.cropPositionY + changeH >= 0
                  ? this.cropOldH - changeH
                  : this.cropPositionY + this.cropOldH;

              this.cropOffsertY =
                this.cropPositionY + changeH >= 0
                  ? this.cropPositionY + changeH
                  : 0;
            } else {
              this.cropH =
                changeH - this.cropOldH <=
                this.containerHeight - this.cropPositionY - this.cropOldH
                  ? changeH - this.cropOldH
                  : this.containerHeight - this.cropPositionY - this.cropOldH;

              this.cropOffsertY = this.cropPositionY + this.cropOldH;
            }
          } else if (this.changeCropTypeY === 2) {
            if (changeH + this.cropOldH > 0) {
              this.cropH =
                changeH <=
                this.containerHeight - this.cropOldH - this.cropPositionY
                  ? this.cropOldH + changeH
                  : this.containerHeight - this.cropPositionY;

              this.cropOffsertY = this.cropPositionY;
            } else {
              this.cropH =
                Math.abs(changeH + this.cropOldH) <= this.cropPositionY
                  ? Math.abs(changeH + this.cropOldH)
                  : this.cropPositionY;

              this.cropOffsertY =
                Math.abs(changeH + this.cropOldH) <= this.cropPositionY
                  ? this.cropPositionY - Math.abs(changeH + this.cropOldH)
                  : 0;
            }
          }
        }
		```
		
**随笔真的很重要哇～**
	
		
		

		
				
		
