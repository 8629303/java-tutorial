> - 用户管理模块：如何保证用户数据安全：https://juejin.cn/post/6916150628955717646
> - SpringBoot接口加密与解密：https://blog.csdn.net/lemon_TT/article/details/125728991
>    Java安全密码学：https://blog.csdn.net/qq_36154832/category_10146305.html
>    Java加密与解密的艺术：https://blog.csdn.net/Jarbein/article/details/103836551
>    【网络】Java密码安全：https://blog.csdn.net/hancoder/article/details/111464250
> - Java中的调皮的AES实例：https://blog.csdn.net/m0_46379371/article/details/108182692
> - Java中常用的加密与解密：https://blog.csdn.net/weixin_43860260/article/details/122101197（**Spring源码与JVM解读**）
> - Java加密与解密：https://blog.csdn.net/qq_35760825/article/details/126356394
> - 通俗易懂的Java安全密码学教程：https://edu.csdn.net/course/detail/29229
> - 接口数据: 如何安全传输存储：https://glory.blog.csdn.net/article/details/121642532
>

# Java 多种加密和解密方式

> Java 多种加密和解密方式：https://blog.csdn.net/guanshengg/article/details/126436675

## 1、BASE64 加解密

Base64 编码是我们程序开发中经常使用到的编码方法，它用 64 个可打印字符来表示二进制数据。这 64 个字符是：小写字母 a-z、大写字母 A-Z、数字 0-9、符号"+"、"/"（再加上作为垫字的"="，实际上是 65 个字符），其他所有符号都转换成这个字符集中的字符。Base64 编码通常用作存储、传输一些二进制数据编码方法，所以说它本质上是一种将二进制数据转成文本数据的方案。

Java 8 内置了 Base64 编码的编码器和解码器。Base64工具类提供了一套静态方法获取下面三种BASE64编解码器：

- 基本：输出被映射到一组字符A-Za-z0-9+/，编码不添加任何行标，输出的解码仅支持A-Za-z0-9+/。
- URL：输出映射到一组字符A-Za-z0-9+_，输出是URL和文件。
- MIME：输出隐射到MIME友好格式。输出每行不超过76字符，并且使用’\r’并跟随’\n’作为分割。编码输出最后没有行分割。

```java
package com.example.examplejavajwt.utils;

import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64Util {
    /***
     * Base64 加密简单字符串
     */
    public static String encodeString(String key) {
        return Base64.getEncoder().encodeToString(key.getBytes(StandardCharsets.UTF_8));
    }

    /***
     * Base64 解密简单字符串
     */
    public static String decoderString(String key) {
        return new String(Base64.getDecoder().decode(key), StandardCharsets.UTF_8);
    }

    /***
     * Base64 加密URL
     * URL加密和上述基本加密类似，它使用URL和文件名安全Base64字母表(没有‘+’和‘/’,增加‘-’和‘_’)，不添加任何换行隔
     */
    public static String encodeUrl(String key) {
        return Base64.getUrlEncoder().encodeToString(key.getBytes(StandardCharsets.UTF_8));
    }

    /***
     * Base64 加密URL
     */
    public static String decoderUrl(String key) {
        return new String(Base64.getUrlDecoder().decode(key), StandardCharsets.UTF_8);
    }

    /***
     * Base64 加密Mime
     * MIME加密工具生成Base64加密输出，输出被映射到“A-Za-z0-9+/”字符集中，
     * 编码输出必须以每行不超过76个字符的形式表示，并使用后面跟着的回车符’ ’和换行符’ ’为分隔符,
     */
    public static String encodeMime(String key) {
        return Base64.getMimeEncoder().encodeToString(key.getBytes(StandardCharsets.UTF_8));
    }

    /***
     * Base64 加密Mime
     */
    public static String decoderMime(String key) {
        return new String(Base64.getMimeDecoder().decode(key), StandardCharsets.UTF_8);
    }

    public static void main(String[] args) {
        String encodeString = encodeString("乘风破浪");
        String decoderString = decoderString(encodeString);
        String encodeUrl = encodeUrl("www.baidu.com");
        String decoderUrl = decoderUrl(encodeUrl);
        String encodeMime = encodeMime("飞驰人生");
        String decoderMime = decoderMime(encodeMime);
        System.out.println(encodeString);
        System.out.println(decoderString);
        System.out.println(encodeUrl);
        System.out.println(decoderUrl);
        System.out.println(encodeMime);
        System.out.println(decoderMime);
    }
}
```

```
5LmY6aOO56C05rWq
乘风破浪
d3d3LmJhaWR1LmNvbQ==
www.baidu.com
6aOe6amw5Lq655Sf
飞驰人生
```



## 2、MD5 加密【非对称加密】

什么是MD5：MD5加密全称是 Message-Digest Algoorithm 5（信息-摘要算法），它对信息进行摘要采集，再通过一定的位运算，最终获取加密后的MD5字符串。

MD5有哪些特点：

1. 针对不同长度待加密的数据、字符串等等，其都可以返回一个固定长度的MD5加密字符串。（通常32位的16进制字符串）
2. 运算简便，且可实现方式多样，通过一定的处理方式也可以避免碰撞算法的破解。
3. 对于一个固定的字符串。数字等等，MD5加密后的字符串是固定的，也就是说不管MD5加密多少次，都是同样的结果。
4. MD5 是一种单向加密算法，几乎不解密或可逆，除非维护一个庞大的Key-Value数据库来进行碰撞破解，否则几乎无法解开。
5. MD5 主要用做数据一致性验证、数字签名和安全访问认证，而不是用作加密（比如登录密码验证）
6. MD5 还经常用于校验文件，因为如果文件的 MD5 不一样，说明文件内容也是不一样的。

```java
import org.springframework.util.DigestUtils;
import java.math.BigInteger;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
 
public class MD5Util {

    /***
     * 1. 使用JDK自带的API实现：MD5加密（生成唯一的MD5值）
     */
    public static String encryMD5JDK(String data) throws Exception {
        // 生成一个MD5加密计算摘要
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        // 计算md5函数
        md5.update(data.getBytes(StandardCharsets.UTF_8));
        /**
         * digest()最后确定返回md5 hash值，返回值为8位字符串。
         * 因为 md5 hash值是16位的hex值，实际上就是8位的字符
         * BigInteger函数则将8位的字符串转换成16位hex值，用字符串来表示；得到字符串形式的hash值
         * 一个byte是八位二进制，也就是2位十六进制字符（2的8次方等于16的2次方）
         */
        return new BigInteger(1, md5.digest()).toString(16); // 16是表示转换为16进制数
    }

    /***
     * Spring的DigestUtils工具类：MD5加密（生成唯一的MD5值）
     */
    public static String encryMD5Spring(String data) throws Exception {
        return DigestUtils.md5DigestAsHex((data).getBytes(StandardCharsets.UTF_8));
    }

    
    public static void main(String[] args) throws Exception {
        System.out.println(encryMD5JDK("天天开心"));
        System.out.println(encryMD5Spring("天天开心"));
    }
}
```



## 3、DES 加解密【对称加密】

DES（Data Encryption Standard）是一种对称加密算法，所谓对称加密算法就是：加密和解密使用相同密钥的算法。DES 加密算法出自 IBM 的研究，后来被美国政府正式采用，之后开始广泛流传。但近些年使用越来越少，因为 DES 使用 56 位密钥，以现代的计算能力，24 小时内即可被破解。

顺便说一下 3DES（Triple DES），它是 DES 向 AES 过渡的加密算法，使用 3 条 56 位的密钥对数据进行三次加密。是 DES 的一个更安全的变形。它以 DES 为基本模块，通过组合分组方法设计出分组加密算法。比起最初的 DES，3DES 更为安全。

使用 Java 实现 DES 加密解密，注意密码长度要是 8 的倍数。加密和解密的 Cipher 构造参数一定要相同，不然会报错。

数据加密标准算法,和BASE64最明显的区别就是有一个工作密钥，该密钥既用于加密、也用于解密，并且要求密钥是一个长度至少大于8位的字符串，示例：

```java
import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.security.SecureRandom;
import java.util.Base64;
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;

public class DesUtil {
    private static Key key;
    static {
        try {
            // 生成DES算法对象
            KeyGenerator generator = KeyGenerator.getInstance("DES");
            // 运用SHA1安全策略, 并设置上密钥种子（根据参数生成key）
            SecureRandom secureRandom = new SecureRandom("123456".getBytes(StandardCharsets.UTF_8));
            // 初始化基于SHA1的算法对象
            generator.init(secureRandom);
            // 生成密钥对象
            key = generator.generateKey();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /***
     * 获取加密的信息
     * @param str
     * @return
     */
    public static String getEncryptString(String str) {
        try {
            // 获取加密对象, Cipher对象实际完成加密操作
            Cipher cipher = Cipher.getInstance("DES");
            // 初始化密码信息, 用密钥key初始化Cipher对象(加密)
            cipher.init(Cipher.ENCRYPT_MODE, key);
            // 加密
            byte[] doFinal = cipher.doFinal(str.getBytes(StandardCharsets.UTF_8));
            // byte[] to encode好的String 并返回
            return Base64.getEncoder().encodeToString(doFinal);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /***
     * 获取解密之后的信息
     * @param str
     * @return
     */
    public static String getDecryptString(String str) {
        try {
            // 获取解密对象, Cipher对象实际完成解密操作
            Cipher cipher = Cipher.getInstance("DES");
            // 初始化解密信息, 用密钥初始化Cipher对象(解密)
            cipher.init(Cipher.DECRYPT_MODE, key);
            // 解密
            byte[] doFinal = cipher.doFinal(Base64.getDecoder().decode(str));
            // 返回String
            return new String(doFinal, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        String str = "hello world";
        String encrypt = getEncryptString(str);
        String decrypt = getDecryptString(encrypt);
        System.out.println("原内容：" + str);
        System.out.println("加密：" + encrypt);
        System.out.println("解密：" + decrypt);
    }
}
```

```
原内容：hello world
加密：MHiD/GLsh48cHG/5XI/+Kw==
解密：hello world
```



## 4、AES 加解密【对称加密】

AES 高级加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称 Rijndael 加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的 DES，已经被多方分析且广为全世界所使用。简单说就是 DES 的增强版，比 DES 的加密强度更高。

AES 与 DES 一样，一共有四种加密模式：电子密码本模式（ECB）、加密分组链接模式（CBC）、加密反馈模式（CFB）和输出反馈模式（OFB）。关于加密模式的介绍，推荐这篇文章：高级加密标准AES的工作模式（ECB、CBC、CFB、OFB）

示例代码：

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class AESUtil {

    public static final String algorithm = "AES";
    // AES/CBC/NOPaddin
    // AES 默认模式
    // 使用CBC模式, 在初始化Cipher对象时, 需要增加参数, 初始化向量IV : IvParameterSpec iv = new
    // IvParameterSpec(key.getBytes());
    // NOPadding: 使用NOPadding模式时, 原文长度必须是8byte的整数倍
    public static final String transformation = "AES/CBC/NOPadding";
    public static final String key = "1234567812345678";

    /***
     * 加密
     * @param original 需要加密的参数（注意必须是16位）
     * @return
     * @throws Exception
     */
    public static String encryptByAES(String original) throws Exception {
        // 获取Cipher
        Cipher cipher = Cipher.getInstance(transformation);
        // 生成密钥
        SecretKeySpec keySpec = new SecretKeySpec(key.getBytes(), algorithm);
        // 指定模式(加密)和密钥
        // 创建初始化向量
        IvParameterSpec iv = new IvParameterSpec(key.getBytes());
        cipher.init(Cipher.ENCRYPT_MODE, keySpec, iv);
        // cipher.init(Cipher.ENCRYPT_MODE, keySpec);
        // 加密
        byte[] bytes = cipher.doFinal(original.getBytes());

        return Base64.getEncoder().encodeToString(bytes);
    }

    /**
     * 解密
     *
     * @param encrypted 需要解密的参数
     * @return
     * @throws Exception
     */
    public static String decryptByAES(String encrypted) throws Exception {
        // 获取Cipher
        Cipher cipher = Cipher.getInstance(transformation);
        // 生成密钥
        SecretKeySpec keySpec = new SecretKeySpec(key.getBytes(), algorithm);
        // 指定模式(解密)和密钥
        // 创建初始化向量
        IvParameterSpec iv = new IvParameterSpec(key.getBytes());
        cipher.init(Cipher.DECRYPT_MODE, keySpec, iv);
        // cipher.init(Cipher.DECRYPT_MODE, keySpec);
        // 解密
        byte[] bytes = cipher.doFinal(Base64.getDecoder().decode(encrypted));

        return new String(bytes);
    }

    public static void main(String[] args) throws Exception {
        String str = "1234567812345678";
        String encrypt = encryptByAES(str);
        String decrypt = decryptByAES(encrypt);
        System.out.println("原内容：" + str);
        System.out.println("加密：" + encrypt);
        System.out.println("解密：" + decrypt);
    }
}
```

```
原内容：1234567812345678
加密：muj9ArNAKIoOe7/w8LpU1g==
解密：1234567812345678
```



## 5、HMAC 散列消息鉴别码【非对称加密】

HMAC（Hash Message Authentication Code，散列消息鉴别码）。使用一个密钥生成一个固定大小的小数据块，即MAC，并将其加入到消息中，然后传输。接收方利用与发送方共享的密钥进行鉴别认证，示例：

```java
import javax.crypto.KeyGenerator;
import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class HMACUtil {

    public static final String KEY_MAC = "HmacMD5";

    /***
     * 初始化HMAC密钥
     * @return
     * @throws Exception
     */
    public static String initMacKey() throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_MAC);
        SecretKey secretKey = keyGenerator.generateKey();
        return Base64.getEncoder().encodeToString(secretKey.getEncoded());
    }

    /**
     * HMAC加密
     *
     * @param data
     * @param key
     * @return
     * @throws Exception
     */
    public static String encryHMAC(byte[] data, String key) throws Exception {
        SecretKey secretKey = new SecretKeySpec(Base64.getDecoder().decode(key), KEY_MAC);
        Mac mac = Mac.getInstance(secretKey.getAlgorithm());
        mac.init(secretKey);
        return Base64.getEncoder().encodeToString(mac.doFinal());
    }

    public static void main(String[] args) throws Exception {
        String str = "hello world";
        String encryHMAC = encryHMAC(str.getBytes(StandardCharsets.UTF_8), initMacKey());
        System.out.println("原内容：" + str);
        System.out.println("加密：" + encryHMAC);
    }
}
```

```
原内容：hello world
加密：uZkyy+EWzlb9iBQbmbfwhA==
```



## 6、恺撒加密【对称加密】

在密码学中，恺撒密码是一种最简单并且最广为人知的加密技术。凯撒算法属于古典密码，是对称加密算法最简单的形式。

它是一种替换加密的技术，明文中的所欲字母都在字母表上向后（或向前）按照一个固定的数目进行偏移后被替换成密文。

例如：当偏移量是3的时候，所有的字母A将被替换成D，B变成E，以此类推。

这个加密方法是以恺撒的名字命名的，当年恺撒曾用此方法与其将军们进行联系。

恺撒密码通常被座位其他更复杂的加密方法中的一个步骤。

```java
public class KaisaUtil {
    /***
     * 使用凯撒加密方式加密数据
     * @param data 原文
     * @param key 密钥
     * @return 加密后的字符
     */
    private static String encryptKaisa(String data, int key) {
        // 将字符串转换为数组
        char[] chars = data.toCharArray();
        StringBuffer buffer = new StringBuffer();
        // 遍历数组
        for (char aChar : chars) {
            // 获取字符的ASCII编码
            int asciiCode = aChar;
            // 偏移数据
            asciiCode += key;
            // 将偏移后的数据转为字符
            char result = (char) asciiCode;
            // 拼接数据
            buffer.append(result);
        }
        return buffer.toString();
    }

    /**
     * 使用凯撒加密方式解密数据
     * @param encryptedData :密文
     * @param key           :密钥
     * @return : 源数据
     */
    private static String decryptKaiser(String encryptedData, int key) {
        // 将字符串转为字符数组
        char[] chars = encryptedData.toCharArray();
        StringBuilder sb = new StringBuilder();
        // 遍历数组
        for (char aChar : chars) {
            // 获取字符的ASCII编码
            int asciiCode = aChar;
            // 偏移数据
            asciiCode -= key;
            // 将偏移后的数据转为字符
            char result = (char) asciiCode;
            // 拼接数据
            sb.append(result);
        }
        return sb.toString();
    }


    public static void main(String[] args) {
        String str = "open fire";
        String encode = encryptKaisa(str, 3);
        System.out.println("加密后：" + encode);

        String decode = decryptKaiser(encode, 3);
        System.out.println("解密后：" + decode);
    }
}
```

```
加密后：rshq#iluh
解密后：open fire
```



## 7、SHA 安全散列算法【非对称加密】

SHA（Secure Hash Algorithm）全名叫做安全散列算法，是FIPS所认证的安全散列算法。能计算出一个数字消息所对应到的，长度固定的字符串（又称消息摘要）的算法。且若输入的消息不同，它们对应到不同字符串的机率很高。

数字签名等密码学应用中重要的工具，被广泛地应用于电子商务等信息安全领域，示例：

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

/**
 * SHA(Secure Hash Algorithm，安全散列算法），
 * 数字签名等密码学应用中重要的工具，被广泛地应用于电子商务等信息安全领域。
 * 虽然，SHA与MD5通过碰撞法都被破解了， 但是SHA仍然是公认的安全加密算法，较之MD5更为安全。
 */
public class SHAUtil {
    /***
     * SHA加密（比MD5更安全）
     * @param data
     * @return
     * @throws Exception
     */
    public static byte[] encryptSHA(byte[] data) throws Exception {
        MessageDigest sha = MessageDigest.getInstance("SHA");
        // sha.digest(data) == sha.update(data);sha.digest();
        return sha.digest(data);
    }

    /***
     * 指定SHA类型加密（比MD5更安全）
     * @param data
     * @param shaType SHA/SHA-1/SHA-256/SHA-512  (注意:SHA = SHA-1)
     * @return
     * @throws Exception
     */
    public static String SHAEncrypt(final String data, String shaType) {
        try {
            MessageDigest messageDigest = MessageDigest.getInstance(shaType);
            byte[] hash = messageDigest.digest(data.getBytes());
            StringBuffer hexValue = new StringBuffer();
            for (byte b : hash) {
                // 将其中的每个字节转成十六进制字符串：byte类型的数据最高位是符号位，通过和0xff进行与操作，转换为int类型的正整数。
                String toHexString = Integer.toHexString(b & 0xff);
                hexValue.append(toHexString.length() == 1 ? "0" + toHexString : toHexString);
            }
            return hexValue.toString();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    // SHA-256加密
    public static String SHA256Encrypt(String sourceStr) {
        try {
            // 获取 SHA-256 MessageDigest对象
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
            // 加密
            byte[] hash = messageDigest.digest(sourceStr.getBytes(StandardCharsets.UTF_8));

            // 把密文转换成十六进制的字符串形式
            StringBuffer hexString = new StringBuffer();
            // 字节数组转换为 十六进制数
            for (int i = 0; i < hash.length; i++) {
                // 将其中的每个字节转成十六进制字符串：byte类型的数据最高位是符号位，通过和0xff进行与操作，转换为int类型的正整数。
                String shaHex = Integer.toHexString(hash[i] & 0xFF);
                if (shaHex.length() == 1) {
                    hexString.append(0);
                }
                hexString.append(shaHex);
            }
            return hexString.toString();
            // return Hex.encodeHexString(hash); // apache common 包的
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        System.out.println(SHAEncrypt("1234567890", "SHA"));
        System.out.println(SHAEncrypt("1234567890", "SHA-1"));
        System.out.println(SHAEncrypt("1234567890", "SHA-256"));
        System.out.println(SHAEncrypt("1234567890", "SHA-512"));
        System.out.println(SHA256Encrypt("1234567890"));
    }
}
```

```
01b307acba4f54f55aafc33bb06bbbf6ca803e9a
01b307acba4f54f55aafc33bb06bbbf6ca803e9a
c775e7b757ede630cd0aa1113bd102661ab38829ca52a6422ab782862f268646
12b03226a6d8be9c6e8cd5e55dc6c7920caaa39df14aab92d5e3ea9340d1c8a4d3d0b8e4314f1f6ef131ba4bf1ceb9186ab87c801af0d5c95b1befb8cedae2b9
c775e7b757ede630cd0aa1113bd102661ab38829ca52a6422ab782862f268646
```



## 8、RSA 加解密【非对称加密】

RSA算法是一种非对称加密算法，所谓非对称就是该算法需要一对密钥，若使用其中一个加密，则需要用另一个才能解密。目前它是最有影响力和最常用的公钥加密算法，能够抵抗已知的绝大多数密码攻击。从提出到现今的三十多年里，经历了各种攻击的考验，逐渐为人们接受，普遍认为是目前最优秀的公钥方案之一。

该算法基于一个的数论事实：将两个大质数相乘十分容易，但是想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。由于进行的都是大数计算，RSA 最快的情况也比 DES 慢上好几倍，比对应同样安全级别的对称密码算法要慢 1000 倍左右。所以 RSA 一般只用于少量数据加密，比如说交换对称加密的密钥。

使用 RSA 加密主要有这么几步：生成密钥对、公开公钥、公钥加密私钥解密、私钥加密公钥解密。

加解密核心步骤（注意：RSA加密包含Base64加密、byte[]和十六进制的转换）：

1. 初始化密钥对。
2. 获取公钥字符串： 初始化后的密钥对 -> 取公钥 -> Base64编码 -> 公钥字符串。
3. 生成私钥字符串： 初始化后的密钥对 -> 取私钥 -> Base64编码 -> 私钥字符串。
4. 公钥加密： 明文 -> 获取公钥（公钥字符串 -> Base64解码 -> 公钥） -> 公钥加密 -> Hex编码 -> 密文。
5. 私钥解密： 密文 -> 获取私钥（私钥字符串 -> Base64解码 -> 私钥） -> 私钥解密 -> Hex解码 -> 明文。

测试地址：

1. RSA密钥对：http://web.chacuo.net/netrsakeypair
2. RSA公钥加密：http://tool.chacuo.net/cryptrsapubkey
3. RSA私钥解密：http://tool.chacuo.net/cryptrsaprikey

```java
import javax.crypto.Cipher;
import org.apache.commons.io.FileUtils;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.nio.charset.Charset;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

public class RsaUtil {
    /**
     * 初始化密钥对生成器时，指定密钥大小的整数值（安全漏洞，长度至少为2048）
     */
    private static final int KEY_PAIR_INIT_SIZE = 2048;
    /**
     * RSA最大加密明文大小
     */
    private static final int MAX_ENCRYPT_BLOCK = 245;
    /**
     * RSA最大解密密文大小，
     * RSA 位数 如果采用1024 上面最大加密和最大解密则须填写: 117 128
     * RSA 位数 如果采用2048 上面最大加密和最大解密则须填写: 245 256
     */
    private static final int MAX_DECRYPT_BLOCK = 256;


    /**
     * 生成密钥对并保存在本地文件中
     *
     * @param algorithm : 算法(这里使用 RSA)
     * @param pubPath   : 公钥保存路径
     * @param priPath   : 私钥保存路径
     * @throws Exception
     */
    private static void generateKeyToFile(String algorithm, String pubPath, String priPath) throws Exception {
        // 获取密钥对生成器, 使用RSA
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(algorithm);
        // 设置密钥对的 bit 数，越大越安全, 此处可配置可不配置
         // keyPairGenerator.initialize(KEY_PAIR_INIT_SIZE);
        // 获取密钥对
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        // 获取公钥
        PublicKey publicKey = keyPair.getPublic();
        // 获取私钥
        PrivateKey privateKey = keyPair.getPrivate();
        // 获取byte数组
        byte[] publicKeyEncoded = publicKey.getEncoded();
        byte[] privateKeyEncoded = privateKey.getEncoded();
        // 进行Base64编码
        String publicKeyString = Base64.getEncoder().encodeToString(publicKeyEncoded);
        String privateKeyString = Base64.getEncoder().encodeToString(privateKeyEncoded);
        // 保存文件
        FileUtils.writeStringToFile(new File(pubPath), publicKeyString, Charset.forName("UTF-8"));
        FileUtils.writeStringToFile(new File(priPath), privateKeyString, Charset.forName("UTF-8"));
    }

    /**
     * 从文件中加载公钥
     *
     * @param algorithm : 算法
     * @param filePath  : 文件路径
     * @return : 公钥
     * @throws Exception
     */
    private static PublicKey loadPublicKeyFromFile(String algorithm, String filePath) throws Exception {
        // 将文件内容转为字符串
        String keyString = FileUtils.readFileToString(new File(filePath), Charset.forName("UTF-8"));
        return loadPublicKeyFromString(algorithm, keyString);
    }

    /**
     * 从字符串中加载公钥
     *
     * @param algorithm : 算法
     * @param keyString : 公钥字符串
     * @return : 公钥
     * @throws Exception
     */
    private static PublicKey loadPublicKeyFromString(String algorithm, String keyString) throws Exception {
        // 进行Base64解码
        byte[] decode = Base64.getDecoder().decode(keyString);
        // 获取密钥工厂
        KeyFactory keyFactory = KeyFactory.getInstance(algorithm);
        // 构建密钥规范
        X509EncodedKeySpec keyspec = new X509EncodedKeySpec(decode);
        // 获取公钥
        return keyFactory.generatePublic(keyspec);

    }

    /**
     * 从文件中加载私钥
     *
     * @param algorithm : 算法
     * @param filePath  : 文件路径
     * @return : 私钥
     * @throws Exception
     */
    private static PrivateKey loadPrivateKeyFromFile(String algorithm, String filePath) throws Exception {
        // 将文件内容转为字符串
        String keyString = FileUtils.readFileToString(new File(filePath), Charset.forName("UTF-8"));
        return loadPrivateKeyFromString(algorithm, keyString);
    }

    /**
     * 从字符串中加载私钥
     *
     * @param algorithm : 算法
     * @param keyString : 私钥字符串
     * @return : 私钥
     * @throws Exception
     */
    private static PrivateKey loadPrivateKeyFromString(String algorithm, String keyString) throws Exception {
        // 进行Base64解码
        byte[] decode = Base64.getDecoder().decode(keyString);
        // 获取密钥工厂
        KeyFactory keyFactory = KeyFactory.getInstance(algorithm);
        // 构建密钥规范
        PKCS8EncodedKeySpec keyspec = new PKCS8EncodedKeySpec(decode);
        // 生成私钥
        return keyFactory.generatePrivate(keyspec);
    }

    /**
     * 使用密钥加密数据
     *
     * @param algorithm      : 算法
     * @param input          : 原文
     * @param key            : 密钥
     * @param maxEncryptSize : 最大加密长度(需要根据实际情况进行调整)
     * @return : 密文
     * @throws Exception
     */
    private static String encrypt(String algorithm, String input, Key key, int maxEncryptSize) throws Exception {
        // 获取Cipher对象
        Cipher cipher = Cipher.getInstance(algorithm);
        // 初始化模式(加密)和密钥
        cipher.init(Cipher.ENCRYPT_MODE, key);
        // 将原文转为byte数组
        byte[] data = input.getBytes();
        // 总数据长度
        int total = data.length;
        // 输出流
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        decodeByte(maxEncryptSize, cipher, data, total, baos);
        // 对密文进行Base64编码
        return Base64.getEncoder().encodeToString(baos.toByteArray());

    }

    /**
     * 解密数据
     *
     * @param algorithm      : 算法
     * @param encrypted      : 密文
     * @param key            : 密钥
     * @param maxDecryptSize : 最大解密长度(需要根据实际情况进行调整)
     * @return : 原文
     * @throws Exception
     */
    private static String decrypt(String algorithm, String encrypted, Key key, int maxDecryptSize) throws Exception {
        // 获取Cipher对象
        Cipher cipher = Cipher.getInstance(algorithm);
        // 初始化模式(解密)和密钥
        cipher.init(Cipher.DECRYPT_MODE, key);
        // 由于密文进行了Base64编码, 在这里需要进行解码
        byte[] data = Base64.getDecoder().decode(encrypted);
        // 总数据长度
        int total = data.length;
        // 输出流
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        decodeByte(maxDecryptSize, cipher, data, total, baos);
        // 输出原文
        return baos.toString();

    }

    /**
     * 分段处理数据
     *
     * @param maxSize : 最大处理能力
     * @param cipher  : Cipher对象
     * @param data    : 要处理的byte数组
     * @param total   : 总数据长度
     * @param baos    : 输出流
     * @throws Exception
     */
    private static void decodeByte(int maxSize, Cipher cipher, byte[] data, int total, ByteArrayOutputStream baos) throws Exception {
        // 偏移量
        int offset = 0;
        // 缓冲区
        byte[] buffer;
        // 如果数据没有处理完, 就一直继续
        while (total - offset > 0) {
            // 如果剩余的数据 >= 最大处理能力, 就按照最大处理能力来加密数据
            if (total - offset >= maxSize) {
                // 加密数据
                buffer = cipher.doFinal(data, offset, maxSize);
                // 偏移量向右侧偏移最大数据能力个
                offset += maxSize;
            } else {
                // 如果剩余的数据 < 最大处理能力, 就按照剩余的个数来加密数据
                buffer = cipher.doFinal(data, offset, total - offset);
                // 偏移量设置为总数据长度, 这样可以跳出循环
                offset = total;
            }
            // 向输出流写入数据
            baos.write(buffer);
        }
    }

    public static void main(String[] args) throws Exception {
        String pubPath = "./public_key";
        String priPath = "./private_key";
        // 初始化密钥对
        generateKeyToFile("RSA", pubPath, priPath);
        // 获取公钥字符串
        PublicKey publicKey = loadPublicKeyFromFile("RSA", pubPath);
        // 获取公钥字符串
        PrivateKey privateKey = loadPrivateKeyFromFile("RSA", priPath);
        // 打印公钥、私钥
        System.out.println("公钥：（填充方式：PKCS1_PADDING，输出类型：base64，字符集：utf8编码）");
        System.out.println("-----BEGIN PUBLIC KEY-----");
        System.out.println(publicKey);
        System.out.println("-----END PUBLIC KEY-----");
        System.out.println("\n");

        System.out.println("私钥：（填充方式：PKCS1_PADDING，输出类型：base64，字符集：utf8编码）");
        System.out.println("-----BEGIN RSA PRIVATE KEY-----");
        System.out.println(privateKey);
        System.out.println("-----END RSA PRIVATE KEY-----");
        System.out.println("\n");

        // 待加密内容，例：123
        String data = "春江潮水连海平，海上明月共潮生。滟滟随波千万里，何处春江无月明。";

        // 进行RSA加密
        String encrypt = encrypt("RSA", data, publicKey, MAX_ENCRYPT_BLOCK);
        System.out.println("密文：（填充方式：PKCS1_PADDING，输出类型：hex，字符集：utf8编码）");
        System.out.println(encrypt);
        System.out.println("\n");

        // 打印解密后内容
        String decrypt = decrypt("RSA", encrypt, privateKey, MAX_DECRYPT_BLOCK);
        System.out.println("解密后明文: ");
        System.out.println(decrypt);
    }
}
```

```
公钥：（填充方式：PKCS1_PADDING，输出类型：base64，字符集：utf8编码）
-----BEGIN PUBLIC KEY-----
Sun RSA public key, 2048 bits
  params: null
  modulus: 16582320951573834141298060888183526147537810640854329357967600944467623520491465765933113089966537050870148571461269892088169382681596677706367557715289768138755961560741478402829183147442099638456077607622597126454069572646367286484816248212505689980483028055628897374844374234844810206729273385291074746664290827689122299768503489398407591571367077834630694095375958347794625096286385307026528172049032243877223712876247878300432451281525621633937249207687769886987562642798593930871029712053252244500490873470545979146528113976831756640683488101335783938772370493064626235166022130646959685998152073110849332761273
  public exponent: 65537
-----END PUBLIC KEY-----


私钥：（填充方式：PKCS1_PADDING，输出类型：base64，字符集：utf8编码）
-----BEGIN RSA PRIVATE KEY-----
SunRsaSign RSA private CRT key, 2048 bits
  params: null
  modulus: 16582320951573834141298060888183526147537810640854329357967600944467623520491465765933113089966537050870148571461269892088169382681596677706367557715289768138755961560741478402829183147442099638456077607622597126454069572646367286484816248212505689980483028055628897374844374234844810206729273385291074746664290827689122299768503489398407591571367077834630694095375958347794625096286385307026528172049032243877223712876247878300432451281525621633937249207687769886987562642798593930871029712053252244500490873470545979146528113976831756640683488101335783938772370493064626235166022130646959685998152073110849332761273
  private exponent: 9731742808481206169684392295669847302845709642927494633049267856723892394298529934072652023533163679304326630079853557372097758467729243296798892302752263029324378496868009556885668282297298263489290888267395985133228922641020775634817012355606662618510737828640289467495679109825000994418745333858207072435296435023326836863519557423579263078608163448409396650632516719864178739917020043825185845896283347928945077229331686048035811206375403205700889949994429330204180080169863224808478183647414243039336977199436997290508446163665173316903214842020220804310853799255558758052184481430534017045529848000691126589441
-----END RSA PRIVATE KEY-----


密文：（填充方式：PKCS1_PADDING，输出类型：hex，字符集：utf8编码）
eChCv+XbPec9FvAv9BuIESVBjcFJqK/2GPnpyKw3DdUY04Qz6Q2UY70C4cT3ig/t1AQ8LFKHLwEnAqCVF6pk7vSHkAf9ygDlkT5BMh4o/tKVkQnG2PUkg772cvvBb7FIZTHMlXY0Ag7H3qeJcBSlnpXnFzQqds/9xsliEQOME+lHvfHMslsjkVhqbwxj8/Mc9iRtVevTZ4GHW34soT5p+Yg2ju3qz3NI4yPoz+FqeL4jaYxAEAWj74NkFzgpirh6+NArpwXP4xQhUlMN9wQQiA+Bu982Eh3KohzHIZiUpEoAsEvHNt0KqOYZJ1DSy4/cr979ywOJseZ3yUvYG2LX/Q==


解密后明文: 
春江潮水连海平，海上明月共潮生。滟滟随波千万里，何处春江无月明。
```



## 9、PBE 加解密【对称加密】

PBE 是一种基于口令的加密算法，使用口令代替其他对称加密算法中的密钥，其特点在于口令由用户自己掌管，不借助任何物理媒体；采用随机数（这里我们叫做盐）杂凑多重加密等方法保证数据的安全性。

PBE 算法是对称加密算法的综合算法，常见算法 PBEWithMD5AndDES，使用MD5和DES算法构建了PBE算法。将盐附加在口令上，通过消息摘要算法经过迭代获得构建密钥的基本材料，构建密钥后使用对称加密算法进行加密解密。

算法/密钥长度/默认密钥长度：

1. PBEWithMD5AndDES/56/56

2. PBEWithMD5AndTripleDES/112,168/168
3. PBEWithSHA1AndDESede/112,168/168
4. PBEWithSHA1AndRC2_40/40 to 1024/128

工作模式：CBC。填充方式：PKCS5Padding。示例代码如下：

```java
import java.security.Key;
import java.security.SecureRandom;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.PBEParameterSpec;

public class PBEUtil {
    public static final String ALGORITHM = "PBEWITHMD5andDES";
    public static final int ITERATION_COUNT = 100;

    public static byte[] initSalt() throws Exception {
        //实例化安全随机数
        SecureRandom random = new SecureRandom();
        return random.generateSeed(8);
    }

    /***
     * 转换密钥
     * @param password 密码
     * @return 密钥
     * @throws Exception
     */
    private static Key toKey(String password) throws Exception {
        // 密钥材料
        PBEKeySpec keySpec = new PBEKeySpec(password.toCharArray());
        // 实例化
        SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
        // 生成密钥
        return factory.generateSecret(keySpec);
    }

    /***
     * 加密
     * @param data 待加密数据
     * @param password 密钥
     * @param salt
     * @return
     * @throws Exception
     */
    public static byte[] encrypt(byte[] data, String password, byte[] salt) throws Exception {
        //转换密钥
        Key key = toKey(password);
        //实例化PBE参数材料
        PBEParameterSpec spec = new PBEParameterSpec(salt, ITERATION_COUNT);
        //实例化
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        //初始化
        cipher.init(Cipher.ENCRYPT_MODE, key, spec);
        return cipher.doFinal(data);
    }


    /***
     * 解密
     * @param data 待解密数据
     * @param password 密钥
     * @param salt
     * @return
     * @throws Exception
     */
    public static byte[] decrypt(byte[] data, String password, byte[] salt) throws Exception {
        //转换密钥
        Key key = toKey(password);
        //实例化PBE参数材料
        PBEParameterSpec spec = new PBEParameterSpec(salt, ITERATION_COUNT);
        //实例化
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        //初始化  
        cipher.init(Cipher.DECRYPT_MODE, key, spec);
        //执行操作  
        return cipher.doFinal(data);
    }


    private static String showByteArray(byte[] data) {
        if (null == data) {
            return null;
        }
        StringBuilder sb = new StringBuilder();
        for (byte b : data) {
            sb.append(b).append(",");
        }
        sb.deleteCharAt(sb.length() - 1);
        sb.append("");
        return sb.toString();
    }


    public static void main(String[] args) throws Exception {
        byte[] salt = initSalt();
        System.out.println("salt：" + showByteArray(salt));
        String password = "1111";
        System.out.println("口令：" + password);
        String data = "PBE数据";
        System.out.println("加密前数据：String:" + data);
        System.out.println("加密前数据：byte[]:" + showByteArray(data.getBytes()));

        byte[] encryptData = encrypt(data.getBytes(), password, salt);
        System.out.println("加密后数据：byte[]:" + showByteArray(encryptData));

        byte[] decryptData = decrypt(encryptData, password, salt);
        System.out.println("解密后数据: byte[]:" + showByteArray(decryptData));
        System.out.println("解密后数据: string:" + new String(decryptData));
    }
}
```

```
salt：21,-109,-57,-101,-118,-48,111,-98
口令：1111
加密前数据：String:PBE数据
加密前数据：byte[]:80,66,69,-26,-107,-80,-26,-115,-82
加密后数据：byte[]:61,68,63,65,99,86,56,-6,46,-85,81,-67,112,121,10,124
解密后数据: byte[]:80,66,69,-26,-107,-80,-26,-115,-82
解密后数据: string:PBE数据
```



## 10、Java 加解密工具类

```java
import com.sun.org.apache.xerces.internal.impl.dv.util.Base64;
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.MessageDigest;
import java.security.SecureRandom;

/**
 *  加密工具
 */
public class EncryptUtil {

    private EncryptUtil() {}

    public static final String MD5 = "MD5";
    public static final String SHA1 = "SHA1";
    public static final String HmacMD5 = "HmacMD5";
    public static final String HmacSHA1 = "HmacSHA1";
    public static final String DES = "DES";
    public static final String AES = "AES";

    /**
     * 编码格式；默认使用uft-8
     */
    public static String charset = "utf-8";
    /**
     * DES
     */
    public static int keysizeDES = 0;
    /**
     * AES
     */
    public static int keysizeAES = 128;


    /**
     * 使用MessageDigest进行单向加密（无密码）
     * @param res 被加密的文本
     * @param algorithm 加密算法名称
     * @return
     */
    private static String messageDigest(String res,String algorithm){
        try {
            MessageDigest md = MessageDigest.getInstance(algorithm);
            byte[] resBytes = charset==null?res.getBytes():res.getBytes(charset);
            return base64(md.digest(resBytes));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 使用KeyGenerator进行单向/双向加密（可设密码）
     * @param res 被加密的原文
     * @param algorithm  加密使用的算法名称
     * @param key 加密使用的秘钥
     * @return
     */
    private static String keyGeneratorMac(String res,String algorithm,String key){
        try {
            SecretKey sk = null;
            if (key==null) {
                KeyGenerator kg = KeyGenerator.getInstance(algorithm);
                sk = kg.generateKey();
            }else {
                byte[] keyBytes = charset==null?key.getBytes():key.getBytes(charset);
                sk = new SecretKeySpec(keyBytes, algorithm);
            }
            Mac mac = Mac.getInstance(algorithm);
            mac.init(sk);
            byte[] result = mac.doFinal(res.getBytes());
            return base64(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 使用KeyGenerator双向加密，DES/AES，注意这里转化为字符串的时候是将2进制转为16进制格式的字符串，不是直接转，因为会出错
     * @param res 加密的原文
     * @param algorithm 加密使用的算法名称
     * @param key  加密的秘钥
     * @param keysize
     * @param isEncode
     * @return
     */
    private static String keyGeneratorES(String res,String algorithm,String key,int keysize,boolean isEncode){
        try {
            KeyGenerator kg = KeyGenerator.getInstance(algorithm);
            if (keysize == 0) {
                byte[] keyBytes = charset==null?key.getBytes():key.getBytes(charset);
                kg.init(new SecureRandom(keyBytes));
            }else if (key==null) {
                kg.init(keysize);
            }else {
                byte[] keyBytes = charset==null?key.getBytes():key.getBytes(charset);
                kg.init(keysize, new SecureRandom(keyBytes));
            }
            SecretKey sk = kg.generateKey();
            SecretKeySpec sks = new SecretKeySpec(sk.getEncoded(), algorithm);
            Cipher cipher = Cipher.getInstance(algorithm);
            if (isEncode) {
                cipher.init(Cipher.ENCRYPT_MODE, sks);
                byte[] resBytes = charset==null?res.getBytes():res.getBytes(charset);
                return parseByte2HexStr(cipher.doFinal(resBytes));
            }else {
                cipher.init(Cipher.DECRYPT_MODE, sks);
                return new String(cipher.doFinal(parseHexStr2Byte(res)));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    private static String base64(byte[] res){
        return Base64.encode(res);
    }

    /**
     * 将二进制转换成16进制
     * @param buf
     * @return
     */
    public static String parseByte2HexStr(byte buf[]) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < buf.length; i++) {
            String hex = Integer.toHexString(buf[i] & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            sb.append(hex.toUpperCase());
        }
        return sb.toString();
    }

    /**
     * 将16进制转换为二进制
     * @param hexStr
     * @return
     */
    public static byte[] parseHexStr2Byte(String hexStr) {
        if (hexStr.length() < 1)
            return null;
        byte[] result = new byte[hexStr.length()/2];
        for (int i = 0;i< hexStr.length()/2; i++) {
            int high = Integer.parseInt(hexStr.substring(i*2, i*2+1), 16);
            int low = Integer.parseInt(hexStr.substring(i*2+1, i*2+2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }

    /**
     * md5加密算法进行加密（不可逆）
     * @param res 需要加密的原文
     * @return
     */
    public static String MD5(String res) {
        return messageDigest(res, MD5);
    }

    /**
     * md5加密算法进行加密（不可逆）
     * @param res  需要加密的原文
     * @param key  秘钥
     * @return
     */
    public static String MD5(String res, String key) {
        return keyGeneratorMac(res, HmacMD5, key);
    }

    /**
     * 使用SHA1加密算法进行加密（不可逆）
     * @param res 需要加密的原文
     * @return
     */
    public static String SHA1(String res) {
        return messageDigest(res, SHA1);
    }

    /**
     * 使用SHA1加密算法进行加密（不可逆）
     * @param res 需要加密的原文
     * @param key 秘钥
     * @return
     */
    public static String SHA1(String res, String key) {
        return keyGeneratorMac(res, HmacSHA1, key);
    }

    /**
     * 使用DES加密算法进行加密（可逆）
     * @param res 需要加密的原文
     * @param key 秘钥
     * @return
     */
    public static String DESencode(String res, String key) {
        return keyGeneratorES(res, DES, key, keysizeDES, true);
    }

    /**
     * 对使用DES加密算法的密文进行解密（可逆）
     * @param res 需要解密的密文
     * @param key 秘钥
     * @return
     */
    public static String DESdecode(String res, String key) {
        return keyGeneratorES(res, DES, key, keysizeDES, false);
    }

    /**
     * 使用AES加密算法经行加密（可逆）
     * @param res 需要加密的密文
     * @param key 秘钥
     * @return
     */
    public static String AESencode(String res, String key) {
        return keyGeneratorES(res, AES, key, keysizeAES, true);
    }

    /**
     * 对使用AES加密算法的密文进行解密
     * @param res 需要解密的密文
     * @param key 秘钥
     * @return
     */
    public static String AESdecode(String res, String key) {
        return keyGeneratorES(res, AES, key, keysizeAES, false);
    }

    /**
     * 使用异或进行加密
     * @param res 需要加密的密文
     * @param key 秘钥
     * @return
     */
    public static String XORencode(String res, String key) {
        byte[] bs = res.getBytes();
        for (int i = 0; i < bs.length; i++) {
            bs[i] = (byte) ((bs[i]) ^ key.hashCode());
        }
        return parseByte2HexStr(bs);
    }

    /**
     * 使用异或进行解密
     * @param res 需要解密的密文
     * @param key 秘钥
     * @return
     */
    public static String XORdecode(String res, String key) {
        byte[] bs = parseHexStr2Byte(res);
        for (int i = 0; i < bs.length; i++) {
            bs[i] = (byte) ((bs[i]) ^ key.hashCode());
        }
        return new String(bs);
    }

    /**
     * 直接使用异或（第一调用加密，第二次调用解密）
     * @param res 密文
     * @param key 秘钥
     * @return
     */
    public static int XOR(int res, String key) {
        return res ^ key.hashCode();
    }

    /**
     * 使用Base64进行加密
     * @param res 密文
     * @return
     */
    public static String Base64Encode(String res) {
        return Base64.encode(res.getBytes());
    }

    /**
     * 使用Base64进行解密
     * @param res
     * @return
     */
    public static String Base64Decode(String res) {
        return new String(Base64.decode(res));
    }

}
```



# SpringBoot 接口加密与解密

## 1、对称与非对称加密

### 1、简介

对称加密只有一个秘钥，加密和解密都是用同一个秘钥，所以叫做对称加密。

非对称加密有两个秘钥，一个是公钥，一个是私钥。非对称的特点在于，公钥加密的私钥可以解密，但私钥加密的，公钥解不出来，只能验证是否由私钥进行加密

目前常见的加密方式是有两种，一种是对称加密(AES为代表），一种是非对称加密（RSA为代表）



### 2、RSA和AES介绍

RSA

- 特点：只需交换公钥；公/秘钥机制，公钥加密，私钥解密(或者私钥加密，公钥解密)；公钥负责加密，私钥负责解密；私钥负责签名，公钥负责验证
- 缺点：加解密速度慢，特别是解密


AES

- 特点：加解密用同一秘钥
- 优点：速度快，效率高；

- 缺点：秘钥交换问题




### 3、RSA/AES组合

对称加密（AES）的优势在于加密较快，但劣势在于秘钥一旦给出去就不安全了。非对称加密（RSA）的优势在于安全，就算提供公钥出去，别人也解密不了数据，但劣势是加密速度较慢

实际使用的过程中常常将两者组合使用（AES+RSA），这样可以安全的传输AES秘钥，避免了RSA加密的慢速度

- 生成一个随机AES秘钥字符串

- 使用RSA公钥加密AES秘钥，然后再用AES秘钥加密真正的内容

- 把skey=加密的AES秘钥，body=AES秘钥加密的内容传过去

- 对面使用RSA私钥解密AES秘钥，然后用AES秘钥解密出内容




### 4、Base64编码的作用

加密后的数据可能不具备可读性，因此我们一般需要对加密后的数据再使用 Base64 算法进行编码，获取可读字符串。换言之，AES 或者RSA加密方法的返回值是一个 Base64 编码之后的字符串，AES或者RSA 解密方法的参数也是一个 Base64 编码之后的字符串，先对该字符串进行解码，然后再解密。




## 2、Java实现加解密/加验签

1、全局Config

```java
public class Config {

    public static final String AES_ALGORITHM = "AES/CBC/PKCS5Padding";
    public static final String RSA_ALGORITHM = "RSA/ECB/OAEPWithSHA-256AndMGF1Padding";
    // 必须是PKCS8格式
    public static final String CLIENT_PRIVATE_KEY = "MIICdgIBADANBgkqhkiG9w0BAQEFAASCAmAwggJcAgEAAoGBAO/8ucCgOTJ7DCPC" +
            "rCCL1VKDnUX61QnxwbAvpGp1/lletEIcjUouM7F0VvMHzViNLvpw7N7NBHPa+5gO" +
            "js68t9hKMUh+a6RTE34SWIqSDRPCzDKVWugsFb04o3vRl3rZ1z6B+QDdW7xwOhEr" +
            "PPoEqmjjIOjQPcU6xs0SPzSimOa1AgMBAAECgYAO5m0OBaSnerZNPhf7yVLMVbmd" +
            "D67MeEMjUkHuDjdlixi8BhPLqESzXtrLKg/Y0KM7D2nVh3sgSldWoIjDUzpCx8Z2" +
            "yHLU1K2wakMdBgEF3xeJPxxZRpP+earl0SyLTA4hMxl48uAjn/mkPgzoMgQkqyQz" +
            "5HOWjjsCLJFyEvqmoQJBAP5cBk0KXpHnCMgOupbi/pXDyaF1o+dCE97GaEdrV/0P" +
            "uwDfYDYfY3wzd1QM7C4b4MmE+SNVpC0W9PyaMONJlN0CQQDxiPiGdwX9actMNJea" +
            "JZ+k3BjCN+mM6Px7j/mtYcXWNZkyCXSXUBI62drZ0htenrh2qwichMlMgNJClvG6" +
            "Gu+5AkEA30R7q2gstrkrNh/nnMZHXcJr3DPc2QNhWayin/4TT+hc51krpJZMxxqN" +
            "5dMqBRcnavwzi9aCs6lxBcF6pCdUaQJANhd7uPls4PzRZ6abkQz9/LjB3rUQ29rN" +
            "uIpc2yR7XuawAVG2x7BJ9N4XMhLoyD75hrH1AsCGKFjtPbZ6OjiQGQJAF2DbIodC" +
            "uYb6eMZ8ux1Ab0wBEWWc5+iGgEVBNh22uZ/klE1/C0+KKzZhqgzaA/vPapq6dhuJ" +
            "sNXlJia10PwYrQ==";

    public static final String CLIENT_PUBLIC_KEY = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDv/LnAoDkyewwjwqwgi9VSg51F" +
            "+tUJ8cGwL6Rqdf5ZXrRCHI1KLjOxdFbzB81YjS76cOzezQRz2vuYDo7OvLfYSjFI" +
            "fmukUxN+EliKkg0TwswylVroLBW9OKN70Zd62dc+gfkA3Vu8cDoRKzz6BKpo4yDo" +
            "0D3FOsbNEj80opjmtQIDAQAB";

    public static final String SERVER_PRIVATE_KEY = "MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAPGkxlAJPKR3BRxT" +
            "PIeB3pDv117j8XbpuEik5UIOlY3GUtAV1sad5NNDUAnP/DB80yAQ8ycm9Xdkutuo" +
            "f25Xlb7w0bRQNpfJlijx9eF8PsB6t63r8KAfWJlqbNHgN8AMK9P5XzVyN4YiEnUl" +
            "Jh/EYiwLiYzflNnmnnfRrI4nUo8fAgMBAAECgYEAvwTxm81heeV4Tcbi33/jUBG4" +
            "4BMzCzyA6DQp4wkiYju3tTS+Xq3seLEKcWdPxYi3YO7lODsM6j/fksrlSXXFMe1i" +
            "ZAF3FNuDVZPz2zdFYS8vh6kdlDHMJAUnU/POMMWJ880MQDtkwTuzH8Tao8OKcAP4" +
            "kc0QuG00wOrmuE+5gZECQQD9bqZkJsN+tj3+pxs57azy6B6gOqgm54/ujB+u63XU" +
            "rO9Sf57asgF4OfUFltaVhjlUMSrWcgp6f4HSy7hBSKJpAkEA9BeML5iDIHOgTIws" +
            "+ID55ELbzO7A/YtcYnUU09mkKCdonMXbXke+EhLApf5vX9ZmreoEfJCdsTnMEcQi" +
            "fkjkRwJBALpf2TXl2/cfhs/zjG45f+rTEVK8UFTsDklb+yDkQC87TnTZLbWfGr2T" +
            "wcFugDhOEXL9BYfXLiWQB6VB9Crug6ECQGEmTiFTbj0oSBCvaeauTsdO5PS3whAn" +
            "u2lkeBmpcfCZXsWm6hyoKTpARHTMw789Mjjd/1Mkq96xxkr76U6h7FkCQHRc2elg" +
            "Dh84wqHIptwa+moosVvd7aSzktuOB4CQRO10qKkSHVFuI+sl47A4KGzH/nX9ydUm" +
            "tpsTnQAlXwBczd4=";

    public static final String SERVER_PUBLIC_KEY = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDxpMZQCTykdwUcUzyHgd6Q79de" +
            "4/F26bhIpOVCDpWNxlLQFdbGneTTQ1AJz/wwfNMgEPMnJvV3ZLrbqH9uV5W+8NG0" +
            "UDaXyZYo8fXhfD7Aeret6/CgH1iZamzR4DfADCvT+V81cjeGIhJ1JSYfxGIsC4mM" +
            "35TZ5p530ayOJ1KPHwIDAQAB";
}
```

2、RSA非对称加密

```java
import javax.crypto.Cipher;
import javax.crypto.spec.OAEPParameterSpec;
import javax.crypto.spec.PSource;
import java.security.*;
import java.security.spec.MGF1ParameterSpec;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import org.springframework.util.Base64Utils;

public class RSACipher {

    /**
     * 获取公钥
     * @param key 密钥字符串（经过base64编码）
     * @return 公钥
     */
    public static PublicKey getPublicKey(String key) throws Exception {
        // 按照X.509标准对其进行编码的密钥
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(Base64Utils.decode(key.getBytes()));
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        // 生成公钥
        PublicKey publicKey = keyFactory.generatePublic(keySpec);
        return publicKey;
    }

    /**
     * 获取私钥
     * @param key 密钥字符串（经过base64编码）
     * @return 私钥
     */
    public static PrivateKey getPrivateKey(String key) throws Exception {
        // 按照PKCS8格式标准对其进行编码的密钥，首先要将key进行base64解码
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(Base64Utils.decode(key.getBytes()));
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        // 生成私钥
        PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
        return privateKey;
    }

    /**
     * 加密方法
     * @param publicKey 公钥
     * @param raw       待加密明文
     * @return 加密后的密文
     */
    public static byte[] encrypt(String publicKey, byte[] raw) throws Exception {
        Key key = getPublicKey(publicKey);
        Cipher cipher = Cipher.getInstance(Config.RSA_ALGORITHM);
        // 初始化
        cipher.init(Cipher.ENCRYPT_MODE, key, new OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA256, PSource.PSpecified.DEFAULT));
        byte[] encryption = cipher.doFinal(raw);
        // 最后将加密后的数据进行base64编码
        return Base64Utils.encode(encryption);
    }

    /**
     * 解密方法
     * @param privateKey 私钥
     * @param enc  待解密密文
     * @return 解密后的明文
     */
    public static byte[] decrypt(String privateKey, byte[] enc) throws Exception {
        Key key = getPrivateKey(privateKey);
        Cipher cipher = Cipher.getInstance(Config.RSA_ALGORITHM);
        // 初始化
        cipher.init(Cipher.DECRYPT_MODE, key, new OAEPParameterSpec("SHA-256", "MGF1", MGF1ParameterSpec.SHA256, PSource.PSpecified.DEFAULT));
        // 先进行base64解密，然后解码
        return cipher.doFinal(Base64Utils.decode(enc));
    }

    /**
     * 签名
     * @param privateKey 私钥
     * @param content    要进行签名的内容
     * @return 签名
     */
    public static String sign(String privateKey, byte[] content) {
        try {
            // privateKey进行base64编码，然后生成PKCS8格式私钥
            PKCS8EncodedKeySpec priPKCS8 = new PKCS8EncodedKeySpec(Base64Utils.decode(privateKey.getBytes()));
            KeyFactory key = KeyFactory.getInstance("RSA");
            PrivateKey priKey = key.generatePrivate(priPKCS8);
            // 签名摘要算法
            Signature signature = Signature.getInstance("SHA256WithRSA");
            // 用私钥初始化此对象以进行签名
            signature.initSign(priKey);
            // 使用指定的字节数组更新签名或验证
            signature.update(content);
            // 获得签名字节
            byte[] signed = signature.sign();
            // 进行base64编码返回
            return new String(Base64Utils.encode(signed));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 验签
     * @param publicKey 公钥
     * @param content   要验签的内容
     * @param sign      签名
     * @return 验签结果
     */
    public static boolean checkSign(String publicKey, byte[] content, String sign) {
        try {
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            // 进行base64解码
            byte[] encodedKey = Base64Utils.decodeFromString(publicKey);
            // 生成公钥
            PublicKey pubKey = keyFactory.generatePublic(new X509EncodedKeySpec(encodedKey));
            // 签名摘要算法
            Signature signature = Signature.getInstance("SHA256WithRSA");
            // 用公钥初始化签名
            signature.initVerify(pubKey);
            // 使用指定的字节数组更新签名或验证
            signature.update(content);
            // base64解码后进行验证
            return signature.verify(Base64Utils.decodeFromString(sign));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    public static void main(String[] args) throws Exception {
        // 客户端代码
        String text = "hello";
        // 使用服务端公钥加密
        byte[] encryptText = RSACipher.encrypt(Config.SERVER_PUBLIC_KEY, text.getBytes());
        System.out.println("加密后:\n" + new String(encryptText));
        // 使用客户端私钥签名
        String signature = RSACipher.sign(Config.CLIENT_PRIVATE_KEY, encryptText);
        System.out.println("签名:\n" + signature);
        // 服务端代码
        // 使用客户端公钥验签
        boolean result = RSACipher.checkSign(Config.CLIENT_PUBLIC_KEY, encryptText, signature);
        System.out.println("验签:\n" + result);
        // 使用服务端私钥解密
        byte[] decryptText = RSACipher.decrypt(Config.SERVER_PRIVATE_KEY, encryptText);
        System.out.println("解密后:\n" + new String(decryptText));
    }
}
```

```
加密后:
ODdEkwo1RgRW8UMoHXPKe9Gwcp6lTCkg4P/Ra3gfkrO+Fw6pSgo0H54nMC5sYSsoUVy1wy2/QXeLSwR6Obfl7SU7DeW+XdGee83O2kgdsDQPbYFwlPYTd0cdOmWwZxtgEOIB9d5G75Iut4kci15vrhXZVtku92U+7aNwtYimSDQ=
签名:
RL1qIScizRyu79/y+r2TN2FL/bSQDxnDj4JlDwSZM6XZR7CL7u5ZjLNHbsSYpHaCv9qKMS4ump50LyF+go05dsPjWZOvFNkgcm9LepkDP1qm8AzKdTGwlzhdBmy2397Ed8uBrQocFGj/721Y2xM/Db0nt7r54zKZkDXbMMlsd9k=
验签:
true
解密后:
hello
```

3、AES对称加密

```java
import org.springframework.util.Base64Utils;
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;

public class AESCipher {

    public static SecureRandom random = new SecureRandom();

    /**
     * 获取随机16位key，key必须要是10的整数倍，否则会出错
    */
    public static String getRandom(int length) {
        StringBuilder ret = new StringBuilder();
        for (int i = 0; i < length; i++) {
            // 输出字母还是数字
            boolean isChar = (random.nextInt(2) % 2 == 0);
            // 字符串
            if (isChar) {
                // 取得大写字母还是小写字母
                int choice = random.nextInt(2) % 2 == 0 ? 65 : 97;
                ret.append((char) (choice + random.nextInt(26)));
            } else { // 数字
                ret.append(random.nextInt(10));
            }
        }
        return ret.toString();
    }

    /**
     * 加密方法，使用key充当向量iv，增加加密算法的强度
     * 更加安全
     * @param key 密钥
     * @param raw 需要加密的内容
     * @return
     */
    public static String encrypt(byte[] key, String raw) throws Exception {
        // 第一次加密
        SecretKeySpec secretKey = new SecretKeySpec(key, "AES");
        byte[] enCodeFormat = secretKey.getEncoded();
        // 获取二次加密的key
        SecretKeySpec secondSecretKey = new SecretKeySpec(enCodeFormat, "AES");
        Cipher cipher = Cipher.getInstance(Config.AES_ALGORITHM);
        // 向量iv，增加加密算法的强度
        IvParameterSpec iv = new IvParameterSpec(key);
        // 初始化加密器
        cipher.init(Cipher.ENCRYPT_MODE, secondSecretKey, iv);
        // 加密
        byte[] result = cipher.doFinal(raw.getBytes());
        // 进行base64编码
        return Base64Utils.encodeToString(result);
    }

    /**
     * 解密方法，使用key充当向量iv，增加加密算法的强度
     * @param key 密钥
     * @param enc 待解密内容
     * @return
     */
    public static String decrypt(byte[] key, String enc) throws Exception {
        SecretKeySpec secretKey = new SecretKeySpec(key, "AES");
        byte[] enCodeFormat = secretKey.getEncoded();
        // 二次加密
        SecretKeySpec secondSecretKey = new SecretKeySpec(enCodeFormat, "AES");

        Cipher cipher = Cipher.getInstance(Config.AES_ALGORITHM);
        IvParameterSpec iv = new IvParameterSpec(key);
        // 初始化
        cipher.init(Cipher.DECRYPT_MODE, secondSecretKey, iv);
        // 首先进行base64解码
        byte[] bytes = Base64Utils.decodeFromString(enc);
        // 解密
        byte[] result = cipher.doFinal(bytes);
        return new String(result);
    }

    public static void main(String[] args) throws Exception {
        // 客户端代码
        String text = "hello";
        // 随机生成16位aes密钥，也可以自己指定16位
        byte[] aesKey = getRandom(16).getBytes();

        String encryptText = AESCipher.encrypt(aesKey, text);
        System.out.println("加密后:\n" + encryptText);
        String decryptText = AESCipher.decrypt(aesKey, encryptText);
        System.out.println("解密后:\n" + decryptText);
    }
}
```

```
加密后:
hwkYAF9eXj/dytmDBD30xg==
解密后:
hello
```



## 3、加解密 starter 实战

> 来源：

### 1、介绍

加密解密本身并不是难事，问题是在何时去处理？定义一个过滤器，将请求和响应分别拦截下来进行处理也是一个办法，这种方式虽然粗暴，但是灵活，因为可以拿到一手的请求参数和响应数据。不过 SpringMVC 中给我们提供了 ResponseBodyAdvice和 RequestBodyAdvice，利用这两个工具可以对请求和响应进行预处理，非常方便。



### 2、前期准备

1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.0</version>
    <scope>provided</scope>
</dependency>
```

scope 几个属性介绍

- compile：默认值 他表示被依赖项目需要参与当前项目的编译，还有后续的测试，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去

- test：依赖项目仅仅参与测试相关的工作，包括测试代码的编译和执行，不会被打包，例如：junit

- runtime：表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过了编译而已。例如JDBC驱动，适用运行和测试阶段

- provided：打包的时候可以不用包进去，别的设施会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是打包阶段做了exclude操作

- system：从参与度来说，和provided相同，不过被依赖项不会从maven仓库下载，而是从本地文件系统拿。需要添加systemPath的属性来定义路径



2、封装公共相应类

```java
public class RespBean {
    private Integer status;
    private String msg;
    private Object obj;

    public static RespBean build() {
        return new RespBean();
    }

    public static RespBean ok(String msg) {
        return new RespBean(200, msg, null);
    }

    public static RespBean ok(String msg, Object obj) {
        return new RespBean(200, msg, obj);
    }

    public static RespBean error(String msg) {
        return new RespBean(500, msg, null);
    }

    public static RespBean error(String msg, Object obj) {
        return new RespBean(500, msg, obj);
    }

    private RespBean() {
    }

    private RespBean(Integer status, String msg, Object obj) {
        this.status = status;
        this.msg = msg;
        this.obj = obj;
    }

    public Integer getStatus() {
        return status;
    }

    public RespBean setStatus(Integer status) {
        this.status = status;
        return this;
    }

    public String getMsg() {
        return msg;
    }

    public RespBean setMsg(String msg) {
        this.msg = msg;
        return this;
    }

    public Object getObj() {
        return obj;
    }

    public RespBean setObj(Object obj) {
        this.obj = obj;
        return this;
    }
}
```



3、定义加解密工具类

加密这块有多种方案可以选择，对称加密、非对称加密，其中对称加密又可以使用 AES、DES、3DES 等不同算法，这里我们使用 Java 自带的 Cipher 来实现对称加密，使用 AES 算法

```java
public class AESUtils {
    private static final String AES_ALGORITHM = "AES/ECB/PKCS5Padding";

    // 获取 cipher
    private static Cipher getCipher(byte[] key, int model) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(key, "AES");
        Cipher cipher = Cipher.getInstance(AES_ALGORITHM);
        cipher.init(model, secretKeySpec);
        return cipher;
    }

    // AES加密
    public static String encrypt(byte[] data, byte[] key) throws Exception {
        Cipher cipher = getCipher(key, Cipher.ENCRYPT_MODE);
        return Base64.getEncoder().encodeToString(cipher.doFinal(data));
    }

    // AES解密
    public static byte[] decrypt(byte[] data, byte[] key) throws Exception {
        Cipher cipher = getCipher(key, Cipher.DECRYPT_MODE);
        return cipher.doFinal(Base64.getDecoder().decode(data));
    }
}
```



4、定义两个自定义注解

接下来我们定义两个注解 `@Decrypt` 和 `@Encrypt`。在以后使用的过程中，哪个接口方法添加了 @Encrypt 注解就对哪个接口的数据加密返回，哪个接口/参数添加了 @Decrypt 注解就对哪个接口/参数进行解密。另外就是 `@Decrypt` 可以用在参数上

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD,ElementType.PARAMETER})
public @interface Decrypt {
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Encrypt {
}
```



5、设置自定义key

定义一个 EncryptProperties 类来读取用户配置的 key，这样就可以自定义key。这里设置了默认值，以后如果用户想自己配置 key，只需要在 application.properties 中配置 `spring.encrypt.key=xxx` 即可。

```java
@ConfigurationProperties(prefix = "spring.encrypt")
@Component
public class EncryptProperties {
    // 这一块一定要16位或者整数倍，最多256
    private final static String DEFAULT_KEY = "www.shawn222.com";
    private String key = DEFAULT_KEY;

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }
}
```



### 3、接口加密与解密

1、介绍

`ResponseBodyAdvice` 在你使用了 `@ResponseBody` 注解的时候才会生效，`RequestBodyAdvice` 在你使用了 `@RequestBody` 注解的时候才会生效，换言之，前后端都是 JSON 交互的时候，这两个才有用

2、接口加密

我们自定义 EncryptResponse 类实现 ResponseBodyAdvice接口，泛型表示接口的返回类型，这里一共要实现两个方法

- supports：这个方法用来判断什么样的接口需要加密，参数 returnType 表示返回类型，我们这里的判断逻辑就是方法是否含有 @Encrypt 注解，如果有，表示该接口需要加密处理，如果没有，表示该接口不需要加密处理。

- beforeBodyWrite：这个方法会在数据响应之前执行，也就是我们先对响应数据进行二次处理，处理完成后，才会转成 json 返回。我们这里的处理方式很简单，RespBean 中的 status 是状态码就不用加密了，另外两个字段重新加密后重新设置值即可。


另外需要注意，自定义的 ResponseBodyAdvice 需要用 @ControllerAdvice 注解来标记。

```java
@EnableConfigurationProperties(EncryptProperties.class)
@ControllerAdvice
public class EncryptResponse implements ResponseBodyAdvice<RespBean> {

    private ObjectMapper om = new ObjectMapper();

    @Autowired
    EncryptProperties encryptProperties;
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
        return returnType.hasMethodAnnotation(Encrypt.class);
    }

    @Override
    public RespBean beforeBodyWrite(RespBean body, MethodParameter returnType, 
                                    MediaType selectedContentType, 
                                    Class<? extends HttpMessageConverter<?>> selectedConverterType, 
                                    ServerHttpRequest request, ServerHttpResponse response) {
        byte[] keyBytes = encryptProperties.getKey().getBytes();
        try {
            if (body.getMsg()!=null) {
                body.setMsg(AESUtils.encrypt(body.getMsg().getBytes(),keyBytes));
            }
            if (body.getObj() != null) {
                body.setObj(AESUtils.encrypt(om.writeValueAsBytes(body.getObj()), keyBytes));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return body;
    }
}
```



3、接口解密

首先大家注意，DecryptRequest 类我们没有直接实现 RequestBodyAdvice 接口，而是继承自 RequestBodyAdviceAdapter 类，该类是 RequestBodyAdvice 接口的子类，并且实现了接口中的一些方法，这样当我们继承自 RequestBodyAdviceAdapter 时，就只需要根据自己实际需求实现某几个方法即可。

- supports：该方法用来判断哪些接口需要处理接口解密，我们这里的判断逻辑是方法上或者参数上含有 @Decrypt 注解的接口，处理解密问题。

- beforeBodyRead：这个方法会在参数转换成具体的对象之前执行，我们先从流中加载到数据，然后对数据进行解密，解密完成后再重新构造 HttpInputMessage 对象返回。

```java
@EnableConfigurationProperties(EncryptProperties.class)
@ControllerAdvice
public class DecryptRequest extends RequestBodyAdviceAdapter {

    @Autowired
    EncryptProperties encryptProperties;
    @Override
    public boolean supports(MethodParameter methodParameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
        return methodParameter.hasMethodAnnotation(Decrypt.class) || methodParameter.hasParameterAnnotation(Decrypt.class);
    }

    @Override
    public HttpInputMessage beforeBodyRead(final HttpInputMessage inputMessage, 
                                           MethodParameter parameter, 
                                           Type targetType, 
                                           Class<? extends HttpMessageConverter<?>> converterType) throws IOException {
        byte[] body = new byte[inputMessage.getBody().available()];
        inputMessage.getBody().read(body);
        try {
            byte[] decrypt = AESUtils.decrypt(body, encryptProperties.getKey().getBytes());
            final ByteArrayInputStream bais = new ByteArrayInputStream(decrypt);
            return new HttpInputMessage() {
                @Override
                public InputStream getBody() throws IOException {
                    return bais;
                }

                @Override
                public HttpHeaders getHeaders() {
                    return inputMessage.getHeaders();
                }
            };
        } catch (Exception e) {
            e.printStackTrace();
        }
        return super.beforeBodyRead(inputMessage, parameter, targetType, converterType);
    }
}
```



### 4、打包发布 starter

1、定义自动化配置类

```java
// 换成自己的包路径
@Configuration
@ComponentScan("com.example.encryption")
public class EncryptAutoConfiguration {
}
```

最后，`resources `目录下定义 `META-INF`，然后再定义 `spring.factories` 文件，这样当项目启动时，就会自动加载该配置类

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.encryption.EncryptAutoConfiguration
```

安装到本地仓库比较简单，直接 `mvn install`，或者在 IDEA 中，点击右边的 Maven，然后双击 install



2、发布线上使用

发不到线上我们可以使用 JitPack （https://jitpack.io/）来做。首先我们在 GitHub 上创建一个仓库，将我们的代码上传上去，上传成功后，点击右边的 Create a new release 按钮，发布一个正式版

发布成功后，打开 jitpack，输入仓库的完整路径，点击 lookup 按钮，查找到之后，再点击 Get it 按钮完成构建，构建成功后，JitPack 上会给出项目引用方式，新建项目时引入即可



### 5、新项目使用

创建实体类

```java
public class User {
    private Long id;
    private String username;
    // 省略 getter/setter
}
```

创建测试类，第一个接口使用了 `@Encrypt` 注解，所以会对该接口的数据进行加密（如果不使用该注解就不加密），第二个接口使用了 `@Decrypt` 所以会对上传的参数进行解密，注意 `@Decrypt` 注解既可以放在方法上也可以放在参数上。

```java
@RestController
public class HelloController {

    @GetMapping("/user")
    @Encrypt
    public RespBean getUser() {
        User user = new User();
        user.setId((long) 99);
        user.setUsername("javaboy");
        return RespBean.ok("ok", user);
    }

    @PostMapping("/user")
    public RespBean addUser(@RequestBody @Decrypt User user) {
        System.out.println("user = " + user);
        return RespBean.ok("ok", user);
    }
}
```



[如何优雅的实现 SpringBoot 接口参数加密解密？](https://mp.weixin.qq.com/s/Upa9UIqB32ssujigewygZA)

[为什么使用 Java Cipher 要指定转换模式？](https://bigzuo.github.io/2019/03/27/java-cipher-tutorial/)

[Hutool加密解密](https://www.hutool.cn/docs/#/crypto/概述)

[【网络】java密码安全](https://blog.csdn.net/hancoder/article/details/111464250)
