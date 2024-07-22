## 一、什么是Axios

```
Axios是基于Promise的Http客户端，可以在浏览器和node.js中使用。
```
## 二、为什么使用Axios

```html
Axios非常适合前后端数据交互，另一种请求后端数据的方式是vue-resource，vue-resource已经不再更新了，且只支持浏览器端使用，而Axios同时支持浏览器和Node端使用。

Vue开发者推荐使用更好的第三方工具，这就是Axios。
```
## 三、安装

### 3.1 npm

```javascript
npm install axios --save
 
```
### 3.2 cdn

```js
<script src="https://cdn.bootcss.com/axios/0.18.0/axios.min.js"></script>
```
## 四、使用

### 4.1 挂载

```javascript
Vue.prototype.$axios = axios;
```
### 4.2 使用

```javascript
var _this = this
          this.$axios.request({
            url:"http://127.0.0.1:8000/api/v1/course/",
            method:"GET"
          }).then(function (res) {
            //ajax请求发送成功后获取的响应的内容
          
          }).catch(function (res) {
            //ajax请求失败之后响应的内容，发生异常执行
          })
```
## 五、拦截器

```javascript
在请求或响应被 then 或 catch 处理前拦截它们。通常用于设置请求头
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });