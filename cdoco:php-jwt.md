# cdoco/php-jwt exists insecure signature verification

## 0x01 Affected version

vendor: https://github.com/cdoco/php-jwt

version: 1.0.0

## 0x02 Vulnerability description

The `jwt_verify_sha_hmac` function of `openssl.c` implements the function of comparing the HS256 algorithm signature.

However, the function uses `strcmp` to compare the calculated signature with the JWT signature we provided, and `strcmp` is not a time-safe comparison function.

https://github.com/GlitchedPolygons/l8w8jwt/blob/b24083d920c93a2f46f30d3d3d7a2663ac19ca09/src/decode.c#L488    

![image-20240126152828348](https://github.com/P3ngu1nW/CVE_Request/blob/main/cdoco:php-jwt/image-20240126152828348.png?raw=true)

This means that the more similar the prefix of the signature we provide to the actual calculated signature, the longer the comparison will take, which can lead to temporal side-channel attacks. An attacker can learn the correct signature and bypass verification.