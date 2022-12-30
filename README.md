# acme-restful

This project can be used to quickly provide a restful-api. It does not support authentication or authorisation, so it should not be used in a production environment.
Currently, only the file system is supported as a backend. This means that all data are stored as JSON files on the hard disk.

## Usage

The first argument must be an existing and writable folder. This is where the JSON files are stored.
```
go run cmd/server/main.go _tmp/
```

## Magic URLs

You can configure two optional "magic" URLs:
* __all.json
* __list.json

### __all.json

This URL returns all data as a JSON object. The keys are the file names without the extension. The values are the content of the files.

### __list.json

This URL returns all file names as a JSON array.

### Usage

```golang
package main

import (
	"github.com/skroczek/acme-restful/pkg/backend"
	"github.com/skroczek/acme-restful/pkg/backend/fs"
	"github.com/skroczek/acme-restful/pkg/router"
	"github.com/skroczek/acme-restful/pkg/server"
)

func main() {
	be := fs.NewMemory()
	s := server.NewServer(
		server.WithBackend(be),
		// You can additional add the encrypted backend to encrypt the data as rest. But you have to set the passphrase
		// in the environment variable ACME_RESTFUL_PASSPHRASE
		//pkg.WithBackend(backend.NewEncrypted(be)),
		server.WithRouterOptions(
			router.WithDefaultCors(true),
		),
		server.WithListAll(),
		server.WithGetAll(),
	)
	// By default, it serves on :8080 unless a
	// PORT environment variable was defined.
	s.Run()
}
```

## Example

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"John Doe"}' http://localhost:8080/users/1.json
{"name":"John Doe"}
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"Jane Doe"}' http://localhost:8080/users/2.json
{"name":"Jane Doe"}
$ curl -X POST -H "Content-Type: application/json" -d '{"name":"John Doe","age":42}' http://localhost:8080/users/1.json
{"name":"John Doe","age":42}
$ curl http://localhost:8080/users/1.json
{"name":"John Doe","age":42}
$ curl http://localhost:8080/users/2.json
{"name":"Jane Doe"}
$ curl http://localhost:8080/users/__all.json
{"1":{"id":"users/1","name":"John Doe","age":42},"2":{"id":"users/2","name":"Jane Doe"}}
$ curl http://localhost:8080/users/__list.json
["1","2"]
```

## Backends

### File System
Currently, only the file system is supported as a backend. This means that all data are stored as JSON files on the hard disk.

## Encryption at rest

It is possible to save the data encrypted through the Encrypted Backend. The encrypted backend acts as a proxy before 
the actual backend. The passphrase in the environment variable ACME_RESTFUL_PASSPHRASE is used for encryption.

### Usage

```golang
package main

import (
	"log"
	"os"
	"path/filepath"

	"github.com/skroczek/acme-restful/pkg/backend"
	"github.com/skroczek/acme-restful/pkg/backend/fs"
	"github.com/skroczek/acme-restful/pkg/router"
	"github.com/skroczek/acme-restful/pkg/server"
)

func main() {
	var err error
	root, err := filepath.Abs(os.Args[1])
	if err != nil {
		panic(err)
	}
	log.Printf("root: %s", root)

	be := fs.NewFilesystemBackend(root, fs.WithCreateDirs(), fs.WithDeleteEmptyDirs())
	// the encrypted backend acts as a proxy before the actual backend
	// the passphrase is read from the environment variable ACME_RESTFUL_PASSPHRASE
    // if the passphrase is not set, the backend will panic
	encryptedBackend := backend.NewEncrypted(be)
	s := server.NewServer(
		server.WithBackend(encryptedBackend),
		// You can additional add the encrypted backend to encrypt the data as rest. But you have to set the passphrase
		// in the environment variable ACME_RESTFUL_PASSPHRASE
		//pkg.WithBackend(backend.NewEncrypted(be)),
		server.WithRouterOptions(
			router.WithDefaultCors(true),
		),
		server.WithListAll(),
		server.WithGetAll(),
	)
	// By default, it serves on :8080 unless a
	// PORT environment variable was defined.
	s.Run()
}
```

# License
You can find the license in the LICENSE file.