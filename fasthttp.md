## Summary

Vendor: fasthttp

Version: 1.56.0

When fasthttp parses the URL, it does not consider #, resulting in incorrect parsing of the host

## Detailed

According to RFC3986,

```
   The fragment identifier component of a URI allows indirect
   identification of a secondary resource by reference to a primary
   resource and additional identifying information.  The identified
   secondary resource may be some portion or subset of the primary
   resource, some view on representations of the primary resource, or
   some other resource defined or described by those representations.  A
   fragment identifier component is indicated by the presence of a
   number sign ("#") character and terminated by the end of the URI.
```

However, when fasthttp parses the URL, it does not take into account the number sign. This may lead to Open Redirect and SSRF.

## Poc

```go
package main

import (
	"fmt"
	"github.com/valyala/fasthttp"
)

func main() {
	u := fasthttp.AcquireURI()
	host := ""
	uri := "http://google.com#@github.com"
	err := u.Parse([]byte(host), []byte(uri))
	if err != nil {
		return
	}
	fmt.Println(string(u.Host()))
}
```

According to RFC3986, this should be parsed to google.com