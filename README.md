## Service use case

This is a prototype job worker service that provides an API to run arbitrary
Linux processes.

#### The service is serving the following main scenarios:

1. Client schedules one or many jobs on server (unary gRPC).
This is a client-streaming RPC API that allows client to send multiple commands to the server. Server responds with a CorrelationID for the jobs in a batch.

2. Client queries the batch job results and logs by a given CorrelationID (unary gRPC).
Server responds with results of execution for each command and corresponding logs by a passed CorrelationID.

3. After Client has successfully scheduled their batch on server and received a CorrelationID, they're able to connect to a streaming log output of a running batch process (server-streaming gRPC).

#### The client's request flow and response from the service look as per below scheme:
![Architecture image](/proposed_arch.jpg)

## Design assumptions

#### Persistency, Logging and Storage
- The service does not have a db persistency. This means that all the logs and execution results are saved locally in separate files on a Linux machine running the service. I/O sync objects are used for querying/writing by multiple clients.

#### Streaming output
- An API are provided for the clients to receive a streaming log output of a running job process. 

#### Administration/Monitoring
- The app must have an API to start/stop the service and to query it's current status by respective admin staff.

#### Traceability
- Each batch is associated with a unique CorrelationID so that all executable commands within a batch are searchable in the Logs and Output files.

#### Security
- The communication channel between the client and server is secured by mutual TLS so that both must provide their TLS certificates to the other. The service environment does not have its own surrounding PKI to be leveraged, so the corresponding mock certificates are generated during the app installation and re-used in all subsequent interactions betwen the client and the service thereafter.

#### Configuration
- The app supports dynamic config on start up and during the run.

## Service APIs

Service provides 3 sets of gRPC APIs:

1. Auth service
```
message LoginRequest {
  string username = 1;
  string password = 2;
}

message LoginResponse { string access_token = 1; }

service AuthService {
  rpc Login(LoginRequest) returns (LoginResponse) {
    option (google.api.http) = {
      post : "/v1/auth/login"
      body : "*"
    };
  };
}
```

2. Job Schedule service
```
message JobRequest {
    string command = 1;
}

message JobResponse {
    string correlation_id = 1;
    string result = 2;
    google.protobuf.Timestamp executed_at = 3;
}

service JobScheduleService {
  rpc ExecuteJob(JobRequest) returns (JobResponse) {
      option (google.api.http) = {
      post : "/v1/job/schedule"
      body : "*"
    };
  };
}
```

3. Job Query service
```
message SearchJobRequest {
    string correlation_id = 1;
}

message SearchJobResponse {
    string correlation_id = 1;
    string result = 2;
    string log_output = 3;
    google.protobuf.Timestamp executed_at = 4;
}

rpc SearchJob(SearchJobRequest) returns (stream SearchJobResponse) {
    option (google.api.http) = {
      get : "/v1/job/search"
    };
  };
```

## Quick start

### Server Deployment

1. Clone this project into a directory
2. Open the project folder in terminal and install the required dependencies by running the command below
```sh
$ make build
```
3. In the project folder start the server by running the following terminal command
```sh
$ make run
```

### Client Deployment
To be confirmed...

## Testing
```sh
$ go test -v -race
```
To be continued...
