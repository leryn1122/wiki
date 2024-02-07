
# Axios
Axios 基于 XHR API（XMLHttpRequest）

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
