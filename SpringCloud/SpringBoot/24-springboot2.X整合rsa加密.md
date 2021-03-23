### [springboot2.X整合rsa加密]()

因项目开发需要，前后台通过rsa公钥私钥加解密


后端：
```java
package com.***.abu.ffii.util;

import org.apache.commons.codec.binary.Base64;
import javax.crypto.Cipher;
import java.security.KeyFactory;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;


/**
 * @author linzyc
 * @desc rsa 加/解密工具类
 */
public class RSAUtil {

    /** 算法名称 */
    private static final String ALGORITHM = "RSA";
    /** 编码格式 */
    private static final String CHARSET_NAME = "UTF-8";

    /**
     * 公钥加密
     */
    public static String encryptByPublicKey(String data, String publicKey) throws Exception {
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey = (RSAPublicKey) KeyFactory.getInstance(ALGORITHM).generatePublic(new X509EncodedKeySpec(decoded));
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);
        String outStr = Base64.encodeBase64String(cipher.doFinal(data.getBytes(CHARSET_NAME)));
        return outStr;
    }

    /**
     * 私钥解密
     */
    public static String decryptByPrivateKey(String cipherData, String privateKey) throws Exception {
        byte[] inputByte = Base64.decodeBase64(cipherData.getBytes(CHARSET_NAME));
        byte[] decoded = Base64.decodeBase64(privateKey);
        RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory.getInstance(ALGORITHM).generatePrivate(new PKCS8EncodedKeySpec(decoded));
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, priKey);
        String outStr = new String(cipher.doFinal(inputByte));
        return outStr;
    }

}


```
配置文件配置
```yml
rsa:  #公钥私钥
  publickey: MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCG77PYUAcCpANyUmsHJfuDIia9FcITsuu9lnfbE2BbEwd4SOxPBFTwEWTZ9/e+mtjP97KFEBohGkwy+VHE5KocypBv0O7YgEevwMgpvxyYY0v104CB/k0yjCFV7lc7FxY5VgEKrMiXTIkMr1ukCnWVvapvRCS6IFcsT/kkjPgfDQIDAQAB
  privatekey: MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAIbvs9hQBwKkA3JSawcl+4MiJr0VwhOy672Wd9sTYFsTB3hI7E8EVPARZNn3976a2M/3soUQGiEaTDL5UcTkqhzKkG/Q7tiAR6/AyCm/HJhjS/XTgIH+TTKMIVXuVzsXFjlWAQqsyJdMiQyvW6QKdZW9qm9EJLogVyxP+SSM+B8NAgMBAAECgYEAhj0FH9dNghUE0MCpdS0WL/jTrRxuPQase6mrhyiZnUErF0EExf87OLE1MZr8voRx2UNEOBgyxmfREozyCfyqNg1OdGYEHSyuJ9wglkhq8GVYO8IzI29Mqej0MSprtsE0BPAKBHRU/DWP19ej5bv5ZnAhLs10K7uVEsuGwJJYcMECQQDibedUr7tnGfojyjFY0vCAaVwgS0vXfno7WQyAXUz0Fv8Uy1q9nyF0RrkeA8BOk7S4ljE77ufX0rr2qL7kHW8pAkEAmI718EnQCKKJUjrQUl4iG/lYoNwW2QnxTGZmESyFwkS95PTt8K4GVHpICqRNP1JJBNxVSEVts/eA4zrxPAoBRQJBAJxxEsOQJwq1B/5yVGXqWABgyyYE4AGjgRBAFkMaM3Dx8ouLdMZOi+6qbnwuW0/u/Y4LNzkRd13GWybQsBMrwwECQEULptmavpG55kaWIcS1n+BjSK59DcYrDs+SJK2vJdaXwA4IoEvmpyzCrypJ1EBNYIjXo61y5sSlxuqQua9/o7UCQGYdM3/mF/FEC3wxdfQq0Pw/Pwn8RQxg1natRfoTyzOJDfE/YUYGjIEe2pQtDI1s+IRCwrXOB0cySbpaSHCjr5U=

```

前端vue：
```js
    test() {
      require("jsencrypt");
      let publicKey = ""; //加密公钥
      let msg = "hello world";
      const encrypt = new JSEncrypt();
      encrypt.setPublicKey(publicKey); //	 publicKey为公钥
      const encryptTxt = encrypt.encrypt(msg);
      console.log("加密之后的：", encryptTxt);

      let primaryKey = ""; //解密私钥
      encrypt.setPrivateKey(primaryKey);
      let txt2 = encrypt.decrypt(encryptTxt);
      console.log("解密之后的：", txt2);
    },

// require("jsencrypt");可以配在main.js中
// import jsencrypt from 'jsencrypt';

```

[参考](https://blog.csdn.net/qy20115549/article/details/83105736)
[问题](http://silloy.me/2020/05/04/%E7%94%B1%E4%B8%80%E6%AC%A1Base64%E7%BC%96%E7%A0%81%E5%BC%82%E5%B8%B8%E5%BC%95%E8%B5%B7%E7%9A%84%E5%B0%8Fbug/)
