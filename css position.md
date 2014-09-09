## 相关定义

* static：无特殊定位，对象遵循正常文档流。top，right，bottom，left等属性不会被应用。
* relative：对象遵循正常文档流，但将依据top，right，bottom，left等属性在正常文档流中偏移位置。而其层叠通过z-index属性定义。
* absolute：对象脱离正常文档流，使用top，right，bottom，left等属性进行绝对定位。而其层叠通过z-index属性定义。
* fixed：对象脱离正常文档流，使用top，right，bottom，left等属性以窗口为参考点进行定位，当出现滚动条时，对象不会随着滚动。而其层叠通过z-index属性定义。

## 文档流
>将窗体自上而下分成一行行, 并在每行中按从左至右的顺序排放元素,即为文档流。只有三种情况会使得元素脱离文档流，分别是：浮动、绝对定位和相对定位。

## 静态定位(static) 
>static，无特殊定位，它是html元素默认的定位方式，即我们不设定元素的position属性时默认的position值就是static，它遵循正常的文档流对象，对象占用文档空间，该定位方式下，top、right、bottom、left、z-index等属性是无效的。

## 相对定位(relative) 
>相对定位相对的是它原本在文档流中的位置而进行的偏移，而我们也知道relative定位也是遵循正常的文档流，它没有脱离文档流，但是它的top/left/right/bottom属性是生效的，可以说它是static到absoult的一个中间过渡属性，最重要的是它还占有文档空间，而且占据的文档空间不会随 top / right / left / bottom 等属性的偏移而发生变动。

## 绝对定位(absoulte)
>使用absoult定位的元素脱离文档流后，就只能根据祖先类元素(父类以上)进行定位，而这个祖先类还必须是以postion非static方式定位的，absoulte是根据祖先类的border进行的定位。

## 固定定位(fixed)
> fixed定位，又称为固定定位，它和absoult定位一样，都脱离了文档流，并且能够根据top、right、left、bottom属性进行定位，但不同的是fixed是根据窗口为原点进行偏移定位的，也就是说它不会根据滚动条的滚动而进行偏移。

## z-index属性
>z-index，又称为对象的层叠顺序，它用一个整数来定义堆叠的层次，整数值越大，则被层叠在越上面，当然这是指同级元素间的堆叠，如果两个对象的此属性具有同样的值，那么将依据它们在HTML文档中流的顺序层叠，写在后面的将会覆盖前面的。
