# Install SSL/TLS Certificate with Full Chain and Secure Permissions

```
sudo cat CERTIFICATE.crt CERTIFICATE-chain.txt > /etc/ssl/certs/CERTIFICATE-fullchain.pem
sudo cp CERTIFICATE.key /etc/ssl/private/CERTIFICATE.key


sudo chmod 600 /etc/ssl/private/CERTIFICATE.key
sudo chmod 644 /etc/ssl/certs/CERTIFICATE-fullchain.pem
```

Why these steps are required: Combining the certificate with the chain ensures browsers have the full trust path to validate your SSL certificate. Copying the key to /etc/ssl/private/ places it in the standard system location. Setting chmod 600 on the private key restricts access to root only, preventing theft or misuse. Setting chmod 644 on the fullchain certificate allows web servers (which run as non-root users) to read the public certificate.
