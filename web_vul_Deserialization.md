### 基础 - Java 序列化与反序列化

* 序列化
  * 概念：把 Java对象 转换为 二进制字节序列 的过程
  * 实现：ObjectOutputStream类的 writeObject() 方法
* 反序列化
  * 概念：把 二进制字节序列 恢复为 Java对象 的过程
  * 实现：ObjectInputStream 类的 readObject() 方法

### 防御

* 原生反序列化防御
  * 避免用户输入可控 - 不要反序列化不可信的数据
  * 认证 - 给反序列数据加密签名且解密需在反序列之前
  * 认证 - 经过认证授权 才能使用反序列化接口
  * 限制访问源 - 反序列化服务只允许监听在本地

[CheatSheetSeries/Deserialization_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Deserialization_Cheat_Sheet.md)
