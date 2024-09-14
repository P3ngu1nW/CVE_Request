## Summary

The behaviour of `Reference` differs from the common browsers in how it handles a URI that would be considered invalid if fully validated against the RFC. Specifically `Reference` and the browser may differ on the value of the host extracted from an invalid URI and thus a combination of Restlet and a vulnerable browser may be vulnerable to a open redirect attack or to a SSRF attack if the URI is used after passing validation checks.

## Details

### Affected components

The vulnerable component is the `Reference` class when used as a utility class in an application.

### Attack overview

The `Reference` class does not handle extra '/'. When an attacker provides a URI with an extra '/' in the prefix, the URI will not fail to be parsed, but will be parsed as a URI with an empty hostname, which is different from common browsers.

### Attack scenario

A typical attack scenario is illustrated in the diagram below. The Validator checks whether the attacker-supplied URL is on the blocklist. If not, the URI is passed to the Requester for redirection. The Requester is responsible for sending requests to the hostname specified by the URI.

This attack occurs when the Validator is the `org.restlet.data.Reference` class and the Requester is the `Browser` (include chrome, firefox). An attacker can send a malformed URI to the Validator (e.g., `http:////////vulndetector.com/` ). After validation, the Validator finds that the hostname is not on the blocklist. However, the Requester can still send requests to the domain with the hostname `vulndetector.com`.

## PoC

payloads:

```
http://///vulndetector.com
```

### Open Redirect Attack Example

We have listed an open redirection attack scenario here and written an example.

```java
package example.controller;

import org.restlet.data.Reference;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.servlet.http.HttpServletResponse;
import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;


@Controller
@RequestMapping("/oauth")
public class OAuthController {

    private static final Set<String> blackDomains = new HashSet<>(Arrays.asList("github.com"));

    @GetMapping
    public String oauth(@RequestParam(name = "redirect_uri") String redirectUri, HttpServletResponse response) throws Exception {
        Reference reference = new Reference(redirectUri);
        String host = reference.getHostDomain();
        
        System.out.println("Hostname: " + host);

        boolean checker = blackDomains.contains(host);
        
        if (checker) {
            return "redirect:/error";
        }

        return "redirect:" + redirectUri;
    
    }
}
```



## Impact

The impact of this vulnerability is huge because the `restlet` the library is widely used. In many cases, developers need a blocklist to block on some hosts. However, the vulnerability will help attackers bypass the protections that developers have set up for hosts. The vulnerability will lead to **SSRF**\[1\] and **URL Redirection**\[2\] vulnerabilities in several cases.

## Reference

\[1\] [https://cwe.mitre.org/data/definitions/918.html](https://cwe.mitre.org/data/definitions/918.html)  
\[2\] [https://cwe.mitre.org/data/definitions/601.html](https://cwe.mitre.org/data/definitions/601.html)  