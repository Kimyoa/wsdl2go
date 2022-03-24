
### Usage

tl;dr

```
wsdl2go < file.wsdl > hello.go
```

wsdl2go is a code generator that consumes WSDL from stdin (or file, or URL) and produces Go on stdout. The generated code contains services and methods described in the WSDL input, in a single output file. It is your responsibility to make it a package, in the sense that you put it in a directory that makes sense for you, and import it in your code later. Note that the generated code depends on the "soap" package that is part of this project.

WSDL inputs that contain import tags (includes) pointing to other WSDL resources (other files or URLs) may be a source of trouble. The default behavior of wsdl2go is to try and load them, recursively. However, wsdl2go does not support authentication for remote HTTP resources, and cannot fetch resources from HTTPS servers with insecure TLS certificates. In those cases, you have to download the WSDL files yourself using curl or whatever, and process them locally. You might have to tweak their import paths.

Once the code is generated, wsd2go runs gofmt on it. You must have gofmt in your $PATH, or $GOROOT/bin, or you'll get an error.

### Using the generated code

Here's how to use the generated code: let's say you have a WSDL that defines the "example" service. You generate the code and make it the "example" package somewhere in your $GOPATH. This service provides an Echo method that takes an EchoRequest and returns an EchoReply.

The process is this:

- Import the generated code
- Create a soap.Client (and here you can configure SOAP authentication, for example)
- Instantiate the service using your soap.Client
- Call the service methods

Example:

```go
import (
	"/path/to/generated/example"

	"github.com/fiorix/wsdl2go/soap"
)

func main() {
	cli := soap.Client{
		URL: "http://server",
		Namespace: example.Namespace,
	}
	soapService := example.NewEchoService(&cli)
	echoReply, err := soapService.Echo(&example.EchoRequest{Data: "hello world"})
	...
}
```

The soap.Client supports two forms of authentication:

- Setting the "Pre" hook to a function that is run on all outbound HTTP requests, which can set HTTP headers and Basic Auth
- Setting the Header attribute to an AuthHeader, to have it as a SOAP header (with username and password) in every request

Note that only the **Document** style of SOAP is supported. The RPC style is currently not supported.

### Status

Works for my needs, been tested with a few SOAP enterprise systems. Not fully compliant to WSDL or SOAP specs.


Types supported:

- [x] byte
- [x] int
- [x] long (int64)
- [x] float (float64)
- [x] double (float64)
- [x] boolean (bool)
- [x] string
- [x] hexBinary ([]byte)
- [x] base64Binary ([]byte)
- [x] date
- [x] time
- [x] dateTime
- [x] simpleType (w/ enum and validation)
- [x] complexType (struct)
- [x] complexContent (slices, embedded structs)
- [x] token (as string)
- [x] any (slice of empty interfaces)
- [x] anyURI (string)
- [x] QName (string)
- [x] union (empty interface w/ comments)
- [x] nonNegativeInteger (uint)
- [ ] faults
- [ ] decimal
- [ ] g{Day,Month,Year}...
- [ ] NOTATION

Date types are currently defined as strings, need to implement XML Marshaler and Unmarshaler interfaces. The binary ones (hex and base64) are also lacking marshal/unmarshal.

For simple types that have restrictions defined, such as an enumerated list of possible values, we generate the validation function using reflect to compare values. This and the entire API might change anytime, be warned.
