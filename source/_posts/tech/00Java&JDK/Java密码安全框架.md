---
title: Java密码
urlname: java_jdk_crypto
date: 2016-04-03
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - 密码
---

# Java密码

参考 Java加解密艺术这本书进行整理如下

## 对称加密 symmetric

- AesCbcUtils
```
package com.security.symmetric;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

/**
 * aes cbc and iv demo.
 * @author fitz
 * @version  1.0.0
 */
public class AesCbcUtils {

    // aes cipher mode
    public static final String CIPHER_ALGORITHM = "AES/CBC/NoPadding";

    // public static final String CIPHER_ALGORITHM = "AES/CBC/PKCS5Padding";

    /**
     * encrypt
     * @param plain 16B
     * @param key 16B
     * @param iv 16B
     * @return byte[] 16B
     * @throws Exception
     */
    public static byte[] encrypt(byte[] plain, byte[] key, byte[] iv) throws Exception {
        if (key == null || key.length != 16 || iv == null || iv.length != 16) {
            throw new Exception("key or iv error");
        }
        SecretKeySpec sKeySpec = new SecretKeySpec(key, "AES");
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);
        cipher.init(Cipher.ENCRYPT_MODE, sKeySpec, ivParameterSpec);
        byte[] encrypted = cipher.doFinal(plain);
        return encrypted;
    }

    /**
     * decrypt
     * @param crypt 16B
     * @param key 16B
     * @param iv 16B
     * @return byte[] 16B
     * @throws Exception
     */
    public static byte[] decrypt(byte[] crypt, byte[] key, byte[] iv) throws Exception {
        if (key == null || key.length != 16 || iv == null || iv.length != 16) {
            throw new Exception("key or iv error");
        }
        SecretKeySpec sKeySpec = new SecretKeySpec(key, "AES");
        sKeySpec.getClass().getSimpleName();
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);
        cipher.init(Cipher.DECRYPT_MODE, sKeySpec, ivParameterSpec);
        byte[] original = cipher.doFinal(crypt);
        return original;
    }

}

```

- AesUtils
```
package com.security.symmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public class AesUtils {
    private static final String KEY_TYPE = "AES";
    private static final String CIPER_MODE = "AES/ECB/PKCS5PADDING";

    private static SecretKey generateSecretKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_TYPE);
        return keyGenerator.generateKey();
    }

    private static byte[] encrypt(SecretKey key, byte[] data) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(key, data, CIPER_MODE);
    }

    private static byte[] decrypt(SecretKey key, byte[] cipherText) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(key, cipherText, CIPER_MODE);
    }

}

```

- CipherAsymmetricUtils

```
package com.security.symmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public abstract class CipherAsymmetricUtils {
    public enum KeyTypeEnum {
        AES("AES"), DES("DES"), DESede("DESede"), IDEA("IDEA");
        private String keyType;
        private KeyTypeEnum(String keyType) {
            this.keyType = keyType;
        }
    }

    public enum WorkTypeEnum {
        ECB("ECB"), CBC("CBC");
        private String workType;
        private WorkTypeEnum(String workType) {
            this.workType = workType;
        }
    }

    public enum PaddingTypeEnum {
        NOPadding("NOPadding"), PKCS5Padding("PKCS5Padding"), PKCS7Padding("PKCS7Padding");
        private String paddingType;
        private PaddingTypeEnum(String paddingType) {
            this.paddingType = paddingType;
        }
    }

    public static byte[] encrypt(Key key, byte[] data, String cipherType) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
        Cipher cipher = Cipher.getInstance(cipherType);
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(Key key, byte[] cipherText, String cipherType) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
        Cipher cipher = Cipher.getInstance(cipherType);
        cipher.init(Cipher.DECRYPT_MODE, key);
        return cipher.doFinal(cipherText);
    }

    public static SecretKey generateSecretKey(String keyType) throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(keyType);
        return keyGenerator.generateKey();
    }
}

```

- Des3Utils

```
package com.security.symmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public class Des3Utils {
    private static final String KEY_TYPE = "DESede";
    private static final String CIPER_MODE = "DESede/ECB/PKCS5PADDING";

    private static SecretKey generateSecretKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_TYPE);
        return keyGenerator.generateKey();
    }

    private static byte[] encrypt(SecretKey key, byte[] data) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(key, data, CIPER_MODE);
    }

    private static byte[] decrypt(SecretKey key, byte[] cipherText) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(key, cipherText, CIPER_MODE);
    }
}

```

- DesUtils
```
package com.security.symmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public class DesUtils {
    private static final String KEY_TYPE = "DES";
    private static final String CIPER_MODE = "DES/ECB/PKCS5PADDING";

    private static SecretKey generateSecretKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_TYPE);
        return keyGenerator.generateKey();
    }

    private static byte[] encrypt(SecretKey key, byte[] data) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(key, data, CIPER_MODE);
    }

    private static byte[] decrypt(SecretKey key, byte[] cipherText) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(key, cipherText, CIPER_MODE);
    }
}

```

- CCM

- IdeaUtils
```
package com.security.symmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public class IdeaUtils {
    private static final String KEY_TYPE = "IDEA";
    private static final String CIPER_MODE = "IDEA/ECB/PKCS5PADDING";

    private static SecretKey generateSecretKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY_TYPE);
        return keyGenerator.generateKey();
    }

    private static byte[] encrypt(SecretKey key, byte[] data) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(key, data, CIPER_MODE);
    }

    private static byte[] decrypt(SecretKey key, byte[] cipherText) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(key, cipherText, CIPER_MODE);
    }
}

```

- PbeUtils

```
package com.security.symmetric;

import javax.crypto.*;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.PBEParameterSpec;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.spec.InvalidKeySpecException;

/**
 * @author fitz
 * @version 1.0
 */
public class PbeUtils {
    /**
     * <pre>
     * PBEWithMD5AndDES
     * PBEWithMD5AndTripleDES
     * PBEWithSHA1AndDESede
     * PBEWithSHA1AndRC2_40
     * </pre>
     */
    private static final String ALGORITHM = "PBEWithMD5AndTripleDES";
    private static final int ITERATION_COUNT = 100;


    public static SecretKey generateSecrectKey(String passwd) throws NoSuchAlgorithmException, InvalidKeySpecException {
        PBEKeySpec pbeKeySpec = new PBEKeySpec(passwd.toCharArray());
        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(ALGORITHM);
        return secretKeyFactory.generateSecret(pbeKeySpec);
    }

    public static byte[] generateSalt() {
        SecureRandom random = new SecureRandom();
        return random.generateSeed(8);
    }

    public static byte[] encrypt(SecretKey key, byte[] data, byte[] salt) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException, InvalidAlgorithmParameterException {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        PBEParameterSpec pbeParameterSpec = new PBEParameterSpec(salt, ITERATION_COUNT);
        cipher.init(Cipher.ENCRYPT_MODE, key, pbeParameterSpec);
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(SecretKey key, byte[] cipherText, byte[] salt) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException, InvalidAlgorithmParameterException {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        PBEParameterSpec pbeParameterSpec = new PBEParameterSpec(salt, ITERATION_COUNT);
        cipher.init(Cipher.DECRYPT_MODE, key, pbeParameterSpec);
        return cipher.doFinal(cipherText);
    }
}

```

- SecretKeyUtils
```
package com.security.symmetric;

import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.SecretKeySpec;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.KeySpec;

/**
 * @author fitz
 * @version 1.0
 */
public class SecretKeyUtils {
    public SecretKey getSecretKey(byte[] keyBytes, String keyType) {
        SecretKey secretKey = new SecretKeySpec(keyBytes, keyType);
        return secretKey;
    }

    public SecretKey generateSecretKey(String keyType, KeySpec keySpec) throws NoSuchAlgorithmException, InvalidKeySpecException {
        SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(keyType);
        return secretKeyFactory.generateSecret(keySpec);
    }
}

```


## 非对称加密

- CipherAsymmetricUtils
```
package com.security.asymmetric;

import javax.crypto.*;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public abstract class CipherAsymmetricUtils {
    public enum KeyTypeEnum {
        RSA("RSA"), ECC("ECC");
        private String keyType;
        private KeyTypeEnum(String keyType) {
            this.keyType = keyType;
        }
    }

    public enum WorkTypeEnum {
        ECB("ECB"), CBC("CBC");
        private String workType;
        private WorkTypeEnum(String workType) {
            this.workType = workType;
        }
    }

    public enum PaddingTypeEnum {
        NOPadding("NoPadding"), PKCS5Padding("PKCS5Padding"), PKCS7Padding("PKCS7Padding");
        private String paddingType;
        private PaddingTypeEnum(String paddingType) {
            this.paddingType = paddingType;
        }
    }

    public static byte[] encrypt(Key key, byte[] data, String cipherType) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
        Cipher cipher = Cipher.getInstance(cipherType);
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return cipher.doFinal(data);
    }

    public static byte[] decrypt(Key key, byte[] cipherText, String cipherType) throws NoSuchPaddingException, NoSuchAlgorithmException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
        Cipher cipher = Cipher.getInstance(cipherType);
        cipher.init(Cipher.DECRYPT_MODE, key);
        return cipher.doFinal(cipherText);
    }

}

```

- DhKeyAgreementUtils
```
package com.security.asymmetric;

import com.security.util.ByteUtils;
import org.bouncycastle.crypto.digests.SHA1Digest;
import org.bouncycastle.crypto.generators.KDF1BytesGenerator;
import org.bouncycastle.crypto.params.ISO18033KDFParameters;

import javax.crypto.KeyAgreement;
import javax.crypto.SecretKey;
import java.math.BigInteger;
import java.security.*;
import java.security.spec.*;

/**
 * @author fitz
 * @version 1.0
 */
public class DhKeyAgreementUtils {

    public static final String KEY_AGREEMENT_ALGORITHM_DH = "DH";
    public static final String KEY_AGREEMENT_ALGORITHM_ECDH = "ECDH";
    public static final String SECRET_KEY_ALGORITHM = "AES";

    public SecretKey generateSecretKey(PublicKey publicKey, PrivateKey privateKey, String keyAgreementType) throws Exception {
        KeyAgreement ka = KeyAgreement.getInstance(keyAgreementType);
        ka.init(privateKey);
        ka.doPhase(publicKey, true);
        SecretKey key = ka.generateSecret(SECRET_KEY_ALGORITHM);
        return key;
    }

    public byte[] generateSessionKeyPki(PublicKey publicKey, PrivateKey privateKey) throws Exception {
        KeyAgreement ka = KeyAgreement.getInstance("ECDH");   //EcDhWithNistKdf256
        ka.init(privateKey);
        ka.doPhase(publicKey, true);
        byte[] secret = ka.generateSecret();
        KDF1BytesGenerator kdf1sha1 = new KDF1BytesGenerator(new SHA1Digest());
        kdf1sha1.init(new ISO18033KDFParameters(secret));
        byte[] key = new byte[16];
        kdf1sha1.generateBytes(key,0,key.length);
        return key;
    }

    public void generateDiffHellmanKeys(String keyType, int keySize) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(keyType);
        keyPairGenerator.initialize(keySize);
        KeyPair keyPair = keyPairGenerator.generateKeyPair();

        // A privateKey & publicKey
        PrivateKey privateKeyA = keyPair.getPrivate();
        PublicKey publicKeyA = keyPair.getPublic();

        // B privateKey & publicKey
        keyPair = keyPairGenerator.generateKeyPair();
        PrivateKey privateKeyB = keyPair.getPrivate();
        PublicKey publicKeyB = keyPair.getPublic();
    }
}

```

- EccCipherUtils
```
package com.security.asymmetric;

import javax.crypto.BadPaddingException;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.interfaces.ECPrivateKey;
import java.security.interfaces.ECPublicKey;

/**
 * @author fitz
 * @version 1.0
 */
public class EccCipherUtils {
    private static final String CIPHER_ALGORITHM = "ECC/ECB/PKCS5Padding";

    public static byte[] encrypt(ECPublicKey publicKey, byte[] data, String cipherType) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(publicKey, data, cipherType);
    }

    public static byte[] decrypt(ECPrivateKey privateKey, byte[] cipherText, String cipherType) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(privateKey, cipherText, cipherType);
    }
}

```

- EccKeyGenerateUtils
```
package com.security.asymmetric;

import org.bouncycastle.util.encoders.Hex;

import java.math.BigInteger;
import java.security.*;
import java.security.spec.ECGenParameterSpec;
import java.security.spec.ECParameterSpec;
import java.security.spec.ECPoint;
import java.security.spec.ECPublicKeySpec;

/**
 * @author fitz
 * @version 1.0
 */
public class EccKeyGenerateUtils {

    /**
     * generate keypair
     * @param keyType
     * @param size
     * @return KeyPair
     *         can use KeyPair's getPublic or getPrivate
     * @throws NoSuchAlgorithmException
     */
    public KeyPair generateKey(String keyType, int size) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(keyType);
        keyPairGenerator.initialize(size);
        return keyPairGenerator.generateKeyPair();
    }

    public PublicKey getPublicKey(String x, String y) throws Exception {
        ECPoint ecPoint = new ECPoint(new BigInteger(Hex.decode(x)), new BigInteger(Hex.decode(y)));
        AlgorithmParameters parameters = AlgorithmParameters.getInstance("EC", "BC");
        parameters.init(new ECGenParameterSpec("secp256r1"));
        ECParameterSpec ecParameters = parameters.getParameterSpec(ECParameterSpec.class);
        return KeyFactory.getInstance ("EC").generatePublic(new ECPublicKeySpec(ecPoint, ecParameters));
    }

}

```

- RsaBlockUtils
```
package com.security.asymmetric;

import javax.crypto.Cipher;
import java.io.ByteArrayOutputStream;
import java.security.Key;

/**
 * no use, just for test block
 * @author fitz
 */
public class RsaBlockUtils {
    public static final String CIPHER_ALGORITHM = "RSA/ECB/OAEPWITHSHA-256ANDMGF1PADDING";


    // for rsa 2048
    private static final int KEY_SIZE = 2048;
    private static final int BLOCK_SIZE = 245;
    private static final int OUTPUT_BLOCK_SIZE = 256;

    // for rsa 1024
    // private static final int KEY_SIZE = 1024;
    // private static final int BLOCK_SIZE = 117;
    // private static final int OUTPUT_BLOCK_SIZE = 128;

    /**
     * encryptByPublicKey
     * @param data
     * @param publicKey
     * @return
     * @throws Exception
     */
    public static byte[] encryptByPublicKey(byte[] data, Key publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        int blocks = data.length / BLOCK_SIZE;
        int lastBlockSize = data.length % BLOCK_SIZE;
        byte[] encryptedData = new byte[(lastBlockSize == 0 ? blocks : blocks + 1) * OUTPUT_BLOCK_SIZE];
        for (int i = 0; i < blocks; i++) {
            cipher.doFinal(data, i * BLOCK_SIZE, BLOCK_SIZE, encryptedData, i * OUTPUT_BLOCK_SIZE);
        }
        if (lastBlockSize != 0) {
            cipher.doFinal(data, blocks * BLOCK_SIZE, lastBlockSize, encryptedData, blocks
                    * OUTPUT_BLOCK_SIZE);
        }
        return encryptedData;

    }

    /**
     * decryptByPrivateKey
     * @param decoded
     * @param privateKey
     * @return
     * @throws Exception
     */
    public static byte[] decryptByPrivateKey(byte[] decoded, Key privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance(CIPHER_ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        int blocks = decoded.length / OUTPUT_BLOCK_SIZE;
        ByteArrayOutputStream decodedStream = new ByteArrayOutputStream(decoded.length);
        for (int i = 0; i < blocks; i++) {
            decodedStream.write(cipher.doFinal(decoded, i * OUTPUT_BLOCK_SIZE, OUTPUT_BLOCK_SIZE));
        }
        return decodedStream.toByteArray();
    }
}

```


- RsaCipherUtils
```
package com.security.asymmetric;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.OAEPParameterSpec;
import javax.crypto.spec.PSource;
import java.security.*;
import java.security.spec.*;

/**
 * @author fitz
 * @version 1.0
 */
public class RsaCipherUtils {
    private static final String CIPHER_ALGORITHM = "RSA/ECB/PKCS5Padding";

    public static byte[] encrypt(PublicKey publicKey, byte[] data, String cipherType) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.encrypt(publicKey, data, cipherType);
    }

    public static byte[] decrypt(PrivateKey privateKey, byte[] cipherText, String cipherType) throws IllegalBlockSizeException, InvalidKeyException, BadPaddingException, NoSuchAlgorithmException, NoSuchPaddingException {
        return CipherAsymmetricUtils.decrypt(privateKey, cipherText, cipherType);
    }

    public static byte[] encryptOaep(byte[] data, PublicKey publicKey) throws NoSuchProviderException, NoSuchAlgorithmException, NoSuchPaddingException, InvalidAlgorithmParameterException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException, InvalidParameterSpecException {
        AlgorithmParameters algp = AlgorithmParameters.getInstance("OAEP", "BC");
        AlgorithmParameterSpec paramSpec = new OAEPParameterSpec("SHA-1", "MGF1", MGF1ParameterSpec.SHA1, PSource.PSpecified.DEFAULT);
        algp.init(paramSpec);
        Cipher oaepFromAlgo = Cipher.getInstance("RSA/ECB/OAEPWITHSHA-1ANDMGF1PADDING");
        oaepFromAlgo.init(Cipher.ENCRYPT_MODE, publicKey, algp);  //algp
        byte[] ct = oaepFromAlgo.doFinal(data);
        return ct;
    }

}

```

- RsaKeyGenerateUtils
```
package com.security.asymmetric;

import java.math.BigInteger;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.*;

/**
 * @author fitz
 * @version 1.0
 */
public class RsaKeyGenerateUtils {

    /**
     * generate keypair
     * @param keyType
     * @param size
     * @return KeyPair
     *         can use KeyPair's getPublic or getPrivate
     * @throws NoSuchAlgorithmException
     */
    public KeyPair generateKey(String keyType, int size) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(keyType);
        keyPairGenerator.initialize(size);
        return keyPairGenerator.generateKeyPair();
    }

    public static Key toPrivateKey(String type, byte[] keyBytes) throws NoSuchAlgorithmException, InvalidKeySpecException {
        PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance(type);
        return keyFactory.generatePrivate(pkcs8EncodedKeySpec);
    }

    public static Key toPublicKey(String type, byte[] keyBytes) throws NoSuchAlgorithmException, InvalidKeySpecException {
        X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance(type);
        return keyFactory.generatePublic(x509EncodedKeySpec);
    }

    /**
     * Generate public key according to modulus and public exponent java Cipher*
     * @param modulus modulus
     * @param exponent public exponent
     * @return
     */
    public static RSAPublicKey getPublicKey(String modulus, String exponent) {
        try {
            BigInteger b1 = new BigInteger(modulus, 16);
            BigInteger b2 = new BigInteger(exponent, 16);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            RSAPublicKeySpec keySpec = new RSAPublicKeySpec(b1, b2);
            return (RSAPublicKey) keyFactory.generatePublic(keySpec);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * Generate private key according to modulus and private exponent java
     * @param modulus modulus
     * @param exponent private exponent
     * @return
     */
    public static RSAPrivateKey getPrivateKey(String modulus, String exponent) {
        try {
            BigInteger b1 = new BigInteger(modulus, 16);
            BigInteger b2 = new BigInteger(exponent, 16);
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            RSAPrivateKeySpec keySpec = new RSAPrivateKeySpec(b1, b2);
            return (RSAPrivateKey) keyFactory.generatePrivate(keySpec);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

```

- EccParams
- RsaParams
- KdfUtils

## 签名
- KeyUtils
```
package com.security.signature;

import java.security.*;

/**
 * @author fitz
 * @version 1.0
 */
public class KeyUtils {
    public static final String ALGORITHM = "DSA";

    /**
     * generate keypair
     * @param keyType
     * @param size
     * @return KeyPair
     *         can use KeyPair's getPublic or getPrivate
     * @throws NoSuchAlgorithmException
     */
    public KeyPair generateKey(String keyType, int size) throws NoSuchAlgorithmException {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(keyType);
        keyPairGenerator.initialize(size);
        return keyPairGenerator.generateKeyPair();
    }
}

```

- EcdsaUtils
```
package com.security.signature;

import java.security.*;

/**
 * @author fitz
 * @version 1.0
 */
public class EcdsaUtils {
    private static final String KEY_ALGORITHM = "ECDSA";
    /**
     * NONEwithECDSA
     * RIPEMD160withECDSA
     * SHA1withECDSA
     * SHA224withECDSA
     * SHA256withECDSA
     * SHA384withECDSA
     * SHA512withECDSA
     */
    private static final String SIGNATURE_ALGORITHM = "SHA512withECDSA";


    public static byte[] sign(PrivateKey privateKey, byte[] data) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return SignatureUtils.sign(privateKey, data, SIGNATURE_ALGORITHM);
    }

    public static boolean verify(PublicKey publicKey, byte[] data, byte[] sign) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return SignatureUtils.verify(publicKey, data, sign, SIGNATURE_ALGORITHM);
    }
}

```

- RsaSignUtils
```
package com.security.signature;

import java.security.*;

/**
 * @author fitz
 * @version 1.0
 */
public class RsaSignUtils {
    public static final String KEY_ALGORITHM = "RSA";
    public static final String SIGNATURE_ALGORITHM = "SHA1withRSA";

    public static byte[] sign(PrivateKey privateKey, byte[] data) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return SignatureUtils.sign(privateKey, data, SIGNATURE_ALGORITHM);
    }

    public static boolean verify(PublicKey publicKey, byte[] data, byte[] sign) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        return SignatureUtils.verify(publicKey, data, sign, SIGNATURE_ALGORITHM);
    }
}

```

- SignatureUtils
```
package com.security.signature;

import java.security.*;

/**
 * @author fitz
 * @version 1.0
 */
public class SignatureUtils {
    public static final String SIGNATURE_ALGORITHM = "SHA1withRSA";

    public static byte[] sign(PrivateKey privateKey, byte[] data, String signatureType) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        Signature signature = Signature.getInstance(signatureType);
        signature.initSign(privateKey);
        signature.update(data);
        return signature.sign();
    }

    public static boolean verify(PublicKey publicKey, byte[] data, byte[] sign, String signatureType) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        Signature signature = Signature.getInstance(signatureType);
        signature.initVerify(publicKey);
        signature.update(data);
        return signature.verify(sign);
    }
}

```

## base64

- Base64BcUtils
```
package com.security.base64;


import org.bouncycastle.util.encoders.Base64;

/**
 * @author fitz
 * @version 1.0
 */
public class Base64BcUtils {

    public static byte[] encode(byte[] bytes) {
        return Base64.encode(bytes);
    }

    public static byte[] decode(byte[] bytes) {
        return Base64.decode(bytes);
    }
}

```
- Base64Utils
```
package com.security.base64;

import java.util.Base64;

/**
 * @author fitz
 * @version 1.0
 */
public class Base64Utils {

    public static byte[] encode(byte[] bytes) {
        return Base64.getEncoder().encode(bytes);
    }

    public static byte[] decode(byte[] bytes) {
        return Base64.getDecoder().decode(bytes);
    }
}

```

- UrlBase64Coder
```
package com.security.base64;

import org.bouncycastle.util.encoders.UrlBase64;

/**
 * @author fitz
 * @version 1.0
 */
public abstract class UrlBase64Coder {

    public final static String ENCODING = "UTF-8";


    public static String encode(String data) throws Exception {

        byte[] b = UrlBase64.encode(data.getBytes(ENCODING));

        return new String(b, ENCODING);
    }

    public static String decode(String data) throws Exception {

        byte[] b = UrlBase64.decode(data.getBytes(ENCODING));

        return new String(b, ENCODING);
    }

}

```

## hash
- crc
- MacStreamUtils
```
package com.security.hash;

import org.apache.commons.codec.digest.DigestUtils;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.security.DigestInputStream;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

/**
 * @author fitz
 * @version 1.0
 */
public class MacStreamUtils {

    public static byte[] digest(FileInputStream fis, String type) throws NoSuchAlgorithmException, IOException {
        DigestInputStream dis = new DigestInputStream(fis, MessageDigest.getInstance(type));

        int bufLen = 1024;
        byte[] buffer = new byte[bufLen];
        while (dis.read(buffer, 0, bufLen) > -1) {
        }
        dis.close();

        MessageDigest md = dis.getMessageDigest();
        // can compare with DigestUtils.md5(fis);
        DigestUtils.md5(fis);
        return md.digest();
    }

    public static byte[] digest1(FileInputStream fis, String type) throws NoSuchAlgorithmException, IOException {
        MessageDigest md = MessageDigest.getInstance(type);
        updateDigest(md, fis);
        return md.digest();
    }

    /**
     * ref: DigestUtils md5
     * @param digest
     * @param data
     * @return
     * @throws IOException
     */
    public static MessageDigest updateDigest(final MessageDigest digest, final InputStream data) throws IOException {
        final int STREAM_BUFFER_LENGTH = 1024;
        final byte[] buffer = new byte[STREAM_BUFFER_LENGTH];
        int read = data.read(buffer, 0, STREAM_BUFFER_LENGTH);

        while (read > -1) {
            digest.update(buffer, 0, read);
            read = data.read(buffer, 0, STREAM_BUFFER_LENGTH);
        }

        return digest;
    }




}

```

- MacUtils
```
package com.security.hash;

import javax.crypto.KeyGenerator;
import javax.crypto.Mac;
import javax.crypto.SecretKey;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * mac utils
 * @author fitz
 * @version 1.0
 */
public class MacUtils {

    private static final String HmacMD2 = "HmacMD2";
    private static final String HmacMD4 = "HmacMD4";
    private static final String HmacSHA224 = "HmacSHA224";
    private static final String HmacSHA256 = "HmacSHA256";
    private static final String HmacRipeMD128 = "HmacRipeMD128";
    private static final String HmacRipeMD160 = "HmacRipeMD160";


    public static SecretKey generateSecretKey(String type) throws NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(type);
        return keyGenerator.generateKey();
    }

    public static byte[] mac(SecretKey key, byte[] bytes) throws NoSuchAlgorithmException, InvalidKeyException {
        Mac mac = Mac.getInstance(key.getAlgorithm());
        mac.init(key);
        return mac.doFinal(bytes);
    }


}

```

- MessageDigestUtils
```
package com.security.hash;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;


/**
 * message digest utils.
 * @author fitz
 * @version 1.0
 */
public class MessageDigestUtils {
    private static final String MD2 ="MD2";
    private static final String MD5 ="MD5";
    private static final String SHA ="SHA";
    private static final String SHA_256 ="SHA-256";
    private static final String SHA_384 ="SHA-384";
    private static final String SHA_512 ="SHA-512";

    public static byte[] md2(byte[] bytes) throws NoSuchAlgorithmException {
        return md(MD2, bytes);
    }

    public static byte[] md5(byte[] bytes) throws NoSuchAlgorithmException {
        return md(MD5, bytes);
    }

    public static byte[] sha1(byte[] bytes) throws NoSuchAlgorithmException {
        return md(SHA, bytes);
    }

    public static byte[] sha256(byte[] bytes) throws NoSuchAlgorithmException {
        return md(SHA_256, bytes);
    }

    public static byte[] sha384(byte[] bytes) throws NoSuchAlgorithmException {
        return md(SHA_384, bytes);
    }

    public static byte[] sha512(byte[] bytes) throws NoSuchAlgorithmException {
        return md(SHA_512, bytes);
    }

    private static byte[] md(String type, byte[] bytes) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance(type);
        return md.digest(bytes);
    }

}

```

- OtherMessageDigestUtils
```
package com.security.hash;

import org.bouncycastle.jce.provider.BouncyCastleProvider;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.Security;

/**
 * @author fitz
 * @version 1.0
 */
public class OtherMessageDigestUtils {
    static {
        if(Security.getProvider("BC") == null ) {
            Security.addProvider(new BouncyCastleProvider());
        }
    }

    private static final String RipeMD128 ="RipeMD128";
    private static final String RipeMD160 ="RipeMD160";
    private static final String RipeMD256 ="RipeMD256";
    private static final String RipeMD320 ="RipeMD320";
    private static final String Tiger ="Tiger";
    private static final String Whirlpool ="Whirlpool";
    private static final String GOST3411 ="GOST3411";

    public static byte[] RipeMD256(byte[] bytes) throws NoSuchAlgorithmException {
        return md(RipeMD256, bytes);
    }

    private static byte[] md(String type, byte[] bytes) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance(type);
        return md.digest(bytes);
    }
}

```

## certificate

- 证书生成 openssl 和 keytool操作
```
echo off

echo 构建目录

mkdir certs
mkdir crl
mkdir newcerts
mkdir private


echo 构建文件
echo 0 > index.txt
echo 01 > serial

echo 构建随机数
openssl.exe rand -out private/.rand 1000

echo 产生私钥
openssl.exe genrsa -out private/ca.key.pem 2048
openssl.exe genrsa -out private/ca1.key.pem 2048
!openssl.exe genrsa -aes256 -out private/ca.key.pem 2048

echo 生成根证书请求 ca.csr
openssl.exe req -new -key private/ca.key.pem -out private/ca.csr -subj "/C=CN/ST=SH/L=PD/O=COM/OU=ANDROID/CN=com.android.security"
openssl.exe req -new -key private/ca1.key.pem -out private/ca1.csr -subj "/C=CN/ST=SH/L=PD/O=COM/OU=ANDROID/CN=com.android.security1"

echo 签发根证书 ca.cer
openssl.exe x509 -req -days 10000 -sha1 -extensions v3_ca -signkey private/ca.key.pem -in private/ca.csr -out certs/ca.cer
openssl.exe x509 -req -days 10000 -sha1 -extensions v3_ca -signkey private/ca1.key.pem -in private/ca1.csr -out certs/ca1.cer

echo 根证书转换 ca.p12
openssl.exe pkcs12 -export -clcerts -in certs/ca.cer -inkey private/ca.key.pem -out certs/ca.p12

echo 颁发服务器证书
openssl.exe genrsa -out private/server.key.pem 2048

echo 生成服务器证书请求 server.csr
openssl.exe req -new -key private/server.key.pem -out private/server.csr -subj "/C=CN/ST=SH/L=PD/O=COM  /OU=ANDROID/CN=com.android.security.server"

echo 签发服务器证书 server.cer
openssl.exe x509 -req -days 3650 -sha1 -extensions v3_req -CA certs/ca.cer -CAkey private/ca.key.pem -CAserial ca.srl -CAcreateserial -in private/server.csr -out certs/server.cer

echo 服务器证书转换 server.p12
openssl.exe pkcs12 -export -clcerts -in certs/server.cer -inkey private/server.key.pem -out certs/server.p12

echo 产生客户私钥
openssl.exe genrsa -out private/client.key.pem 2048

echo 生成客户证书请求 client.csr
openssl.exe req -new -key private/client.key.pem -out private/client.csr -subj "/C=CN/ST=SH/L=PD/O=COM/OU=ANDROID/CN=com.android.security.client"

echo 签发客户证书 client.cer
openssl.exe x509 -req -days 3650 -sha1 -extensions v3_req -CA certs/ca.cer -CAkey private/ca.key.pem -CAserial ca.srl -CAcreateserial -in private/client.csr -out certs/client.cer
openssl.exe x509 -req -days 3650 -sha1 -extensions v3_req -CA certs/ca1.cer -CAkey private/ca1.key.pem -CAserial ca.srl -CAcreateserial -in private/client.csr -out certs/client.cer
!openssl.exe ca -in private/client.csr -days 3650 -out certs/client.cer -cert certs/ca.cer -keyfile private/ca.key.pem -notext

echo 客户证书转换 client.p12
openssl.exe pkcs12 -export -inkey private/client.key.pem -in certs/client.cer -out certs/client.p12

echo 根密钥库转换 ca.keystore
keytool -importkeystore -v -srckeystore certs/ca.p12 -srcstorepass 123456 -destkeystore certs/ca.keystore -srcstoretype pkcs12 -deststorepass 123456
keytool -list -keystore certs/ca.keystore -v -storepass 123456

echo 服务器密钥库转换 server.keystore
keytool -importkeystore -v -srckeystore certs/server.p12 -srcstorepass 123456 -destkeystore certs/server.keystore -srcstoretype pkcs12 -deststorepass 123456
keytool -list -keystore certs/server.keystore -v -storepass 123456

echo 客户密钥库转换 client.keystore
keytool -importkeystore -v -srckeystore certs/client.p12 -srcstorepass 123456 -destkeystore certs/client.keystore -srcstoretype pkcs12 -deststorepass 123456
keytool -list -keystore certs/client.keystore -v -storepass 123456

pause

echo on
```


- UdpSocket
```
package com.security.certificate.udpsocket;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.SocketException;


public class UdpSocket {

    private byte[] buffer = new byte[1024];

    private DatagramSocket receiveSocket;

    private DatagramSocket sendSocket;

    private String remoteHost;

    private int sendPort;

    public UdpSocket(String localHost, String remoteHost, int receivePort,
                     int sendPort) throws SocketException {
        this.remoteHost = remoteHost;
        this.sendPort = sendPort;

        this.receiveSocket = new DatagramSocket(new InetSocketAddress(
                localHost, receivePort));
        this.sendSocket = new DatagramSocket();
    }


    public byte[] receive() throws IOException {
        DatagramPacket dp = new DatagramPacket(buffer, buffer.length);

        receiveSocket.receive(dp);

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(dp.getData(), 0, dp.getLength());
        byte[] data = baos.toByteArray();
        baos.flush();
        baos.close();
        return data;
    }

    public void send(byte[] data) throws IOException {
        DatagramPacket dp = new DatagramPacket(buffer, buffer.length,
                InetAddress.getByName(remoteHost), sendPort);
        dp.setData(data);
        sendSocket.send(dp);
    }


    public void close() {
        try {

            if (receiveSocket.isConnected()) {
                receiveSocket.disconnect();
                receiveSocket.close();
            }

            if (sendSocket.isConnected()) {
                sendSocket.disconnect();
                sendSocket.close();
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}

``` 

- client server
```
package com.security.certificate.clientserver;

import com.security.certificate.CertificateUtils;

import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * @author fitz
 * @version 1.0
 */
public class Client implements Runnable {
    public static final String PROTOCOL = "TLS";
    private static final String PASSWD = "123456";
    private static final String ip = "127.0.0.1";
    private static final int port = Server.PORT;
    private SSLSocketFactory socketFactory;

    public Client() throws Exception {
        // key manager
        KeyStore keyStore = getKeyManagerStore();
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        keyManagerFactory.init(keyStore, PASSWD.toCharArray());

        // trust manager
        KeyStore trustStore = getTrustManagerStore();
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init(trustStore);

        // socket init
        SSLContext sslContext = SSLContext.getInstance(PROTOCOL);
        sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), new SecureRandom());
        socketFactory = sslContext.getSocketFactory();

    }

    private static KeyStore getKeyManagerStore() throws CertificateException, NoSuchAlgorithmException, KeyStoreException, IOException {
        InputStream inputStream = Client.class.getResourceAsStream("/certs/client.p12");
        KeyStore keyStore = CertificateUtils.getKeyStore(inputStream, "123456");
        inputStream.close();
        return keyStore;
    }

    private static KeyStore getTrustManagerStore() throws KeyStoreException, CertificateException, IOException, NoSuchAlgorithmException {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        InputStream inputStream = Server.class.getResourceAsStream("/certs/ca.cer");
        X509Certificate certificate = (X509Certificate) CertificateUtils.getCertificate(inputStream, "X.509");
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", certificate);
        return keyStore;
    }

    @Override
    public void run() {
        int i = 0;
        while(i++ <= 1) {
            try {
                SSLSocket sslSocket = (SSLSocket) socketFactory.createSocket(ip, port);
                InputStream is = sslSocket.getInputStream();
                OutputStream os = sslSocket.getOutputStream();
                DataOutputStream out = new DataOutputStream(os);
                out.writeUTF("GET /index.html HTTP/1.0");
                out.flush();

                DataInputStream bin = new DataInputStream(is);
                String ln;
                if ((ln = bin.readUTF()) != null) {
                    System.out.println(ln);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

```
package com.security.certificate.clientserver;

import com.security.certificate.CertificateUtils;

import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * @author fitz
 * @version 1.0
 */
public class Server implements Runnable {
    private SSLServerSocket serverSocket;
    public static final String PROTOCOL = "TLS";
    private static final String PASSWD = "123456";
    public static final int PORT = 6666;

    public Server() throws Exception {
        // key manager
        KeyStore keyStore = getKeyManagerStore();
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        keyManagerFactory.init(keyStore, PASSWD.toCharArray());

        // trust manager
        KeyStore trustStore = getTrustManagerStore();
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init(trustStore);

        // ssl server init
        SSLContext sslContext = SSLContext.getInstance(PROTOCOL);
        sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), new SecureRandom());
        SSLServerSocketFactory serverSocketFactory = sslContext.getServerSocketFactory();
        serverSocket = (SSLServerSocket) serverSocketFactory.createServerSocket(PORT);
        serverSocket.setNeedClientAuth(true);
    }

    private static KeyStore getKeyManagerStore() throws CertificateException, NoSuchAlgorithmException, KeyStoreException, IOException {
        InputStream inputStream = Server.class.getResourceAsStream("/certs/server.p12");
        KeyStore keyStore = CertificateUtils.getKeyStore(inputStream, "123456");
        inputStream.close();
        return keyStore;
    }

    private static KeyStore getTrustManagerStore() throws KeyStoreException, CertificateException, IOException, NoSuchAlgorithmException {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        InputStream inputStream = Server.class.getResourceAsStream("/certs/ca.cer");
        X509Certificate certificate = (X509Certificate) CertificateUtils.getCertificate(inputStream, "X.509");
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", certificate);
        return keyStore;
    }


    @Override
    public void run() {
        while (true) {
            try {

                SSLSocket socket = (SSLSocket) serverSocket.accept();
                socket.startHandshake();
//                new Thread(new Runnable() {
//                    @Override
//                    public void run() {
//                        try {
//                            //socket.startHandshake();
//
//
//                        } catch (IOException e) {
//                            e.printStackTrace();
//                        }
//                    }
//                });

                OutputStream outputStream = socket.getOutputStream();
                InputStream inputStream = socket.getInputStream();
                DataOutputStream out = new DataOutputStream(outputStream);
                DataInputStream bin = new DataInputStream(inputStream);
                String str = null;
                if((str = bin.readUTF()) != null) {
                    System.out.println(str);
                }

                out.writeUTF("<html><body><h1>hello world</h1></body></html>");
                out.flush();
                out.close();
                bin.close();
                socket.close();
                System.out.println(Thread.currentThread().getName());

                System.out.println("ssl socket...");

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

```

```
package com.security.certificate.clientserver;

/**
 * @author fitz
 * @version 1.0
 */
public class ClientServerMain {
    public static void main(String[] args) throws Exception {
        Thread serverThread = new Thread(new Server());
        serverThread.start();
        new Thread(new Client()).start();
    }
}

```


- CertificateUtils
```
package com.security.certificate;

import com.security.signature.SignatureUtils;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.security.*;
import java.security.cert.Certificate;
import java.security.cert.CertificateException;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;

/**
 * @author fitz
 * @version 1.0
 */
public class CertificateUtils {

    /**
     * p12
     * @param keyStorePath
     * @param passwd
     * @return
     * @throws KeyStoreException
     * @throws IOException
     * @throws CertificateException
     * @throws NoSuchAlgorithmException
     */
    public static KeyStore getKeyStore(String keyStorePath, String passwd) throws KeyStoreException, IOException, CertificateException, NoSuchAlgorithmException {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        FileInputStream fis = new FileInputStream(keyStorePath);
        keyStore.load(fis, passwd.toCharArray());
        fis.close();
        return keyStore;
    }

    public static KeyStore getKeyStore(InputStream fis, String passwd) throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        keyStore.load(fis, passwd.toCharArray());
        return keyStore;
    }

    /**
     * p12
     * @param keyStorePath
     * @param passwd
     * @return
     * @throws KeyStoreException
     * @throws CertificateException
     * @throws NoSuchAlgorithmException
     * @throws IOException
     */
    public static KeyStore createKeyStore(String keyStorePath, String passwd) throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException {
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        keyStore.load(null, passwd.toCharArray());
        FileOutputStream out = new FileOutputStream(keyStorePath);
        keyStore.store(out, passwd.toCharArray());
        out.close();
        return keyStore;
    }

    public static void storeKey(KeyStore keyStore, Key key, String alias, String passwd) throws KeyStoreException {
        keyStore.setKeyEntry(alias, key, passwd.toCharArray(),null);
    }

    public static void storeCertificate(KeyStore keyStore, Certificate certificate, String alias) throws KeyStoreException {
        keyStore.setCertificateEntry(alias, certificate);
    }

    public static PrivateKey getPrivateKey(KeyStore keyStore, String alias, String passwd) throws UnrecoverableKeyException, NoSuchAlgorithmException, KeyStoreException {
        return (PrivateKey) keyStore.getKey(alias, passwd.toCharArray());
    }

    public static Certificate getCertificate(KeyStore keyStore, String alias) throws KeyStoreException {
        return keyStore.getCertificate(alias);
    }

    public static PublicKey getPublicKey(Certificate cert) {
        return cert.getPublicKey();
    }

    /**
     *
     * @param certPath
     * @param certType
     *        can be "X.509"
     * @return
     * @throws CertificateException
     * @throws IOException
     */
    public static Certificate getCertificate(String certPath, String certType) throws CertificateException, IOException {
        CertificateFactory certificateFactory = CertificateFactory.getInstance(certType);
        FileInputStream in = new FileInputStream(certPath);
        Certificate certificate = certificateFactory.generateCertificate(in);
        in.close();
        return certificate;
    }

    public static Certificate getCertificate(InputStream is, String certType) throws CertificateException, IOException {
        CertificateFactory certificateFactory = CertificateFactory.getInstance(certType);
        Certificate certificate = certificateFactory.generateCertificate(is);
        return certificate;
    }


    public static byte[] sign(KeyStore keyStore, String alias, String passwd, byte[] data) throws UnrecoverableKeyException, NoSuchAlgorithmException, KeyStoreException, SignatureException, InvalidKeyException {
        PrivateKey privateKey = getPrivateKey(keyStore, alias, passwd);
        X509Certificate certificate = (X509Certificate) getCertificate(keyStore, alias);
        return SignatureUtils.sign(privateKey, data, certificate.getSigAlgName());
    }

    public static boolean verify(X509Certificate certificate, byte[] data, byte[] sign) throws NoSuchAlgorithmException, InvalidKeyException, SignatureException {
        Signature signature = Signature.getInstance(certificate.getSigAlgName());
        signature.initVerify(certificate);
        return signature.verify(sign);
    }


}

```

- HttpsCertificate

```
package com.security.certificate;

import javax.net.ssl.*;
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.InputStream;
import java.net.URL;
import java.security.KeyStore;
import java.security.SecureRandom;
import java.security.cert.Certificate;
import java.security.cert.CertificateFactory;
import java.security.cert.X509Certificate;

/**
 * @author fitz
 * @version 1.0
 */
public class HttpsCertificate {

    public static final String PROTOCOL = "TLS";


    /**
     * ssl https best practice
     * ref: https://developer.android.com/training/articles/security-ssl.html?hl=zh-cn
     * @throws Exception
     */
    public static void androidHttps() throws Exception {
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        // From https://www.washington.edu/itconnect/security/ca/load-der.crt
        InputStream caInput = new BufferedInputStream(new FileInputStream("load-der.crt"));
        Certificate ca;
        try {
            ca = cf.generateCertificate(caInput);
            System.out.println("ca=" + ((X509Certificate) ca).getSubjectDN());
        } finally {
            caInput.close();
        }

        // Create a KeyStore containing our trusted CAs
        String keyStoreType = KeyStore.getDefaultType();
        KeyStore keyStore = KeyStore.getInstance(keyStoreType);
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", ca);

        // Create a TrustManager that trusts the CAs in our KeyStore
        String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
        TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
        tmf.init(keyStore);

        // Create an SSLContext that uses our TrustManager
        SSLContext context = SSLContext.getInstance("TLS");
        context.init(null, tmf.getTrustManagers(), null);

        // Tell the URLConnection to use a SocketFactory from our SSLContext
        URL url = new URL("https://certs.cac.washington.edu/CAtest/");
        HttpsURLConnection urlConnection = (HttpsURLConnection) url.openConnection();
        urlConnection.setSSLSocketFactory(context.getSocketFactory());
        InputStream in = urlConnection.getInputStream();
    }


    /**
     * First and major difference between trustStore and keyStore is that trustStore is used by TrustManager
     * and keyStore is used by KeyManager class in Java. KeyManager and TrustManager performs different job
     * in Java, TrustManager determines whether remote connection should be trusted or not i.e. whether remote
     * party is who it claims to and KeyManager decides which authentication credentials should be sent to the
     * remote host for authentication during SSL handshake. if you are an SSL Server you will use private key
     * during key exchange algorithm and send certificates corresponding to your public keys to client, this
     * certificate is acquired from keyStore. On SSL client side, if its written in Java, it will use certificates
     * stored in trustStore to verify identity of Server.
     *
     * @param password
     * @param keyStorePath
     * @param trustStorePath
     * @return
     * @throws Exception
     */
    private static SSLSocketFactory getSSLSocketFactory(String password,
                                                        String keyStorePath, String trustStorePath) throws Exception {

        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());

        KeyStore keyStore = CertificateUtils.getKeyStore(keyStorePath, password);

        keyManagerFactory.init(keyStore, password.toCharArray());

        TrustManagerFactory trustManagerFactory = TrustManagerFactory
                .getInstance(TrustManagerFactory.getDefaultAlgorithm());

        KeyStore trustStore = CertificateUtils.getKeyStore(trustStorePath, password);

        trustManagerFactory.init(trustStore);

        SSLContext ctx = SSLContext.getInstance(PROTOCOL);

        ctx.init(keyManagerFactory.getKeyManagers(), trustManagerFactory
                .getTrustManagers(), new SecureRandom());

        return ctx.getSocketFactory();

    }


    public static void configSSLSocketFactory(HttpsURLConnection conn,
                                              String password, String keyStorePath, String trustKeyStorePath) throws Exception {

        // 获得SSLSocketFactory
        SSLSocketFactory sslSocketFactory = getSSLSocketFactory(password,
                keyStorePath, trustKeyStorePath);

        // 设置SSLSocketFactory
        conn.setSSLSocketFactory(sslSocketFactory);
    }
}

```

## asn1


## util
```
package com.security.util;

/**
 * @author fitz
 * @version 1.0
 */
public class ByteUtils {
    public static byte[] stringToBytes(String s) {
        if(s == null || s.length() % 2 != 0) {
            return null;
        }
        int len = s.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(s.charAt(i), 16) << 4)
                    + Character.digit(s.charAt(i+1), 16));
        }
        return data;
    }

    public static String bytesToString(byte[] bytes) {
        final char[] hexChars = {'0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f'};
        char[] chars = new char[bytes.length * 2];
        int byteValue;
        for (int j = 0; j < bytes.length; j++) {
            byteValue = bytes[j] & 0xFF;
            chars[j * 2] = hexChars[byteValue >>> 4];
            chars[j * 2 + 1] = hexChars[byteValue & 0x0F];
        }
        return new String(chars);
    }
}

```

