# 常见加密算法

**对称加密**

```
DES、3DES、DESX、Blowfish、IDEA、RC4、RC5、RC6、AES
```

**非对称加密**

```
RSA、ECC（移动设备用）、Diffie-Hellman、El Gamal、DSA（数字签名用）
```

**Hash算法（摘要算法）**

```
MD2、MD4、MD5、HAVAL、SHA、SHA-1、HMAC、HMAC-MD5、HMAC-SHA1、bcrypt
```



# 常见场景

### 用户密码

用户密码以明文直接存入数据库极不安全，通常采用Hash算法进行加密然后存入。为增加暴力破解的难度，可以增加密码盐进行加密：

```ruby
password_hash = hash(password, salt)
```

加盐也只是增加了破解的复杂度，目前更常用的方法是在此基础上增加计算强度的`bcrypt`算法，其原理是增加计算的复杂度，以提高单次解密的时间，从而使暴力方式破解密文的时间极大的增加。



### 加密内容

对某些敏感内容可以采用独立密码对方式，先对独立密码进行校验（方法同用户密码）

```ruby
# 通过不同的Hash算法或者密码盐, 校验身份
password_hash = hash1(password, salt1)
# 再通过不同的hash算法或者密码盐生成hash_code
hash_code = hash2(password, salt2)
# 对敏感内容对称加密
encryption_text = symmetric(text,h2)
```



### 加密内容搜索

有一个实用的可搜索加密方案`SWP`，其大概思路是：

>  对内容进行拆词，加密每个单词。服务器对搜索内容进行加密计算，然后去内容中匹配对应的数据。

但该方案目前还没有落地使用。目前常用的搜索系统采用同步数据的方式，将数据库的数据单向同步到如`elasticsearch`的搜索引擎中。

还有一种思路就是：`同态加密`

> 它允许人们对密文进行特定形式的代数运算得到仍然是加密的结果，将其解密所得到的结果与对[明文](https://zh.wikipedia.org/wiki/明文)进行同样的运算结果一样。换言之，这项技术令人们可以在加密的数据中进行诸如检索、比较等操作，得出正确的结果，而在整个处理过程中无需对数据进行[解密](https://zh.wikipedia.org/wiki/解密)。其意义在于，真正从根本上解决将数据及其操作委托给第三方时的保密问题，例如对于各种[云计算](https://zh.wikipedia.org/wiki/云计算)的应用。

但目前来说该技术还不成熟。





# 密码发展简介

秘密书信的历史非常久远，其方式大致可分为两类，隐匿法和密码法。几千年来编码者和解码者的对决最终演变成了今天的密码学。



## 单套字母替代式密码

替代式密码法在军事上的应用首度记载于凯撒大帝的《高卢战记》。他将希腊字母替代罗马字母以把信息转译成敌人看不懂的符号。他还常把信息内容的字母改写成它后三位的字母来改写信息。通常我们把这一类替换法称为凯撒密码法:

> ABCDE =>  DEFGH

至少从后来的一千多年来看，凯撒密码法及其衍生的一些加密方法的破解是一件几乎不可能完成的任务。直到遇到了阿拉伯的密码分析家。

阿拉伯的神学家再研究古兰经时，会去研究书中的单词以及句型，以尝试证明每一句话都真的出自穆罕默德之口。不仅如此他们还会分析个别字母出现的频率。这些结果造成了日后密码分析学的一次重大突破。

即使是再千奇百怪的符号，在频率分析面前也会被打回原形。最难破解的要属法国国王路易十四使用的“大密码”，改密码为了避免字母的比例被频率分析识破，其密文对应加密的是每个单词的发音。但这套密码在400年后还是被频率分析法识破了。

归根结底，单套替换式密码的问题出现在，其密文终究只有一个对应的明文与之对应。



## 维吉尼亚密码

16世纪诞生的维吉尼亚密码则有效解决了这个问题。其大概思路是，将上述的凯撒密码分别以1到25的移位组成26套单套密码。在使用时，随机使用其中的多套密码来加密信息。

比如我需要加密只包含ABCD字母组成的信息时，先构建四套密码其中ABCD分别对应：

```
1) BCDA
2) CDAB
3) DABC
4) ABCD
```

然后我可以随机选择其中的两套，比如2）3）来依次加密`AAABBB`这样第一个`A`对应的就是`C`，第二个`A`对应的就是`D`，最后密文则是：`CDCADA`。

这样明文`A`可以被`C`或`D`替换，同时`C`或`D`又可以表示别的字符，如果解码者不知道使用的第几套密码，以及顺序，是没有办法解密信息的。

多套密码加密既是改密码的优点，但也是改密码的缺点。原因在于，根据各个密文字母出现的间隔，可以推测出密码重复的周期。在一段较长的密文中，多个重复出现的字符串的间隔数，其共同的因子极大概率是该密文加密的周期。这样一来便可以使用频率分析法，来分析各套密码的明文。比如我们假设某密文的周期是5，于是我们取出密文中的第1、6、11...个字符使用频率分析法解密，然后再取第2、7、12...个字符，以此类推。



## 机械化加密

### ”奇迷“

除了上述问题，维吉尼亚密码还存在一个问题，那就是效率。在工业革命之前，编码和解码会耗费大量的人力，以至于维吉尼亚密码没办法普及。此后随着技术的革新，让我们把二战期间德军使用的通信设备”奇迷“上。

其大致运行机制是，键盘键入的字母产生电流并通过三个编码器最后通过反射器连接对应字母的灯板，解密时将密文输入，则可以得到原来的信息。三个编码器类似水表盘的数字，每当键盘键入一个字母以后，第一个编码器就会转动1/26，当第一个编码器转动一圈的时候，第二个编码器也会转动1/26，以此类推。

我们可以看到，其实”奇迷“不过是维吉尼亚密码的机械增强版。只不过其加密模式比维吉尼亚密码多得多，而与维吉尼亚密码不同的是”奇迷“所使用的密码只需要约定一些相当于密钥的初始配置信息，比如三个编码器的顺序，编码器的起始位置等。

而”奇迷“也不是完全无法破解的，如果有对照文（即密文和明文的对照）可以从”奇迷“的初始设置开始一次次尝试，直到输入的明文对应到密文为止。其实二战时期的同盟国便是以这种方式来破解”奇米“的，只不过运用了一些技巧，简化了尝试的次数。

### 纳瓦霍密语

同是二战时期的美国，则采用了一种意想不到的方式传递信息——方言。

美国军方直接挑选并培训纳瓦霍族人使用他们本族的语言翻译并传递信息，纳瓦霍族是北美洲的本土部落，而且除了美国极少数研究他们的学者外没有人懂这种语言。在太平洋战场上，纳瓦霍密码为美军提供了极大的帮助。日本曾破译了美国空军的密码，但是对纳瓦霍密码却束手无策。

他们的密码也是少数有史以来从未被破解的密码之一。



## 信息时代的加密

第二次世界大战以后，随着信息技术的发展，信息演变成了通过二进制的ASCLL码来传递。信息的加密和解密也演变成了对二进制数的数学运算。比如IBM公司曾研发出的”魔王“便是根据密钥对加密信息的每64位做16次非常复杂的代替法运算，来加密整条信息。收信人根据相同的逆运算来机密信息。这种后来被称为对称加密的算法，在1976年被美国国家安全局NSA限定密钥为56位后确立为标准，称之为数据加密标准（Data Encryption Standard，DES）。

DES解决了加密问题，但是密钥的传输仍是个问题。如果密钥泄露，DES加密的信息仍旧是不安全的。所以问题的关键聚焦再了密钥的传递上。1976年，黑尔曼提出了一套通过单向函数来传递密钥的方法，这使得发送者和接收者双发可以安全的传递密钥。此后由瑞韦斯特、薛米尔和艾多曼三人在这套理论的基础上进行演进，提出了至今还使用的公开密钥加密系统RSA。



## 结语

以上内容是看完Simon Singh的《The Code Book》一书的一些简单的梳理。作者成书于上世纪末，如今很多技术有了新的革新，可能内容有些滞后。书中作者在最后一章中还提到了绝对安全的量子机密技术。有兴趣的话可以翻阅原著。