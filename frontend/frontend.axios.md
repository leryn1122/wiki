
# Axios
Axios 基于 XHR API（XMLHttpRequest）而不是 Fetch API 的 HTTP Client 库，它轻量化的封装了 XHR。它是目前比较风靡的 HTTP Client 库，许多前端项目都使用二次封装的 axios 作为 HTTP 请求库。



Axios 请求时：后端 Spring 注解`@RequestBody`，`@RequestParam`对应的用法。
```javascript
let data = {
  username: "root",
  password: "123456",
};
axios
  .post("/api", qs.stringify(data))
  .then((response) => {
    console.log(response);
  });
```
```javascript
let data = {
  username: "root",
  password: "123456",
};
axios
  .post("/api", data)
  .then((response) => {
    console.log(response);
  });
```
