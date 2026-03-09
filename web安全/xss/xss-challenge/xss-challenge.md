1.无过滤直接弹窗
2.把value属性提前闭合，然后使用script直接弹
3.因为输入的参数有两个，尝试p1，发现不行，应该是因为多了两个双引号无法执行命令"<script>alert(document.domain)</script>"  然后就使用p2参数，这个可以直接弹
