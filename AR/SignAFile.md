If your certificate/private key is stored in:

Java Keystore (JKS) → Use KeyStore.getInstance("JKS")
Windows Certificate Store → Use KeyStore.getInstance("Windows-MY")

Below are production-style examples for both approaches.


## Option 1: Sign File Using JKS


```java
import java.nio.file.*;
import java.security.*;
import java.util.Base64;

public class FileSignerJKS {

	public static void signFile(String filePath, String jksPath, String jksPassword, String alias, String signatureFile) throws Exception {

		KeyStore keyStore = KeyStore.getInstance("JKS");

		try (java.io.FileInputStream fis = new java.io.FileInputStream(jksPath)) {

			keyStore.load(fis, jksPassword.toCharArray());
		}

		PrivateKey privateKey = (PrivateKey) keyStore.getKey(alias, jksPassword.toCharArray());

		Signature signature = Signature.getInstance("SHA256withRSA");

		signature.initSign(privateKey);

		byte[] fileBytes = Files.readAllBytes(Paths.get(filePath));

		signature.update(fileBytes);

		byte[] signedBytes = signature.sign();

		Files.write(Paths.get(signatureFile), Base64.getEncoder().encode(signedBytes));

		System.out.println("File signed successfully.");
	}

	public static void main(String[] args) throws Exception {

		signFile("sample.txt", "certs\\mykeystore.jks", "changeit", "myalias", "sample.sig");
	}
}
```


**Verify Signature Using JKS Certificate**

```java
import java.nio.file.*;
import java.security.*;
import java.security.cert.Certificate;
import java.util.Base64;

public class FileVerifierJKS {

	public static boolean verify(String filePath, String signatureFile, String jksPath, String jksPassword, String alias) throws Exception {

		KeyStore keyStore = KeyStore.getInstance("JKS");

		keyStore.load(Files.newInputStream(Paths.get(jksPath)), jksPassword.toCharArray());

		Certificate cert = keyStore.getCertificate(alias);

		PublicKey publicKey = cert.getPublicKey();

		Signature signature = Signature.getInstance("SHA256withRSA");

		signature.initVerify(publicKey);

		signature.update(Files.readAllBytes(Paths.get(filePath)));

		byte[] sigBytes = Base64.getDecoder().decode(Files.readAllBytes(Paths.get(signatureFile)));

		return signature.verify(sigBytes);
	}

	public static void main(String[] args) throws Exception {

		System.out.println("Valid Signature : " + verify("sample.txt", "sample.sig", "certs\\mykeystore.jks", "changeit", "myalias"));
	}
}
```



## Option 2: Sign File Using Windows Certificate Store

Java can directly access the Windows Certificate Store (SunMSCAPI Provider).

**List Available Certificates**

```java
import java.security.KeyStore;
import java.security.cert.X509Certificate;
import java.util.Enumeration;

public class ListWindowsCertificates {

	public static void main(String[] args) throws Exception {

		KeyStore ks = KeyStore.getInstance("Windows-MY");

		ks.load(null, null);

		Enumeration<String> aliases = ks.aliases();

		while (aliases.hasMoreElements()) {

			String alias = aliases.nextElement();

			X509Certificate cert = (X509Certificate) ks.getCertificate(alias);

			System.out.println("--------------------------------");
			System.out.println("Alias : " + alias);
			System.out.println("Subject : " + cert.getSubjectDN());
		}
	}
}
```

**Sign File from Windows Certificate Store**

```java
import java.nio.file.*;
import java.security.*;
import java.util.Base64;

public class WindowsCertificateSigner {

	public static void signFile(String alias, String filePath, String signatureFile) throws Exception {

		KeyStore keyStore = KeyStore.getInstance("Windows-MY");

		keyStore.load(null, null);

		PrivateKey privateKey = (PrivateKey) keyStore.getKey(alias, null);

		Signature signature = Signature.getInstance("SHA256withRSA");

		signature.initSign(privateKey);

		byte[] fileBytes = Files.readAllBytes(Paths.get(filePath));

		signature.update(fileBytes);

		byte[] signBytes = signature.sign();

		Files.write(Paths.get(signatureFile), Base64.getEncoder().encode(signBytes));

		System.out.println("File Signed Successfully");
	}

	public static void main(String[] args) throws Exception {

		signFile("my-certificate-alias", "sample.txt", "sample.sig");
	}
}

```

# Verify File Using Windows Certificate Store

```java
import java.nio.file.*;
import java.security.*;
import java.security.cert.Certificate;
import java.util.Base64;

public class WindowsCertificateVerifier {

	public static boolean verify(String alias, String filePath, String signatureFile) throws Exception {

		KeyStore ks = KeyStore.getInstance("Windows-MY");

		ks.load(null, null);

		Certificate cert = ks.getCertificate(alias);

		PublicKey publicKey = cert.getPublicKey();

		Signature signature = Signature.getInstance("SHA256withRSA");

		signature.initVerify(publicKey);

		signature.update(Files.readAllBytes(Paths.get(filePath)));

		byte[] signBytes = Base64.getDecoder().decode(Files.readAllBytes(Paths.get(signatureFile)));

		return signature.verify(signBytes);
	}

	public static void main(String[] args) throws Exception {

		boolean result = verify("my-certificate-alias", "sample.txt", "sample.sig");

		System.out.println("Signature Valid : " + result);
	}
}
```

