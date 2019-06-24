# Golang web application up and running securely

Before we write any code we want to generate our keys that we will use. In production you probably want to put the cert in something like /root/certs and keys in something like /etc/ssl/private.

First generate the private key:
```
openssl genrsa -out server.key 2048

openssl ecparam -genkey -name secp384r1 -out server.key
```

Then generate the x509 self-signed public key:
```
  openssl req -new -x509 -sha256 -key server.key -out server.pem -days 3650
```

Now we can create our application in a main.go:
```
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

type Response struct {
	Message    string `json:"message"`
	HTTPStatus int    `json:"HttpStatus"`
}

func main() {
	certPath := "server.pem"
	keyPath := "server.key"

	http.HandleFunc("/hello", hello)

	err := http.ListenAndServeTLS(":3000", certPath, keyPath, nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

func hello(w http.ResponseWriter, req *http.Request) {
	w.WriteHeader(200)
	json.NewEncoder(w).Encode(Response{Message: "Hello world !", HTTPStatus: 200})

}
```


We can now access our application at https://localhost:3000/hello. 

You should get a warning if you are using Chrome, but it’s only because we are using a self-signed certificate.  In production you would want to use a certificate from a legitimate CA.  If you continue through, it should render “hello, world!” in the browser.


