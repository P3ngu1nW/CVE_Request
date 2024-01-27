# xinliangnote/go-gin-api exists insecure signature verification

## 0x01 Affected version

vendor: https://github.com/xinliangnote/go-gin-api

version: 1.2.7

## 0x02 Vulnerability description

The `Verify` function of `signature_verify.go` implements the function of comparing the Hmac signature.

However, the function uses `==` to compare the calculated signature with the signature we provided, and `==` is not a time-safe comparison function, developers should use `equal()` instead.

https://github.com/xinliangnote/go-gin-api/blob/8fd9a005d1953545f74f68cc0e0bc1fa14b332cf/pkg/signature/signature_verify.go#L71

![image-20240127142808364](https://github.com/P3ngu1nW/CVE_Request/blob/main/xinliangnotego-gin-api/image-20240127142808364.png?raw=true)

This means that the more similar the prefix of the signature we provide to the actual calculated signature, the longer the comparison will take, which can lead to temporal side-channel attacks. An attacker can learn the correct signature and bypass verification.