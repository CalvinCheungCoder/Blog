一. 简述

MD5: 全称为message digest algorithm 5(信息摘要算法), 可以进行加密, 但是不能解密, 属于单向加密, 通常用于文件校验

Base64: 把任意序列的8为字节描述为一种不易为人识别的形式, 通常用于邮件、http加密. 登陆的用户名和密码字段通过它加密, 可以进行加密和解密.

二. 代码

1.MD5: 

``` Swift
public class MD5Utils {
	/**
	 * 使用md5的算法进行加密
	 * @param plainText 加密明文
	 * @return 加密密文
	 */
	public static String getDigest(String plainText) {
		byte[] secretBytes = null;
		try {
			secretBytes = MessageDigest.getInstance("md5").digest(plainText.getBytes());
		} catch (NoSuchAlgorithmException e) {
			throw new RuntimeException("error happens", e);
		}
		return new BigInteger(1, secretBytes).toString(16);
	}
}
```

2.Base64:  

```Swift
public class Base64Util {
	/**
	 * 使用Base64进行编码
	 * @param encodeContent 需要编码的内容
	 * @return 编码后的内容
	 */
	public static String encode(String encodeContent) { 
		if (encodeContent == null) {
			return null;
		}
		BASE64Encoder encoder = new BASE64Encoder(); 
		return encoder.encode(encodeContent.getBytes());
	}

	/**
	 * 使用Base64进行编码
	 * @param encodeContent 需要编码的内容
	 * @return 编码后的内容
	 */
	public static String encode(byte[] encodeText) { 
		return encode(new String(encodeText));
	}
	
	/**
	 * 使用Base64进行解码
	 * @param encodeContent 需要解码的内容
	 * @return 解码后的内容
	 */
	public static String decode(String decodeContent) { 
		byte[] bytes = null; 
		if (decodeContent == null) {
			return null;
		}
		try {
			bytes = new BASE64Decoder().decodeBuffer(decodeContent);
		} catch (IOException e) {
			throw new RuntimeException("error happens", e);
		} finally {
		}
		return new String(bytes);
	}
}
```

3.测试代码:

```Swift
public class Test {
	/**
	 * 先使用MD5算法加密, 再使用base64算法进行编码
	 * @param args
	 */
	public static void main(String[] args) {
		 String plainText = "pwd";  
		 String encodedPassword = MD5Utils.getDigest(Base64Util.encode(plainText)); 
		 System.out.println(encodedPassword);
	}
}
```

