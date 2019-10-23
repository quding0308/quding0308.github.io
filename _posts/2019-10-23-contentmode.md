---
layout: post
title:  "UIImageView的contentMode属性"
categories: blog
---

* 目录
{:toc}

### 测试

contentMode 属性

默认值是 scaleToFill

> A flag used to determine how a view lays out its content when its bounds change.

> The content mode specifies how the cached bitmap of the view’s layer is adjusted when the view’s bounds change. This property is often used to implement resizable controls. Instead of redrawing the contents of the view every time, you can use this property to specify that you want to scale the contents (either with or without distortion) or pin them to a particular spot on the view.


下面的图是一张 600 * 300 的 @2x 的图片，使用下面的图片来做测试。

![](/assets/img/contentmode/default@2x.png)


### top topLeft topRight

top 为 topCenter

使用 top topLeft topRight 不会对图片做拉升或压缩，而是会根据 imageView 的 width height 对图片做剪裁或留空白

例如上面的图片 600 * 300 ，在代码中对应宽高是 300 * 150

以 topLeft 为例( topLeft 会基于图片的左上角为参照对图片做裁剪)

|  imageView的宽高   | 实际显示  | 裁剪后的图片 |
|  ----  | ----  |
| (0,0,300,150)  | 图片正好全部显示 | ![](/assets/img/contentmode/default@2x.png) |
| (0,0,300,75)  | 图片会裁剪下半部分(一半) | ![](/assets/img/contentmode/300-75.png) |
| (0,0,150,150)  | 图片会裁剪右半部分(一半) | ![](/assets/img/contentmode/150-150.png) |
| (0,0,150,75)  | 图片裁剪左上部分(四分之一) | ![](/assets/img/contentmode/150-75.png) |
| (0,0,400,150)  | 图片全部显示，imageView右边有100px显示空白 | ![](/assets/img/contentmode/400-150.png) |
| (0,0,300,200)  | 图片全部显示，imageView下面有50px显示空白 | ![](/assets/img/contentmode/300-200.png)|


### bottom bottomLeft bottomRight

bottom 为 bottomCenter

使用 bottom bottomLeft bottomRight 不会对图片做拉升或压缩，而是会根据 imageView 的 width height 对图片做剪裁或留空白

bottomLeft 会基于图片的左下角为参照点对图片做裁剪

### center

center 以图片的中心为参照点对图片做裁剪，不会拉升或压缩图片

|  imageView的宽高   | 实际显示  | 裁剪后的图片 |
|  ----  | ----  |
| (0,0,300,150)  | 图片正好全部显示 | ![](/assets/img/contentmode/default@2x.png) |
| (0,0,150,75)  | 图片裁剪左上部分(四分之一) | ![](/assets/img/contentmode/center-150-75.png) |
| (0,0,400,150)  | 图片全部显示，imageView上下各留50px的空白 | ![](/assets/img/contentmode/center-400-150.png) |
| (0,0,300,200)  | 图片全部显示，imageView左右各留25px的空白 | ![](/assets/img/contentmode/center-300-200.png)|


### scaleToFill

图片不做裁剪，通过拉伸或压缩图片来适应 imageview 的宽高

|  imageView的宽高   | 实际显示  | 裁剪后的图片 |
|  ----  | ----  |
| (0,0,300,150)  | 图片正好全部显示 | ![](/assets/img/contentmode/default@2x.png) |
| (0,0,150,150)  | 图片水平被压缩一半 | ![](/assets/img/contentmode/scaleToFill-150-150.png) |
| (0,0,300,75)  | 图片垂直被压缩一半 | ![](/assets/img/contentmode/scaleToFill-300-75.png) |
| (0,0,400,150)  | 图片水平被拉伸 | ![](/assets/img/contentmode/scaleToFill-400-150.png) |


### scaleAspectFill

scale aspect 按比例伸缩

scaleAspectFill 按比例伸缩，图片完全填充满 imageView ，部分图片被裁剪掉

|  imageView的宽高   | 实际显示  | 裁剪后的图片 |
|  ----  | ----  |
| (0,0,300,150)  | 图片正好全部显示 | ![](/assets/img/contentmode/default@2x.png) |
| (0,0,300,75)  | 图片上下部分被裁减了37.5px | ![](/assets/img/contentmode/scaleAspectFill-300-75.png) |

### scaleAspectFit

scaleAspectFill 按比例伸缩，图片填充满 imageView，图片不会被裁掉，不够的部分填充空白

|  imageView的宽高   | 实际显示  | 裁剪后的图片 |
|  ----  | ----  |
| (0,0,300,150)  | 图片正好全部显示 | ![](/assets/img/contentmode/default@2x.png) |
| (0,0,300,75)  | 图片左右部分留空白75px | ![](/assets/img/contentmode/scaleAspectFit-300-75.png) |