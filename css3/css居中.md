## 水平居中

#### inline & inline-*元素

>在块级（block）父元素在部的inline & inline-*元素，可以直接使用：


    .center-children {
      text-align: center;
    }

#### 块级元素

>在制定宽度的块级元素中，可以设定margin-left和margin-right的值为auto，使content居中


    .center-me {
      margin: 0 auto;
    }

#### 多个块级元素
>使用flex布局：


    .flex-center {
      display: flex;
      justify-content: center;
    }

## 垂直居中

#### inline & inline-*元素

>单行,设定等值的padding-top和padding-bottom

    .link {
      padding-top: 30px;
      padding-bottom: 30px;
    }

>多行，等padding-top和padding-bottom的方式可能也行，但如果不行的话，可以转换成table（display: table-cell）布局，在该布局下对子元素使用垂直居中（vertical-align: middle）。

#### 块级元素


    .parent {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
    
## 水平 & 垂直居中

#### 固定宽高

>使用负margin

#### 宽高未知

>使用transform负值

#### 使用flex布局

    .parent {
      display: flex;
      justify-content: center;
      align-items: center;
    }
