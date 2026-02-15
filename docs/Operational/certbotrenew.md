# Renewing certbot certificate

See certificates status:
```
certbot certificates
```

Renew a certificate:
```
certbot renew --cert-name vikunja.amirkheirandish.ir
```

Delete a certificate
```
certbot delete --cert-name test.amirkheirandish.ir
```

Renew all certificates:
```
certbot renew
```