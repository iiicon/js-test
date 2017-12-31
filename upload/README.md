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

input file类型控件有一个属性，名为accept, 可能有些小伙伴不太了解。可以用来指定浏览器接受的文件类型，也就是的那个我们打开系统的选择文件弹框的时候，默认界面中呈现的文件类型。例如：accept="image/jpeg"，则界面中只有jpg图片, 兼容谷歌所以如下
<input type="file" accept="image/gif,image/jpeg,image/jpg,image/png">

### 接下来该熟悉各种api

#### Filelist 对象和 file 对象

HTML 中的 input[type="file"] 标签有个 multiple 属性，允许用户选择多个文件，FileList对象则就是表示用户选择的文件列表。这个列表中的每一个文件，就是一个 file 对象。

file 对象的属性
1. name 文件名，不包含路径
2. type 文件类型。图片类型的文件都会以 image/ 开头，可以由此来限制只允许上传图片。
3. size 文件大小 可根据文件大小进行其他操作
4. lastModified 文件最后修改的时间
```
  <input type="file" id="files" multiple>
  <script>
      var elem = document.getElementById('files');
      elem.onchange = function (event) {
          var files = event.target.files;
          for (var i = 0; i < files.length; i++) {
              // 文件类型为 image 并且文件大小小于 200kb
              if(files[i].type.indexOf('image/') !== -1 && files[i].size < 204800){
                  console.log(files[i].name);
              }
          }
      }
  </script>
``` 

#### Blob 对象
Blob 对象相当于一个容器，可以用于存放二进制数据。它有两个属性，size 属性表示字节长度，type 属性表示 MIME 类型。
Blob 对象可以用 Blob() 创建
```
var blob = new Blob(['hello'], {type:"text/plain"});
```

Blob 构造函数中的第一个参数是一个数组，可以存放 ArrayBuffer对象、ArrayBufferView 对象、Blob对象和字符串。
```
var newblob = blob.slice(0,5, {type:"text/plain"});
```

canvas.toBlob() 也可以创建 Blob 对象。toBlob() 使用三个参数，第一个为回调函数，第二个为图片类型，默认为 image/png，第三个为图片质量，值在0到1之间。
```
var canvas = document.getElementById('canvas');
canvas.toBlob(function(blob){ console.log(blob); }, "image/jpeg", 0.5);
```

### 下载文件
Blod 对象可以通过 window.URL 对象生成一个网络地址，结合 a 标签的 download 属性来实现下载文件功能。
比如把 canvas 下载为一个图片文件。
```
var canvas = document.getElementById('canvas');
canvas.toBlob(function(blob){
    // 使用 createObjectURL 生成地址，格式为 blob:null/fd95b806-db11-4f98-b2ce-5eb16b38ba36
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.download = 'canvas';
    a.href = url;
    // 模拟a标签点击进行下载
    a.click();
    // 下载后告诉浏览器不再需要保持这个文件的引用了
    URL.revokeObjectURL(url);
});

```

### FileReader 对象
FileReader 对象主要用来把文件读入内存，并且读取文件中的数据。通过构造函数创建一个 FileReader 对象
`var reader = new FileReader();`

该对象有一下方法
abort：中断读取操作。
readAsArrayBuffer：读取文件内容到ArrayBuffer对象中。
readAsBinaryString：将文件读取为二进制数据。
readAsDataURL：将文件读取为data: URL格式的字符串。
readAsText：将文件读取为文本。

### 上传文件预览
```
<input type="file" id="files" accept="image/jpeg,image/jpg,image/png">
<img src="blank.gif" id="preview">
<script>
    var elem = document.getElementById('files'),
        img = document.getElementById('preview');
    elem.onchange = function () {
        var files = elem.files,
            reader = new FileReader();
        if(files && files[0]){
            reader.onload = function (ev) {
                img.src = ev.target.result;
            }
            reader.readAsDataURL(files[0]);
        }
    }
</script>
```

### 数据备份与恢复
FileReader 对象的 readAsText() 可以读取文件的文本，结合 Blob 对象下载文件的功能，那就可以实现将数据导出文件备份到本地，当数据要恢复时，通过 input 把备份文件上传，使用 readAsText() 读取文本，恢复数据。

### Base64 编码
在 HTML5 中新增了 atob 和 btoa 方法来支持 Base64 编码。它们的命名也很简单，b to a 和 a to b，即代表着编码和解码。
```
var a = "https://lin-xin.github.io";
var b = btoa(a);
var c = atob(b);

console.log(a);     // https://lin-xin.github.io
console.log(b);     // aHR0cHM6Ly9saW4teGluLmdpdGh1Yi5pbw==
console.log(c);     // https://lin-xin.github.io
```

btoa 方法对字符串 a 进行编码，不会改变 a 的值，返回一个编码后的值。atob 方法对编码后的字符串进行解码。
但是参数中带中文，已经超出了8位ASCII编码的字符范围，浏览器就会报错。所以需要先对中文进行 encodeURIComponent 编码处理。
```
var a = "哈喽 世界";
var b = btoa(encodeURIComponent(a));
var c = decodeURIComponent(atob(b));

console.log(b);     // JUU1JTkzJTg4JUU1JTk2JUJEJTIwJUU0JUI4JTk2JUU3JTk1JThD
console.log(c);     // 哈喽 世界
```

### ie 如果要在上传前实现预览就用使用自己独有的滤镜，（以后再做总结，今天要跨年，暂不做总结）😁