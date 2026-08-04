



|------------------------|------------------------|---------------------------------------------------------------------|
| Certificate Type       | Extension(s)           | Expiry Validation Command                                    	|
|------------------------|------------------------|---------------------------------------------------------------------|
| PEM                    | `.pem`, `.crt`, `.cer` | `openssl x509 -enddate -noout -in cert.pem`                  	|
| DER                    | `.der`                 | `openssl x509 -inform DER -enddate -noout -in cert.der`      	|
| PKCS12                 | `.p12`, `.pfx`         | `openssl pkcs12 -info -in client.p12` 				|
| JKS                    | `.jks`           	  | `keytool -list -v -keystore keystore.jks` 				|
| TrustStore             | `.jks`, `cacerts` 	  | `keytool -list -v -keystore truststore.jks` 			|
| Remote HTTPS Site      | N/A              	  | `openssl s_client -connect host:443 \| openssl x509 -enddate -noout`|
| Kubernetes TLS Secret  | `tls.crt`        	  | `openssl x509 -enddate -noout -in tls.crt` 				|
| CSR (Certificate Signing Request) | `.csr` 	  | No expiry date exists (CSR contains request information only). 	|
|------------------------|------------------------|---------------------------------------------------------------------|


Example Output for following commands

- openssl x509 -enddate -noout -in cert.pem notAfter=Dec 31 23:59:59 2027 GMT

- keytool -list -v -keystore keystore.jks | grep "Valid from"

- openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -enddate -noout


Output:
	notAfter=Jan 15 12:00:00 2028 GMT



Details:

1. PEM Certificate (.pem, .crt, .cer)

Most common server certificate format.

View Full Certificate
```text
openssl x509 -text -noout -in cert.pem
```
Display Expiry Date Only
```text
openssl x509 -enddate -noout -in cert.pem
```

Output:
notAfter=Aug 01 23:59:59 2027 GMT

Check if Already Expired
```text
openssl x509 -checkend 0 -noout -in cert.pem
```

2. DER Certificate (.der, .cer)

Binary format certificate.

Check Expiry
```text
openssl x509 \
-in certificate.der \
-inform DER \
-noout \
-enddate
```
Display Full Details

```text
openssl x509 \
-in certificate.der \
-inform DER \
-text \
-noout
```

3. PKCS12 Certificate (.p12, .pfx)

Commonly used for client certificates in mTLS.

View Expiry Information
```text
openssl pkcs12 \
-in client.p12 \
-info \
-nokeys
```

Extract Certificate
```text
openssl pkcs12 \
-in client.p12 \
-clcerts \
-nokeys \
-out client.pem
```

Then:
```text
openssl x509 \
-noout \
-enddate \
-in client.pem
```


4. Java KeyStore (.jks)

Frequently used in Spring Boot and enterprise Java applications.

List Certificates
```text
keytool -list \
-v \
-keystore keystore.jks
```

Output:

Valid from:
Mon Aug 05 2024

until:
Wed Aug 05 2027

Check Specific Alias
```text
keytool -list \
-v \
-keystore keystore.jks \
-alias client-cert
```

5. TrustStore (.jks, .cacerts)

Contains trusted CA certificates.

View Expiry
```text
keytool -list \
-v \
-keystore truststore.jks
```

Java Default TrustStore

```text
keytool -list \
-v \
-keystore $JAVA_HOME/lib/security/cacerts
```

```text
Default password:
	changeit
```

6. Remote Website Certificate

Check certificate expiry directly from a live server.

Display Expiry Date
```text
openssl s_client \
-connect api.company.com:443 \
</dev/null 2>/dev/null |
openssl x509 -noout -enddate
```

Output:

notAfter=Aug 01 23:59:59 2027 GMT

7. View Complete Server Certificate Chain
```text
openssl s_client \
-connect api.company.com:443 \
-showcerts
```

Useful for:
- Root CA
- Intermediate CA
- Server Certificate

8. Kubernetes TLS Secret Certificate

Extract certificate:  
```text
kubectl get secret tls-secret \
-o jsonpath='{.data.tls\.crt}' |
base64 -d > tls.crt
```

Check expiry:
```text
openssl x509 \
-in tls.crt \
-noout \
-enddate
```


9. CSR (Certificate Signing Request)

CSR itself has no expiry date because it is not a certificate.

View CSR Details
```text
openssl req \
-text \
-noout \
-verify \
-in server.csr
```

Output shows:
- Subject
- Public Key
- SAN Entries

No Not After field exists.

10. Certificate Stored in Kubernetes Ingress
```text
echo | openssl s_client \
-connect mydomain.com:443 \
-servername mydomain.com 2>/dev/null |
openssl x509 -noout -dates
```

Output:

notBefore=Aug 01 2025 GMT
notAfter=Aug 01 2026 GMT

11. Apache / NGINX Server Certificate

Locate certificate:   eg. /etc/ssl/certs/server.crt


Check expiry:

```text
openssl x509 \
-in server.crt \
-noout \
-enddate

```

12. Windows PFX Certificate

View details:

	certutil -dump client.pfx
Or:

```text
openssl pkcs12 \
-in client.pfx \
-info
```

13. Verify Certificate Expires Within N Days (say 30 Days) 
```text
openssl x509 \
-checkend 2592000 \
-noout \
-in cert.pem
```

Where:
2592000 seconds = 30 days


Output:

Certificate will expire	  or
Certificate will not expire



