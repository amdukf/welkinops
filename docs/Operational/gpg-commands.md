# GPG commands

```
gpg --full-generate-key

gpg --list-keys

gpg --export -a "amdukf@gmail.com" > public.key

gpg --export-secret-keys -a "amdukf@gmail.com" > private.key
```

For better secutry, we can encrypt private key
```
gpg --symmetric private.key
```
### for importing:
```
gpg --import public.key
gpg --import private.key
gpg --list-secret-keys
```
### for encrypting:
```
gpg --encrypt --recipient "email" file
```
### for decrypting:

```
gpg --decrypt file.gpg
```