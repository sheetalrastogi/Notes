```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.Arrays;
import java.util.Base64;

public class AESUtil {

	private static SecretKeySpec getKey(String password) throws Exception {

		MessageDigest sha = MessageDigest.getInstance("SHA-256");

		byte[] key = sha.digest(password.getBytes(StandardCharsets.UTF_8));

		// AES-128 requires 16 bytes
		key = Arrays.copyOf(key, 16);

		return new SecretKeySpec(key, "AES");
	}

	public static String encrypt(String plainText, String password) throws Exception {

		SecretKeySpec secretKey = getKey(password);

		Cipher cipher = Cipher.getInstance("AES");

		cipher.init(Cipher.ENCRYPT_MODE, secretKey);

		byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));

		return Base64.getEncoder().encodeToString(encrypted);
	}

	public static String decrypt(String encryptedText, String password) throws Exception {

		SecretKeySpec secretKey = getKey(password);

		Cipher cipher = Cipher.getInstance("AES");

		cipher.init(Cipher.DECRYPT_MODE, secretKey);

		byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedText));

		return new String(decrypted, StandardCharsets.UTF_8);
	}

	public static void main(String[] args) throws Exception {

		String secret = "My Bank Account Number is 123456789";

		String password = "Sheetal@123";

		String encrypted = encrypt(secret, password);

		System.out.println("Encrypted:");
		System.out.println(encrypted);

		String decrypted = decrypt(encrypted, password);

		System.out.println("\nDecrypted:");
		System.out.println(decrypted);
	}
}
```
