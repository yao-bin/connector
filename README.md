# Project Overview
connector intends to provide a simpler coding experience with gRPC, and microservices development.

When it comes to setting up gRPC server, connecting from gRPC client, as well as supporting microservices related needs,
instead of creating a large framework or package, with many features that we may not use, an emphasis is placed on coding
productivity as well as reducing amount of repetitive typing / boilerplate coding.

On the server end, the entry point is /service folder's Service struct. 
On the client side, the entry point is /client folder's Client struct.
Both server and client supports config file based setup.

Since we use AWS extensively, many features integrates with AWS: (Goal is for these features to work automatically with minimum configuration)

- Service Discovery
    - connector's service upon launch, auto registers with AWS cloud map for service discovery
    - On the client side, service discovery is automatic, see config for discovery options
- Health Checks
    - service instance health is integrated with cloud map's health check status
    - auto register and deregister on service discovery based on instance health
    - container-level service serving status utilizes the gRPC health v1
    - config can be set on client to auto health check for serving status, as well as manual health probe
    - TODO: need to create out of process health witness for instance health management
- Load Balancing
    - client side name resolver is setup to retrieve multiple service endpoints and perform round robin load balancing
    - load balancing is per rpc call rather than per connection
- Metadata
    - Metadata helper methods provided
- RpcError
    - Rpc Error helper methods provided
- Compressor
    - gzip decompressor supported on service level
    - note added on client struct for passing gzip compressor via RPC call
- Server TLS / mTLS
    - server TLS / mTLS is configured via service or client config file
    - see /build/openssl-pem/make-pem.sh for CA, Server and Client Pem and Key self-signed creation  
    - server TLS / mTLS setup in gRPC service and client is required in order to secure channel
- Auth
    - TODO: will integrate via interceptor
- Circuit Breaker
    - client side, default using Hystrix-Go package for circuit breaker
    - circuit breaker is handled in client side unary and stream interceptors
    - circuit breaker options configured via client config file
- Rate Limiter
    - server side, default using Uber-RateLimit package for rate limiter
    - rate limit is handled in server side In-Tap-Handle
    - rate limit option configured via server config file
- Logger
    - TODO: Currently local logging using log.* but will update to zap
- Monitoring
    - TODO: looking to use prometheus, but might end up using something else
- Tracer
    - TODO: planning on using aws xray
- Queue
    - TODO: SQS
- Notification
    - TODO: SNS

# Service connector
#### Overview
/service folder contains the gRPC service (server) wrapper.  
#### Example of Use
See /example/cmd/server for a working gRPC server setup that serves test service
#### Notes
- TLS self sign certificates must be setup and placed into x509 sub folder
- Use /build/openssl-pem/make-pem.sh to create self-signed openssl ca, cert and key
- the service.yaml config file must be set properly and aws resources enabled
- to save aws access id and secret key, use aws cli => aws configure

# Client connector
#### Overview
/client folder contains the gRPC client (dialer) wrapper.
#### Example of Use
See /example/cmd/client for a working gRPC client setup to consume the gRPC service server
#### Notes
- Each gRPC server service that client needs to consume, create its own service yaml in /endpoint folder
- Each target gRPC service is described via service yaml config file in endpoint, so be sure to correctly define its config values
- TLS self sign certificates must be setup and placed into x509 sub folder
- Use /build/openssl-pem/make-pem.sh to create self-signed openssl ca, cert and key
- to save aws access id and secret key, use aws cli => aws configure

# Pre-Requisites
#### 1) Install or Upgrade ***protoc***
- In Terminal: (Mac)
    - ~ brew install protobuf
    - ~ brew upgrade protobuf
#### 2) Install ***protoc-gen-go***
- In Terminal:
    - ~ go install google.golang.org/protobuf/cmd/protoc-gen-go
    - More Info: https://developers.google.com/protocol-buffers/docs/reference/go-generated
#### 3) Install ***protoc-gen-go-grpc***
- install ***go-grpc_out***
    - Info: https://github.com/grpc/grpc-go/tree/master/cmd/protoc-gen-go-grpc
    - Info: https://grpc.io/docs/languages/go/quickstart/
- In Terminal:
    - ~ export GO111MODULE=on
    - ~ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
#### 4) Info on ***proto3***
- https://developers.google.com/protocol-buffers/docs/gotutorial
- Define ***package*** using OrganizationOrProject.Service.Type pattern
    - com.company.service
    - project.proto.service
    - domain.service.type, etc.
- Use ***option go_package = "xyz";***
    - xyz should point to the full path from $GOPATH/src to this proto file folder
    - for example, "github.com/aldelo/connector/example/proto/test"
        - where "test" is the folder that contains proto files
- ***import*** should contain full path from project folder to the proto file
    - in GoLand (JetBrains), Preferences -> Languages & Preferences -> Protocol Buffers
        - Uncheck "Configure Automatically"
        - Add path to $GOPATH/src
#### 5) Executing ***protoc***
- In Terminal:
    - go to the folder containing .proto files
    - ~ protoc --go_out=$GOPATH/src --go-grpc_out=$GOPATH/src --proto_path=$GOPATH/src $GOPATH/src/xyz.../*.proto
        - where xyz... refers to the actual full path below $GOPATH/src up to the folder containing the proto files
    
# Checking Version Info
#### 1) Latest protoc Version on brew
- https://formulae.brew.sh/formula/protobuf
#### 2) Latest protobuf Version
- https://github.com/golang/protobuf/releases
#### 3) Latest grpc for go Version
- https://github.com/grpc/grpc-go/releases
#### 4) Latest protoc-gen-go Version
- https://pkg.go.dev/google.golang.org/protobuf/cmd/protoc-gen-go
#### 5) Latest protoc-gen-go-grpc Version
- https://github.com/grpc/grpc-go/tree/master/cmd/protoc-gen-go-grpc
#### 6) Latest genproto Version
- https://pkg.go.dev/google.golang.org/genproto
