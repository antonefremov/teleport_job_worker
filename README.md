## Overview

This is a prototype job worker service that provides an API to run arbitrary
Linux processes.

## Proposed Technical Architecture

Before a draft simplified Technical Architecture can be drawn, let's make a few design assumptions.

#### First of all, the app will provide 3 gRPC APIs:

1. Client schedules several jobs on server (client-streaming gRPC).
This is a client-streaming RPC API that allows client to send multiple commands to the server.

Server responds with a CorrelationID for the jobs.

2. Client queries the batch job results and logs by a given CorrelationID (unary gRPC).
Server responds with results of execution for each command and corresponding logs by a passed CorrelationID.

3. After Client has scheduled several jobs on server and received a CorrelationID, they can connect to a streaming log output of a running batch process (server-streaming gRPC).

#### Secondly, the app will have the following properties:

- The app won't have any db persistency. Therefore, all the execution results and logs will be saved locally in files on a Linux machine running the app.
- Each batch should be associated with a unique CorrelationID so that all executable commands within a batch should be easily found in the Logs and Output. 
- The app will not have its own surrounding PKI to be used, so the corresponding certificates will be generated during the app installation and re-used in all subsequent interactions thereafter.

The simplified Technical Architecture of the app will look as follows:
![Architecture image](/proposed_arch.jpg)

## Proposed features of the app

- Streaming output
- Dynamic config
- Logging
- Monitoring

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
