#### 使用OkHttp3做Github登录时获取Json数据

##### 在获取json请求数据时遇到两种情况

1、使用response.body().toString()，得到了com.squareup.okhttp3.Call$RealResponseBody@41c16aa8类型的	  字符串；

2、使用response.body().string()，得到了正确的json数据.

**解释：**1、`toString()`: 以字符串格式返回您的对象。

​			2、`.string()`: 返回你的response内容。

[stackoverflow上的解释](https://stackoverflow.com/questions/28300359/cant-get-okhttps-response-body-tostring-to-return-a-string)

