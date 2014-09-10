
## 测量和影响性能的工具与技术

#### YSlow

#### WebPageTest

#### 压缩

## 性能的标准化

>window.performance对象是W3C制定的用于收集浏览器性能度量值的一种新的标准化方法。Performance Timing对象收集并获取性能指标，Performance Navigation对象判断用户以何种方式访问网站，Performance对象公开的高分辨率时间，用来帮助用户跟踪精度精度高于毫秒的时间数据。

## Web性能优化

### 优化页面的渲染瓶颈

#### javascript置于HTML的末尾

>原因：在标记化过程中，如果渲染引擎识别出一个外部脚本，它就会暂停对脚本内容的解析，转而下载脚本。只有当下载完毕并得到执行之后，渲染引擎才会回过头来继续解析。

#### 脚本加载

>如果碰到脚本标签没有src属性，那么渲染引擎就只会将代码传递给javascript解析器去执行。因此，创建一段内嵌的javascript代码，动态地将脚本标签附加到文档后边，能提升渲染性能。

#### 异步——async

### 惰性加载

## 运行时性能

#### 跨作用域的缓存变量和属性

#### 核心javascript与frameworks的比较

#### 使用队列操作DOM（DocumentFragement）

#### 减少使用嵌套循环
