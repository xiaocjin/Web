## css盒子模型

#### w3c盒子标准

>总宽度 = margin-left + border-left + padding-left + width +height+ padding-right + border-right + margin-right

#### IE5~IE6盒子标准

>总宽度 = margin-left + width + margin-right

#### box-sizing

> box-sizing 用来改变 CSS 盒模型 ，从而改变元素高宽的计算方式。这个属性用于模拟那些非正确支持标准盒模型的浏览器的表现。


    content-box
      默认值，标准盒模型。 width 与 height 是内容区的宽与高， 不包括边框，内边距，外边距。
    padding-box
      width 与 height 包括内边距，不包括边框与外边距。
    border-box
      width 与 height 包括内边距与边框，不包括外边距。这是IE 怪异模式（Quirks mode）使用的 盒模型 。
