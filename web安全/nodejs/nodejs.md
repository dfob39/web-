1.fs模块
该模块主要用于对文件进行操作，主要涉及到读和写两种类型的函数
2.web服务器启动（http模块）

```plain
// 导入http模块
const http = require('http')
// 创建web 服务器实例
const server = http.createServer()
// 为服务器实例绑定request事件，监听客户端的请求
server.on( 'request', (req,res) => {
	// 只要有客户端来请求我们自己的服务器，就会触发request 事件，从而调用这个事件处理函数
    console.log( 'Someone visit our web server.' )
})

// 启动服务器
server.listen(8080, () =>{
	console.log('http server running at http://127.0.0.1:8080')
})
#调用模块都要都要先导入对应模块
```
3.nodejs弱类型比较
字符串和字符串进行比较会把字符串的第一个字符转成ASCII码后再进行比较，所以会出现console.log('111'<'3')的情况出现，且字符串和任何数字比较的结果都是false
数组比较，空数组直接比较都为false，数组之间比较只比较第一个值，数组永远小于非数值型字符串，数组和数值型也只比较第一个字符
![image.png](./nodejs.assert/1760885240155-bc272bd6-0adf-4940-8c6c-882537e3ddcc.png)

4.nodejs MD5绕过
两个不同对象在nodejs里面表示的类型是一致的，比如a={x:1},b={x:2},使用console.log输出后都是[object,object],因此进行MD5加密后的值显然是一致的
5.nodejs编码绕过
16进制编码绕过 console.log("a"==="\x61");//true
unicode编码绕过 console.log("a"==="\u0061");true
base64编码绕过  console.log(Buffer.from("dGVzdA==",'BASE64').toString())//test
6.命令执行
![image.png](./nodejs.assert/1760885629876-82a9e9e7-3b54-488a-948a-096e5f4ce382.png)

7.文件读写
![image.png](./nodejs.assert/1760885660689-78823d3c-4cc3-477f-9000-54fc3c79cda7.png)

8.rce bypass
  字符拼接
![image.png](./nodejs.assert/1760885706764-d5150d5c-dbc6-45b1-b2c8-89eec9ba62b5.png)

编码绕过
![image.png](./nodejs.assert/1760885741649-6fbd4f1d-b072-4a2f-924b-331d392d98a9.png)

模板拼接
![image.png](./nodejs.assert/1760885779531-494f0ce1-edf0-4b4b-be51-48600f7a46a9.png)

