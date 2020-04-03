#### 前言

​		前几日在使用Github的OAuth服务做第三方登录后，在登录过程中偶尔会出现 **java.net.SocketException:Connection Reset**异常, 但是重启项目就能解决，现在彻底登录不了，一直报`Connetction Reset`.

#### 查询资料，初步了解异常详情

​		该异常在客户端和服务器端均有可能发生，引起该异常的原因有两个。

​		第一个是Connect reset by peer：就是如果一端的Socket被关闭，可能是主动关闭或者因为异常退出而引起的关闭，此时另一端仍发送数据，发送的第一个数据包引发该异常；另一个是Connection reset：一端退出，但退出时并未关闭该连接，另一端如果在从连接中读数据则会抛出该异常。

​		**总结：在连接断开后的读和写操作引起的。**

#### 定位异常

```java
public GithubUser getUser(String accessToken) {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
        .url("https://api.github.com/user?access_token=" + accessToken)
        .build();
    try (Response response = client.newCall(request).execute()) {	//此处出现异常
        String string = response.body().string();
        return JSON.parseObject(string, GithubUser.class);
    } catch (IOException e) {
        e.printStackTrace();
        return null;
    } 
}
```



#### 一探究竟

在Github官方文档提到，从11月13日开始，将禁止使用`access_token`作为查询参数访问API（以用户或GitHub应用程序的身份）

**e.g：使用`access_token`的查询参数**

如果您当前正在进行类似于以下内容的API调用

```
curl "https://api.github.com/user/repos?access_token=my_access_token"
```

相反，您应该在标头中发送令牌：

```
curl -H 'Authorization: token my_access_token' https://api.github.com/user/repos
```



#### 代码修改

```java
<Change>
Request request = new Request.Builder()
    .url("https://api.github.com/user?access_token=" + accessToken)
    .build(); 

<to>
Request request = new Request.Builder()
                .url("https://api.github.com/user")
                .header("Authorization", "token " + accessToken)
                .build();
```



参考Github相关文章：[通过查询参数弃用API身份验证](https://developer.github.com/changes/2020-02-10-deprecating-auth-through-query-param/)



*Thank you.*



