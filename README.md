# Gommander

A Go library for commanding multiple servers over ssh. This library uses crypto.go/ssh for the basic SSH functionality, but provides the ability to:

- Execute commands across all or subset of servers.
- Copy files to the remote server(s).
- Write files on the remote server(s).

## Usage

The following is an example of managing a cluster of Vagrant instances.

```go
package main

import (
	"golang.org/x/crypto/ssh"
	. "github.com/aerospike/gommander"

	"fmt"
)

func printResponse(responses chan Response, err error) {
	if err != nil {
		panic(err)
	}

	for r := range responses {
		switch r.ExitCode {
		case 0: // Success!
			fmt.Printf("\033[0;97m%s > %s\033[0m\n", r.Node.Host, r.Stdout.String())
		default: // Failure!
			fmt.Printf("\033[0;91m%s > %s\033[0m\n", r.Node.Host, r.Stderr.String())
		}
	}
}

func main() {

	// Parse the ssh key. This is the Vagrant SSH private key.
	vagrantKey, err := Parsekey("/Users/ME/.vagrant.d/insecure_private_key")
	if err != nil {
		panic(err)
	}

	// Array of auth methods, needed for authenticating w/ each node
	authMethods := []ssh.AuthMethod{ssh.PublicKeys(vagrantKey)}

	// The nodes we are using
	nodes := NodeList{
		NewNode("127.0.0.1", 2221, "vagrant", authMethods),
		NewNode("127.0.0.1", 2222, "vagrant", authMethods),
	}

	// Connect to the nodes
	nodes.Connect()

	// Say Hello
	printResponse(nodes.Run("echo hello"))
	
	// Use pipes
	printResponse(nodes.Run("echo \"abc\" | tr \"a-z\" \"A-Z\""))

	// Close the connections to the cluster
	nodes.Close()
}
```

## License

This software is made availabled under the terms of the
Apache License, Version 2, as stated in the file ``LICENSE``.

Individual files may be made available under their own specific license,
all compatible with Apache License, Version 2. Please see individual
files for details.