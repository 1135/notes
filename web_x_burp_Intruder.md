### burpsuite - Intruder - Options 可配置以下设置

> 官方https://portswigger.net/burp/help/intruder_options.html


```
* Request Headers
 * Update Content-Length header 
 * Set Connection: close

* Request Engine

* Attack Results

* Grep - Match 匹配
* Grep - Extract提取   批量提取信息
* Grep - Payloads载荷
* Redirections重定向
```


### burpsuite - Intruder - Positions  简介4种 Attack type攻击类型

Attack type[攻击类型] - [攻击设置]
  * Attack type:Sniper(狙击兵)
  * Attack type:Battering ram(攻城槌)
  * Attack type:Pitchfork(草叉)
  * Attack type:Cluster bomb(集束炸弹)

#### 1.Attack type:Sniper(狙击兵)

用§§标注多个position
```
如username=§1§&password=§2§&x=28&y=19
```

用载荷集合中的内容 按顺序替换 标注过的$position1$（用$$符号标注的第1个地方），发包...
直到第1个位置已经用过了载荷集1（payload set:1）的所有元素，第二个位置$position2$开始用载荷集1的所有元素。

攻击前配置的载荷如下

payload set:1 **载荷集合 强制只能为1** 不能设置payload set:2

Payload type:Simple list[可选其他模式]
```
Payload
a
b
c
d
```


发包结果
```
Position  Payload
1          a
1          b
1          c
1          d
2          a
2          b
2          c
2          d
```

#### 2.Attack type:Battering ram(攻城槌)

用§§标注多个position如
```
username=§1§&password=§2§&x=28&y=19
```

用载荷集合中的内容 按顺序替换 **所有**标注过的`$position$`（用$$符号标注的地方全都替换为同一内容），发包...

（下面的例子，看发出的包的内容，第1次发包时，三个position的内容都为a）

攻击前配置的载荷如下

payload set:1 **载荷集合 强制只能为1** 不能设置payload set:2

Payload type:Simple list[可选其他模式]

```
Payload
a
b
c
d
```

发包结果
```
Request  Payload
          a
          b
          c
          d
```


#### 3.Attack type:Pitchfork(草叉)

Pitchfork(草叉)【一一对应模式】

发包前

用载荷集合1中的内容 替换position1

用载荷集合2中的内容 替换position2

（最多支持5个集合，5个position）

发包

（本例共3次发包,交换Payload1和Payload2里的内容也一样只发3次包）

攻击前配置的载荷如下

**有几个posion 就必须设置几个 载荷集合 payload set**

payload set:1 **第1个 载荷集合**

Payload type:Simple list[可选其他模式]

```
Payload
X
Y
Z
```


payload set:2**第2个 载荷集合**

Payload type:Simple list[可选其他模式]

```
Payload
a
b
c
d
```

发包结果
```
Payload1 Payload2
X          a
Y          b
Z          c
```

#### 4.Attack type:Cluster bomb(集束炸弹)

Cluster bomb(集束炸弹)【笛卡尔积模式】

>笛卡尔积 
![{\displaystyle X\times Y=\left\{\left(x,y\right)\mid x\in X\land y\in Y\right\}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b511477a8bb079f00e37db2d8205df2787a648ad)
举例：

集合X是13个元素的点数集合{ A, K, Q, J, 10, 9, 8, 7, 6, 5, 4, 3, 2 }

集合Y是4个元素的花色集合{♠, ♥, ♦, ♣}

则这两个集合的笛卡儿积是有13*4=52个元素的标准扑克牌的集合{ (A, ♠), (K, ♠), ..., (2, ♠), (A, ♥), ..., (3, ♣), (2, ♣) }


第1次发包：

用载荷集合1中的第1条内容 替换position1

用载荷集合2中的第**1**条内容替换position2

第2次发包：

用载荷集合1中的第1条内容 替换position1（和上一个包中position2的内容一样）

用载荷集合2中的第**2**条内容替换position2

直到Payload1中的每条内容 与 Payload2中第1条内容都发过包了，positon2替换为下一条内容

开始发包：

用载荷集合1中的第1条内容 替换position1

用载荷集合2中的第**2**条内容替换position2

开始发包：

用载荷集合1中的第2条内容 替换position1

用载荷集合2中的第**2**条内容替换position2

开始发包：
用载荷集合1中的第3条内容 替换position1

用载荷集合2中的第**2**条内容替换position2

....

直到Payload1中的每条内容 与 Payload2中每条内容都组合发过包了，整体结束。


例，用§§标注多个position如username=§1§&password=§2§&x=28&y=19

攻击发包过程(共12次) 

Payload1中的每条内容 与 Payload2中的每条内容 都会组合发一次包

第一个集合的元素+第二个集合第1,2,3,4..个元素

攻击前配置的载荷如下

**有几个posion 就必须设置几个 载荷集合 payload set**

payload set:1

Payload type:Simple list[可选其他模式]

```
Payload
X
Y
Z
```


payload set:2

Payload type:Simple list[可选其他模式]

```
Payload
a
b
c
d
```


发包结果
```
Payload1 Payload2
X          a
Y          a
Z          a
X          b
Y          b
Z          b
X          c
Y          c
Z          c
X          d
Y          d
Z          d
```
