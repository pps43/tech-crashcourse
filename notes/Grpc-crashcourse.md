# [Grpc](https://grpc.io/)


You have a service written in C++, client A in Android, and client B in Ruby.
All clients need to have a `grpc Stub` component, which handles request/response with `grpc server`.


## protobuf

`protobuf` is in the core of grpc to define messages and service, and communication (instead of `json`)


### .proto
```proto
syntax = "proto3";

message Greeting {
    string first_name = 1;
}

message GreetRequest {
    Greeting = 1;
}

message GreetResponse {
    string result = 1;
}

service GreetService {
    rpc Greet(GreetRequest) returns (GreetResponse) {};
}
```

### advantages
- Bakcward compatibility 

API can evolve without breaking existing clients.