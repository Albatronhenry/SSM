[SpringMVC中使用Ajax POST请求以json格式](https://blog.csdn.net/u010648555/article/details/79084466)
-----------------
在项目中,SpringMv接收前端传递过来的json,有两种实现方式:

* ①使用request.getParameter("data");
```js
//修改前无法获取到json 
contentType : "application/json",
data : JSON.stringify(reqData),
```
```
  正常的post请求(不包括ajax请求)在http头中的content-type为`application/x-www-form-urlencoded`,这时在java后台可以通过request.getParameter("data")的形式获取.但是通过原生ajax请求时,在java后台通过request.getParameter("data")的形式却无法获取到传入的参数.
```
```js
//修改后可以获取到json
contentType : "application/x-www-form-urlencoded",(默认就是这个,所以可以不写)
data : reqData, //不需要使用JSON.stringify()
```
* ② 使用@RequestBody
```java
 @RequestMapping(value="/test",method=RequestMethod.POST)
    @ResponseBody
    public String closeSession(@RequestBody Object object){......}
```
