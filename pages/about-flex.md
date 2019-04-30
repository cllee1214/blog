贴一下的概念：


flex-basis：在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

flex-shrink：项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

看一个列子：


```
 #content {
  display: flex;
  width: 500px;
}

#content div {
  flex-basis: 120px;
}

.box { 
  flex-shrink: 1;
}

.box1 { 
  flex-shrink: 2; 
}

```

```
  <div id="content">
        <div class="box" style="background-color:red;">1</div>
        <div class="box" style="background-color:lightblue;">1</div>
        <div class="box" style="background-color:yellow;">1</div>
        <div class="box1" style="background-color:brown;">2</div>
        <div class="box1" style="background-color:lightgreen;">2</div>
    </div>
```

即：一共5个item,每个item基本宽度为120。缩小占比分别为1:1:1:2:2。


外层总宽度为500。120*5肯定是大于500的。也就是说，flex-shrink生效，需要缩小。


那么每个item占比多少？  比如box1，占比为2，由于是缩小，所占比例越多，应该是缩得更多，基本宽度所剩下的也更少。所以 500*(2/7)的算法肯定是错误的。


正确的算法是怎么样的呢？这里的缩小比例，实际上指的是每一个item要分担溢出宽度的比例。这里的的基础宽度为120，所以理论上总宽度为120*5 = 600，但是外层总宽度为500，因此 600 - 500 = 100，即溢出宽度为100。那么这个100的宽度就给5个item按照比例分了。比如box2,实际的宽度应该为 120 - (100 * 2 / 7) = 91.42

如果每个item有10px的border?  溢出宽度为 (120 + 10 *2) * 5 - 500 = 200, box2实际宽度为 120 - (200 * 2 / 7) + 2 * 10 = 82.85714285714286

公式总结：

实际宽度 = 基础宽度 - (外层总宽度 - (基础宽度 + border) * item个数) / 缩小比例 + border