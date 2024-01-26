# benmcollins/libjwt exists insecure signature verification

## 0x01 Affected version

vendor: https://github.com/benmcollins/libjwt

version: 1.15.3

## 0x02 Vulnerability description

The `jwt_verify_sha_hmac` function of `jwt-wincrypt.c` implements the function of comparing the HS256 algorithm signature.

However, the function uses `strcmp` to compare the calculated signature with the JWT signature we provided, and strcmp is not a time-safe comparison function.

https://github.com/benmcollins/libjwt/blob/323fb1c76f435b2d0d57f992ffa2a6cdc0d9f397/libjwt/jwt-wincrypt.c#L627

![image-20240126142916673.png](https://github.com/P3ngu1nW/CVE_Request/blob/main/benmcollins:libjwt/image-20240126142916673.png?raw=true)

This means that the more similar the prefix of the signature we provide to the actual calculated signature, the longer the comparison will take, which can lead to temporal side-channel attacks. An attacker can learn the correct signature and bypass verification.