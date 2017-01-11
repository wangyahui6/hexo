---
title: js处理二进制数据
date: 2017-01-10 11:34:43
categories: web前端
tags: javascript
---
## 二进制数据
首先要知道什么是二进制数据。有人会很纳闷，计算机所有的数据都是以二进制方式存储的，那这个二进制数据到底是什么？其实，各种数据格式的不同在于它的存储和解读规则不同，存储和解读规则要一一对应才能正确解析。比如整型和浮点型的5，都是以二进制0和1存储的，但是存储规则不一样，存储的0、1序列当然也不一样，但是浮点型的5也可以用整型规则解析，只不过解析出来的不是5，是其他另外一个值。说了这么多，就是想说计算机中所有的数据都是二进制码，不同类型的数据编码规则不同，正确使用数据的前提是你有正确解析数据的规则。Javascript采用Unicode字符集，编码方式是UCS-2。二进制数据不是字符数据，不能按照UCS-2方式解码，如果想解读二进制数据，可以利用javascript提供的charCodeAt函数得到二进制数据的真正0、1序列。ES5提供了Blob对象(binary large object)来处理二进制数据。
<!-- more -->
## Blob对象
Blob对象提供了一系列的API来处理二进制数据。其他的文件处理对象都是继承了它的API，有如下几种：
- FileList对象，File对象的集合，一般用于表单提交文件或者拖拽文件时
- File对象，用于处理文件形式的二进制数据，是FileList对象的属性值
- FileReader对象，读取二进制数据转化为不同的格式
- URL对象，用于对二进制数据生成URL。

Blob对象代表一段二进制数据，然后有各种接口处理二进制数据。有两种方法生成Blob数据，一是使用Blob构造函数生成，而是使用Blob对象的slice方法对现有Blob对象进行切割。Blob构造函数有两个参数，一是用于保存二进制数据的数组，二是数据的类型。
```
var blob = new Blob(['hello world!'],{type:'text/plain'});
```
Blob对象的slice方法，按照字节分隔原来的Blob对象生成新的Blob对象
```
var newBlob = oldBlob.slice(startByte,endByte);
```
我们可以利用这个方法分段上传大文件，加快处理速度。
```
document.querySelector('input[type="file"]').addEventListener('change',function(e) {
    var startByte = 0,
        endByte = 0,
        mountByte = 100,
        blob = this.file[0];
    while(startByte < blob.size){
        var xhr = new xmlHttpRequest();
        xhr.open('POST',url,true);
        endByte = startByte + mountByte;
        var newBlob = blob.slice(startByte,endByte);
        xhr.send(newBlob);
        startByte = endByte;
    }
});

```
Blob对象有两个只读属性
- size：二进制数据的大小，单位为字节
- type：二进制数据的MIME类型，全部为小写，如果类型未知，则该值为字符串

## Filelist对象和File对象
Filelist对象是针对表单file控件提供的处理文件类型的二进制数据的接口，用户通过file控件选取文件时，该空间的files值就是FileList对象，FileList是一个数组，包含用户选择的多个文件。数组的每一项都是一个File对象。File对象继承了Blob对象，File对象有自己的一些只读属性：
- name:文件名，该属性只读
- size:文件大小，单位为字节，该属性只读
- type:二进制数据的MIME类型，全部为小写，如果类型未知，则该值为字符串，该属性只读
- lastModified:文件上次修改的时间,格式为时间戳
- lastModifiedDate:文件上次修改时间，格式为Date对象实例

## FileReader对象 
FileReader对象用于读取二进制数据，将其转换为不同个格式，有以下几个方法：
1. readAsBinaryString(Blob|File) 返回一个二进制字符串，该字符串每个字节包含一个0-255的整数
2. readAsText(Blob|File,opt_encoding) 返回文本字符串。默认情况下，文本编码格式是’UTF-8’，可以通过可选的格式参数，指定其他编码格式的文本
3. readAsDataURL(Blob|File) 返回一个基于Base64编码的data-uri对象,可用于下载或者图片显示。
4. readAsArrayBuffer(Blob|File)：返回一个ArrayBuffer对象。
采用readAsDataURL可以将二进制数据转化为base64编码，可以作为img标签的src属性或者a标签的href属性。
```
var reader = new FileReader();
reader.onload = function(e) {
    console.log(e.target.result);
}
reader.readAsDataURL(blob);
```
## URL对象 
URL对象用于生成指向File对象或Blob对象的URL。由window.URL.createObjectURL生成。
var url = window.URL.createObjectURL(blob);
```
var url = window.createObject()
```
上面的代码会对二进制数据生成一个URL，类似于“blob:http%3A//test.com/666e6730-f45c-47c1-8012-ccc706f17191”。这个URL可以放置于任何通常可以放置URL的地方，比如img标签的src属性。需要注意的是，即使是同样的二进制数据，每调用一次URL.createObjectURL方法，就会得到一个不一样的URL。
这个URL的存在时间，等同于网页的存在时间，一旦网页刷新或卸载，这个URL就失效。除此之外，也可以手动调用URL.revokeObjectURL方法，使URL失效。

综上所述，es5提供了blob对象用于处理二进制数据。file对象继承自blob对象。其中FileReader对象的readAsDataURL和window.URL.createObjectURL方法可以将二进制数据转化为src和href可用的数据，用于显示和下载。