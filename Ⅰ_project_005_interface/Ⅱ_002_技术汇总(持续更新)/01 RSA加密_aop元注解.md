##### 01 依赖
aop所需依赖
```xml
<!--springboot aop-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

##### 02 RSA工具类
`package com.consmation.demo.utils;`

此处难点在于RSA加密串不能超过128byte, 因此需要采用分段加密

```java
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;
import javax.crypto.Cipher;
import java.io.ByteArrayOutputStream;
import java.nio.charset.Charset;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.*;


public class RSAUtil {

    public static final String SIGN_ALGORITHMS = "SHA256WithRSA";
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    public static BASE64Encoder base64Encoder = new BASE64Encoder();
    public static BASE64Decoder base64Decoder = new BASE64Decoder();

    /** */
    /**
     * RSA最大加密明文大小
     */
    private static final int MAX_ENCRYPT_BLOCK = 117;

    /** */
    /**
     * RSA最大解密密文大小
     */
    private static final int MAX_DECRYPT_BLOCK = 128;


    /**
     * 创建非对称加密RSA秘钥对
     * @return
     * @throws Exception
     */
    public static KeyPair generateRSAKeyPair(int keyLength) throws Exception {
        KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA");
        generator.initialize(keyLength);
        return generator.generateKeyPair();
    }

    /**
     * 生成公钥STR
     */
    public static String getPublicKeyStr(KeyPair keyPair) throws NoSuchAlgorithmException {
        PublicKey publicKey = keyPair.getPublic();
        byte[] encoded = publicKey.getEncoded();
        return Base64.getEncoder().encodeToString(encoded);
    }

    /**
     * 生成私钥STR
     */
    public static String getPrivateKeyStr(KeyPair keyPair){
        PrivateKey privateKey = keyPair.getPrivate();
        byte[] encoded = privateKey.getEncoded();
        return Base64.getEncoder().encodeToString(encoded);
    }


    /**
     * 签名
     *
     * @param content
     * @param privateKey
     * @return
     */
    public static String sign(String content, String privateKey) {
        try {
            PKCS8EncodedKeySpec priPKCS8 = new PKCS8EncodedKeySpec(Base64.getDecoder().decode(privateKey));
            KeyFactory keyf = KeyFactory.getInstance("RSA");
            PrivateKey priKey = keyf.generatePrivate(priPKCS8);
            Signature signature = Signature.getInstance(SIGN_ALGORITHMS);
            signature.initSign(priKey);
            signature.update(content.getBytes(DEFAULT_CHARSET));
            byte[] signed = signature.sign();
            return Base64.getEncoder().encodeToString(signed);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 验签
     *
     * @param content
     * @param sign
     * @param publicKey
     * @return
     */
    public static boolean verify(String content, String sign, String publicKey) {
        try {
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            byte[] encodedKey = Base64.getDecoder().decode(publicKey);
            PublicKey pubKey = keyFactory.generatePublic(new X509EncodedKeySpec(encodedKey));

            Signature signature = Signature.getInstance(SIGN_ALGORITHMS);

            signature.initVerify(pubKey);
            signature.update(content.getBytes(DEFAULT_CHARSET));

            return signature.verify(Base64.getDecoder().decode(sign));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }


    /**
     * 获取公钥
     * @param key
     * @return
     * @throws Exception
     */
    public static PublicKey getPublicKey(String key) throws Exception {
        byte[] keyBytes = Base64.getDecoder().decode(key);
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PublicKey publicKey = keyFactory.generatePublic(keySpec);
        return publicKey;
    }

    /**
     * 获取私钥
     * @param key
     * @return
     * @throws Exception
     */
    public static PrivateKey getPrivateKey(String key) throws Exception {
        byte[] keyBytes = Base64.getDecoder().decode(key);
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
        return privateKey;
    }


    /**
     * 公钥分段加密
     *
     * @param content
     * @param publicKeyStr
     * @return
     * @throws Exception
     */
    public static String publicEncrpyt(String content, String publicKeyStr) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(1, getPublicKey(publicKeyStr));
        byte[] bytes = content.getBytes(DEFAULT_CHARSET);

        int inputLen = bytes.length;
        int offSet = 0;
        byte[] cache;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int i = 0;
        // 对数据分段加密
        while (inputLen - offSet > 0) {
            if (inputLen - offSet > MAX_ENCRYPT_BLOCK) {
                cache = cipher.doFinal(bytes, offSet, MAX_ENCRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(bytes, offSet, inputLen - offSet);
            }
            out.write(cache, 0, cache.length);
            i++;
            offSet = i * MAX_ENCRYPT_BLOCK;
        }
        byte[] encryptedData = out.toByteArray();
        out.close();
        return Base64.getEncoder().encodeToString(encryptedData);
    }


    /**
     * 私钥分段加密
     *
     * @param content
     * @param privateKeyStr
     * @return
     * @throws Exception
     */
    public static String privateEncrpyt(String content, String privateKeyStr) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(1, getPrivateKey(privateKeyStr));
        byte[] bytes = content.getBytes(DEFAULT_CHARSET);

        int inputLen = bytes.length;
        int offSet = 0;
        byte[] cache;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int i = 0;
        // 对数据分段加密
        while (inputLen - offSet > 0) {
            if (inputLen - offSet > MAX_ENCRYPT_BLOCK) {
                cache = cipher.doFinal(bytes, offSet, MAX_ENCRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(bytes, offSet, inputLen - offSet);
            }
            out.write(cache, 0, cache.length);
            i++;
            offSet = i * MAX_ENCRYPT_BLOCK;
        }
        byte[] encryptedData = out.toByteArray();
        out.close();
        return Base64.getEncoder().encodeToString(encryptedData);
    }


    /**
     * 私钥分段解密
     *
     * @param content
     * @param privateKeyStr
     * @return
     * @throws Exception
     */
    public static String privateDecrypt(String content, String privateKeyStr) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(2, getPrivateKey(privateKeyStr));
        byte[] bytes = Base64.getDecoder().decode(content);
        int inputLen = bytes.length;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int offSet = 0;
        byte[] cache;
        int i = 0;
        // 对数据分段解密
        while (inputLen - offSet > 0) {
            if (inputLen - offSet > MAX_DECRYPT_BLOCK) {
                cache = cipher.doFinal(bytes, offSet, MAX_DECRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(bytes, offSet, inputLen - offSet);
            }
            out.write(cache, 0, cache.length);
            i++;
            offSet = i * MAX_DECRYPT_BLOCK;
        }
        byte[] decryptedData = out.toByteArray();
        out.close();
        return new String(decryptedData);
    }


    /**
     * 公钥分段解密
     *
     * @param content
     * @param publicKeyStr
     * @return
     * @throws Exception
     */
    public static String publicDecrypt(String content, String publicKeyStr) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(2, getPublicKey(publicKeyStr));
        byte[] bytes = Base64.getDecoder().decode(content);
        int inputLen = bytes.length;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        int offSet = 0;
        byte[] cache;
        int i = 0;
        // 对数据分段解密
        while (inputLen - offSet > 0) {
            if (inputLen - offSet > MAX_DECRYPT_BLOCK) {
                cache = cipher.doFinal(bytes, offSet, MAX_DECRYPT_BLOCK);
            } else {
                cache = cipher.doFinal(bytes, offSet, inputLen - offSet);
            }
            out.write(cache, 0, cache.length);
            i++;
            offSet = i * MAX_DECRYPT_BLOCK;
        }
        byte[] decryptedData = out.toByteArray();
        out.close();
        return new String(decryptedData);
    }


}
```

##### 03 获取秘钥对, 存储至Constant
- 测试类
`package com.consmation.demo.test;`
```java
package com.consmation.demo.test;

import com.consmation.demo.InterfaceApplication;
import com.consmation.demo.model.vo.UserVo;
import com.consmation.demo.utils.Constants;
import com.consmation.demo.utils.RSAUtil;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import java.security.KeyPair;

/**
 * @author SaddyFire
 * @date 2022/3/25
 * @TIME:9:48
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = InterfaceApplication.class)
public class RSADTest02 {

    /**
     * 加解密测试
     */
    @Test
    public void test01() throws Exception {
        //生成秘钥对
        KeyPair keyPair = RSAUtil.generateRSAKeyPair(1023);
        String publicKeyStr = RSAUtil.getPublicKeyStr(keyPair);

        //封装对象
        UserVo user01 = UserVo.builder()
                .id(1L)
                .name("张三")
                .age(23).build();
        String userJson = "{\"age\":23,\"id\":1,\"name\":\"张三\"},{\"age\":24,\"id\":2,\"name\":\"李四\"},{\"age\":25,\"id\":3,\"name\":\"王五\"},{\"age\":26,\"id\":4,\"name\":\"赵六\"},{\"age\":27,\"id\":5,\"name\":\"钱七\"},{\"age\":28,\"id\":6,\"name\":\"勾八\"},{\"age\":29,\"id\":7,\"name\":\"西西\"},{\"age\":30,\"id\":8,\"name\":\"东东\"},{\"age\":31,\"id\":9,\"name\":\"楠楠\"},{\"age\":32,\"id\":10,\"name\":\"北北\"}";
        //公钥分段加密
        String publicEncrpyt = RSAUtil.publicEncrpyt(userJson, publicKeyStr);
        System.out.println("公钥分段加密 "+ publicEncrpyt);

        //获取私钥
        String privateKeyStr = RSAUtil.getPrivateKeyStr(keyPair);

        //私钥分段解密
        String privateDecrypt = RSAUtil.privateDecrypt(publicEncrpyt, privateKeyStr);
        System.out.println("私钥分段解密 "+privateDecrypt);
    }

    /**
     * 私钥解密
     */
    @Test
    public void test02(){
        //生成秘钥对
        KeyPair keyPair = null;
        try {
            //生成密钥对
            keyPair = RSAUtil.generateRSAKeyPair(1023);
            //String privateKeyStr = RSAUtil.getPrivateKeyStr(keyPair);
            String publicEncrpyt = "F5f9OMsnr0ZI+SCIwyz5M2ccMdz1g18xOC1/6ram5VHF1nK9JYFNtBHMoXbzQzTTkI+r4sOe3MI+AneNP+ooXm11/3TQZMLgY25rWR7jJ01AaSVYP7Q+cp3nsar4IgWFtfLhygOMyDHjfS0nzl3Besw065D5QXS6d+OdNliPF8wHA3EV9k9ucsNdA8d1m15O3dO3pTgvin7Ol7VlEG/SI7i+ZQu6jZPGEbnAiM+TL7LkRMbo+0dRwb3i2HhmZ6kvHEvWmaik5eE85fSppI1wzIi+5wMi1KZtF4nIAPCRbEWCbhjwaR3QyhqSNM0eb2JXOfZhPlsENd3Gdw0reZRvyg/YyPrzF7tyxaVJvkZeXcIRiFhkjNVF+8321WAiTnbfKW+qG6YH/NmrjESCN3ESZMdg+19YsBtI+iqzvia0fXUaHoFQHCzisOtjPRvt7ZhIPRRHWCbw+7hyu19cmMleUqOL/DIf/WfElhuldmMol0B6FWlxNKsspI1WiNSKBjeXFDwbSi4BASEPlvdBh/r+/pAocQ9/FWeXbiJCBihp96yWVZnWOaBfkeNqGgdpBm3brU++4G2bH7TLaii/3YTlxAe5apByCQsoEPzUOF40eqE+3nRwVSx6+JVmUbRbQe6sN/Nb43BYtFjKR87xDjfxtJLFVpwyqR4aqgA6t2EIZx8=";
            String privateDecrypt = RSAUtil.privateDecrypt(publicEncrpyt, Constants.privateKeyStr);
            System.out.println("私钥分段解密 " + privateDecrypt);


        } catch (Exception e) {
            e.printStackTrace();
        }
    }



    //{"age":23,"id":1,"name":"张三"},{"age":24,"id":2,"name":"李四"},{"age":25,"id":3,"name":"王五"},{"age":26,"id":4,"name":"赵六"},{"age":27,"id":5,"name":"钱七"},{"age":28,"id":6,"name":"勾八"},{"age":29,"id":7,"name":"西西"},{"age":30,"id":8,"name":"东东"},{"age":31,"id":9,"name":"楠楠"},{"age":32,"id":10,"name":"北北"}


    /**
     * 公私钥测试
     */
    @Test
    public void test03() throws Exception {
        //生成秘钥对
        KeyPair keyPair = RSAUtil.generateRSAKeyPair(1023);
        String publicKeyStr = RSAUtil.getPublicKeyStr(keyPair);
        //获取私钥
        String privateKeyStr = RSAUtil.getPrivateKeyStr(keyPair);
        System.out.println("公钥: " + publicKeyStr);
        System.out.println("私钥: " + privateKeyStr);

    }
}
```
`package com.consmation.demo.utils;`
```java
/**
 * @author SaddyFire
 * @date 2022/3/25
 * @TIME:13:49
 */
public class Constants {
    //RSA公钥
    public static final String publicKeyStr = "MIGeMA0GCSqGSIb3DQEBAQUAA4GMADCBiAKBgEAa8GhWjFy7MpgeOC7EKcapzBHjX1zaha8K3lLEsk4jL+qMIqWRDOfRydjukFWgrrmK3cZxTkjK1ui6kG9i0VrR+X/sayaIkZhB3ZXSU+wsLwtFadLrCoojKiYZSvgCdbBnMtbdNXzOWk25BxuEEvvQqoEYBJMUyDgN125oXoQzAgMBAAE=";
    //RSA私钥
    public static final String privateKeyStr = "MIICcwIBADANBgkqhkiG9w0BAQEFAASCAl0wggJZAgEAAoGAQBrwaFaMXLsymB44LsQpxqnMEeNfXNqFrwreUsSyTiMv6owipZEM59HJ2O6QVaCuuYrdxnFOSMrW6LqQb2LRWtH5f+xrJoiRmEHdldJT7CwvC0Vp0usKiiMqJhlK+AJ1sGcy1t01fM5aTbkHG4QS+9CqgRgEkxTIOA3XbmhehDMCAwEAAQKBgCJExGNicOJZh/BdpzcI0jRLLLYbUC04++HY84RXdeHjWYgOpa7QXY/HTBnVXf8ISJ8TJv8gLvMmy7/Zi8CfmL5lLTI61Okfb3s16o7vblQOYdL2/pBdIIBq27vjZX2ikQUjMTETkeSsdH30ezA+5QQmMobqoDXlFecm/+FT/zdxAkEAgQ/7TDN8FjhnTXJelTlLLI913QrsNqTq2hgVtcoyCNHu2Cqa7YUotDY1so0V1Ven05RgxrZpqLKLbj+/C733pQJAfyexJBqVYoRBTEv2yhRkbm4xGm+Je1nJs3ZMbk69pQDHW+xc0CTVIT/qp7kqZKvm/FnMGeTchsle9L4xBX4E9wJABM+es4F7z6w8lZN82R0wozGZ2CqPEZ5mLskVDhjCcre4qpA0BEShds5KhCRkOvawh9+RF/c2yxYUwoBX0806DQJAZKJpVAWmDR5W/6dvXmfdRHj5a86ypGlfdSU/QF4ZQanoHhxnKGS+OV54vN2Ta7GRUk9PdX7n+dUNze1opswh6QJAAMtNQOYBdz/j1OBkp7h6Ib2amMPsHXWs8umbuGvpm+6AWMJw6fm5KfJ3bz0+sOKbJWJwvbk36GUY1Kc+6ZTAHw==";
}

```





