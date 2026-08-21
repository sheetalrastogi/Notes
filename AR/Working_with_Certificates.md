# Create a certificate with Powershell

You can create a self-signed certificate containing a public key and private key on Windows using Powershell as:

## Option 1: Using PowerShell (Recommended)
Create a Self-Signed Certificate - Open PowerShell as Administrator and run:

```text
$cert = New-SelfSignedCertificate `
    -Subject "CN=MyTestCertificate" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -HashAlgorithm SHA256 `
    -KeyExportPolicy Exportable
```

This creates:

Public Key
Private Key
Certificate stored in Windows Certificate Store (%APPDATA%\Microsoft\SystemCertificates) as My


# Export Certificate (.cer) - Public Key Only

use following powershell to export public key to "C:\Temp\" location as MyCert.cer

```text
Export-Certificate `
    -Cert $cert `
    -FilePath "C:\Temp\MyCert.cer"
```


## Option 2: Using Java keytool

```text
keytool -genkeypair ^
-alias mycert ^
-keyalg RSA ^
-keysize 2048 ^
-validity 365 ^
-keystore mykeystore.jks
```

Export Public Certificate:

```text
keytool -exportcert ^
-alias mycert ^
-keystore mykeystore.jks ^
-file public.cer
```

**Convert JKS to PFX**

```text
keytool -importkeystore ^
-srckeystore mykeystore.jks ^
-destkeystore mycert.pfx ^
-deststoretype PKCS12
```

