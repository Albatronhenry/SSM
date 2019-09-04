### [springboot2.X整合rsa加密]()

因项目开发需要，前后台通过rsa公钥私钥加解密


后端：
```java
package com.fap.sync.client.common.utils;

import javax.crypto.Cipher;

import org.apache.tomcat.util.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Value;

import java.io.ByteArrayOutputStream;
import java.math.BigInteger;
import java.security.Key;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.RSAPrivateKeySpec;
import java.security.spec.RSAPublicKeySpec;
 
/**
 * rsa简单加/解密工具类  
 * @author 
 *
 */
public class RSAUtil { 
    /**
     * 公钥和私钥直接由支付宝加密客户端生成，存在配置文件中
     */
    /**
     * 公钥前台加密，后台不再处理
     */
  /*  public static byte[] encryptByPublicKey(byte[] data, RSAPublicKey publicKey) throws Exception {
      
    }*/
 
    /**
     * 私钥解密
     * 前台传过来的加密数据 cipherdata 私钥 privateKey
     */
    public static String decryptByPrivateKey(String cipherdata,String pKey) throws Exception {
    	//私钥从配置文件获得
    	 byte[] data = cipherdata.getBytes();
    	 byte[] keyBytes = pKey.getBytes();
    	// 取得私钥
 		PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(Base64.decodeBase64(keyBytes));
 		KeyFactory keyFactory = KeyFactory.getInstance("RSA");
 		Key privateKey = keyFactory.generatePrivate(pkcs8KeySpec);
 		// 对数据解密
 		Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
 		cipher.init(Cipher.DECRYPT_MODE, privateKey);
 		//返回解密结果
    	return new String(cipher.doFinal(Base64.decodeBase64(data))); 
    }
  
}

```
配置文件配置
```yml
rsa:  #公钥私钥
  publickey: MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDUDrwM****JYOGw3BAi+6AhcdBnjo/prNUH/PRA1DUZToAyfHe4F6oSbAY5LvQevgKMWE8t16qNh0++JZbfcZ1WMaTCTyAluLcHHUnNmaE9U9tLF+W9j3G5jea0Eh0tJo579+PcAkWGyy1undwIDAQAB
  privatekey: MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgE****bDcECL7oCFx0GeOj+ms1Qf89EDUNRlOgDJ8d7gXqhJsBjku9B6+AoxYTy3Xqo2HT74llt9xnVYxpMJPICW4twcdSc2ZoT1T20sX5b2PcbmN5rQSHS0mjnv349wCRYbLLW6d3AgMBAAECgYBf9SsFrFoWPJ8iQlZDzw1D3gpvUaUKK55y8sFH2oBdhN3QRX9DFuaANDulN7xdm2wyFYe1X79WYVzMIItQ+StCF6oilION3OI9OcPqm70yoY/k2hpkaJX82J+t/fhEvbiafl10oX4HxWRkEdu5fsr6PzU+z0eMyH5LbQnRHRYdQQJBAOzz7BsTFnSLP0YfQPt65tt28QtQ6casmfB//FhpPNuEd0RG8Xtogqjlk5H3LPYj42UPtAVjrwX8xxnUhojgk0MCQQDlGoL6DfDW+u31X7Ja1v0EdyTeMzGPb0fSLilF7SGmrPs8IneEpV28BswGPNRcIjklsKfhKd7d41UPx8AfnOW9AkEA0G0S3xHgK62cf6LYNxz5Wkx6ZLjMmbyTQBBkOKSBKpqPilhY63OXktc2AiwIuY4B5JB2ilMPzlV2EMt3d4kLHwJBAKfoATwAQZVdPE7L/uwiijbelw+eV2E2/l0k5azQ+Qut1UciP5Pgmkz2ckrUBBMuJdHgoXkc9bCLLsks7Tp+A8UCQGT6o/SWK4bicxaixAADhCxzmnFIz0lH/LDhJfevRxaMpYWvTyBX5Z9jTz1U8n+0Q73rrHKI65tJWtvr7eu9XTY=

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


