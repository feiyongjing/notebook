### 参考：https://blog.csdn.net/qq_35930739/article/details/120799915?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-120799915-blog-56832167.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-120799915-blog-56832167.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=3


用BASE64算法加密后的字符串放在HTTP Request中的Header Authorization中发送给服务端， 这种方式叫HTTP基本认证(Basic Authentication)

Basic认证是一种较为简单的HTTP认证方式，客户端通过明文（Base64编码格式）传输用户名和密码到服务端进行认证，通常需要配合HTTPS来保证信息传输的安全。

HTTPBasic是由HTTP协议定义的最基础的认证方式。每次请求时，在请求头Authorization参数中附带用户/密码的base64编码。这个方式并不安全，不适合在web项目中使用。但它是一些现代主流认证的基础，而且在spring security的oauth中，内部认证默认就是用的HTTPBasic。
