lemma
======

Mailgun Cryptographic Tools.

```
Warning!  Still being actively developed and not ready for production use!
```

**Overview**

An keyed-hash message authentication code (HMAC) is used to provide integrity and
authenticity of a message between web services. The following elements are input
into the HMAC. Only the items in bold are required to be passed in by the user, the
other elements are either optional or build by lemma for you.

* **Shared secret**, a randomly generated number from a CSPRNG.
* Timestamp in epoch time (number of seconds since January 1, 1970 UTC).
* Nonce, a randomly generated number from a CSPRNG.
* **Request body.**
* Optionally the HTTP Verb and HTTP Request URI.
* Optionally an additional headers to sign.

Each request element is delimited with the character `|` and each request element is
preceded by it's length. A simple example with only the required parameters:

```
shared_secret = '042DAD12E0BE4625AC0B2C3F7172DBA8'
timestamp     = '1330837567'
nonce         = '000102030405060708090a0b0c0d0e0f'
request_body  = '{"hello": "world"}'

signature     = HMAC('042DAD12E0BE4625AC0B2C3F7172DBA8',
   '10|1330837567|32|000102030405060708090a0b0c0d0e0f|18|{"hello": "world"}')
```

The timestamp, nonce, signature, and signature version are set as headers for the
HTTP request to be signed. They are then verified on receiving side by running the
same algorithm and verifying that the signatures match.

Note: By default the service can securely handle authenticating 5,000 requests per
second. If you need to authenticate more, increase the capacity of the nonce 
cache when initializing the package.

**Examples**


_Signing a Request_

```go
import (
    "net/http"
    "strings"

    "github.com/mailgun/lemma"
)

auths := lemma.New(&lemma.Config{Keypath: "/path/to/file.key"})

[...]

// build new request
requestBody := strings.NewReader(`{"hello":"world"}`)
request, _ := http.NewRequest("POST", "", requestBody)

// sign request
err := auths.SignRequest(request)
if err != nil {
    return err
}

// submit request
client := &http.Client{}
response, _ := client.Do(request)
```

_Signing a Request with Headers_

```go
import (
    "net/http"
    "strings"

    "github.com/mailgun/lemma"
)

auths := lemma.New(&lemma.Config{
    Keypath: "/path/to/file.key",
    HeadersToSign: []string{"X-Mailgun-Header"},
})

[...]

// build new request
requestBody := strings.NewReader(`{"hello":"world"}`)
request, _ := http.NewRequest("POST", "", requestBody)
request.Header.Set("X-Mailgun-Header", "foobar")

// sign request
err := auths.SignRequest(request)
if err != nil {
    return err
}

// submit request
client := &http.Client{}
response, _ := client.Do(request)
```

_Signing a Request with HTTP Verb and URI_

```go
import (
    "net/http"
    "strings"

    "github.com/mailgun/lemma"
)

auths := lemma.New(&lemma.Config{
    Keypath: "/path/to/file.key",
    SignVerbAndURI: true,
})

[...]

// build new request
requestBody := strings.NewReader(`{"hello":"world"}`)
request, _ := http.NewRequest("POST", "", requestBody)

// sign request
err := auths.SignRequest(request)
if err != nil {
    return err
}

// submit request
client := &http.Client{}
response, _ := client.Do(request)
```

_Authenticating a Request_

```go
import (
    "fmt"
    "net/http"
    "strings"

    "github.com/mailgun/lemma"
)

auths := lemma.New(&lemma.Config{Keypath: "/path/to/file.key"})

[...]

func handler(w http.ResponseWriter, r *http.Request) {
    // authenticate request
    isValid, err := auths.AuthenticateRequest(r)
   
    // request is invalid
    if !isValid {
        fmt.Fprintf(w, "<p>Unable to Authenticate Request: %v</p>", err)
        return
    }
  
    // valid request
    fmt.Fprintf(w, "<p>Request Authenticated, welcome!</p>")
}
```

_Generate a n-bit random number_

```go
import (
    "fmt"

    "github.com/mailgun/lemma"
)

outputSize = 16 // 128 bit = 16 byte

csprng := lemma.CSPRNG{}
hexDigest := csprng.HexDigest(outputSize)
fmt.Printf("%v\n", hexDigest)
```

Will print out something like the following to the console:

```
45eb93fdf5afe149ee3e61412c97e9bc
```

Raw bytes are also avaiable using the `Bytes()` method.
