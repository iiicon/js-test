### 图片上传

#### 第一种方式 form提交

随着原生HTML5表单对多图(multiple属性)、上传前预览，二进制上传等支持越来越广泛

对于PC项目，IE8-IE9浏览器还是不能忽略的。所以，现在，很流行的一种处理方式，就是HTML5 file上传和flash swfupload上传一起整合的模式，优先使用原生HTML5上传，不支持的，使用flash上传。

如果要想使用原生浏览器实现上传，父级的form元素有个东西不能丢，就是enctype='multipart/form-data',再次强调对于文件只能使用这个

只有低版本的IE浏览器貌似有方法，使用私有的滤镜，超越安全的限制（其实是利用了不好的东西），实现图片直接预览；但是呢，那个时候，Chrome, FireFox没有这一出，于是，想要使用原生file input实现图片的上传前预览，兼容性坎很难跨过去

但是，后来，HTML5来了，我们出现了转机，IE10+以及其他现代浏览器，可以让我们直接读取图片的数据，然后在页面上呈现，实现了上传前预览；加上之前老IE的滤镜策略，貌似，可行。但是呢但是，老的IE浏览器只能最多一次选择一个文件，因此，只有单图上传的时候，大家可以考虑考虑

传统的form提交，是要改变页面流的，也就是刷新后跳转。好的体验应该是走Ajax交互的。HTML5里面支持二进制formData数据提交，因此，可以从容Ajax提交上传的文件数据；那老旧的IE浏览器怎么办？
一般方法如下：

1. form元素新增target属性，其值指向页面内隐藏的一个<iframe>元素的id, 如下示意：
<form action="" method="post" enctype="multipart/form-data" target="uploadIframe"><
<iframe id="uploadIframe"></iframe>  
2. 处理<iframe>元素的onload事件，获得返回内容（如下代码示意），具体细节非本文重点，不表。
var doc = iframe.contentDocument ? iframe.contentDocument : frames[iframe.id].document;
var response = doc.body && doc.body.innerHTML;

input file类型控件有一个属性，名为accept, 可能有些小伙伴不太了解。可以用来指定浏览器接受的文件类型，也就是的那个我们打开系统的选择文件弹框的时候，默认界面中呈现的文件类型。例如：accept="image/jpeg"，则界面中只有jpg图片

