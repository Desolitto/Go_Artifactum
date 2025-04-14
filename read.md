# Artifactum Project

## Overview
Artifactum is a command-line application designed to manage and analyze a distributed database of artifacts. The application supports scalability, balancing, and consensus among multiple database instances, ensuring fault tolerance and data integrity.

## Features
- **Scalability**: Ability to dynamically add or remove database instances while maintaining service availability.
- **Fault Tolerance**: The system continues functioning even if some instances fail.
- **Data Consistency**: Implements eventual consistency for data operations, ensuring that all nodes eventually reflect the same state.
- **Leader-Follower Architecture**: Introduces Leader and Follower nodes to manage requests and maintain synchronization.
- **UUID4 Keys**: Utilizes UUID4 strings as unique identifiers for artifacts.
- **Heartbeat Mechanism**: Regularly checks the status of nodes to ensure they are operational and updates the list of available nodes.

## Getting Started

### Prerequisites
- Go programming language installed (version 1.16 or higher recommended)
- Git for version control

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/Desolitto/Go_Artifactum.git
   ```
2. Navigate to the project directory:
   ```bash
   cd Go_Artifactum
   ```

### 1. Starting the Servers

You can choose one of two methods to start the servers: manually or using Makefile.

#### Option 1: Manual Start

1. **Compile the server:**
   ```bash
   cd bd && go build cmd/server.go
   ```

2. **Start the primary server:**
   ```bash
   cd bd && ./server
   ```

3. **Start an additional server with specified addresses and ports:**
   ```bash
   cd bd && ./server -HBD 127.0.0.1 -P <PORT> -PBD <PBD>
   ```
   Replace `<PORT>` and `<PBD>` with your desired values. For example, to start a second server on port 1235 and a base port of 8765:
   ```bash
   cd bd && ./server -HBD 127.0.0.1 -P 1235 -PBD 8765
   ```

#### Option 2: Using Makefile

If you prefer to use Makefile to simplify the process, you can execute the following commands:

- **Build the primary server:**
   ```bash
   make server
   ```

- **Start the primary server:**
   ```bash
   make serverP
   ```

- **Start an additional server with parameters:**
   ```bash
   make serverF PORT=1235 PBD=8765
   ```

### 2. Starting the Client

After starting the servers, open a new terminal and run the following command to start the client. The client can connect to different ports as specified in the command line arguments:

```bash
cd client && go run cmd/client.go -H <HOST> -P <PORT>
```
Replace `<HOST>` and `<PORT>` with your desired values. By default, the client connects to `127.0.0.1:8765`. Alternatively, you can use Makefile to build and run the client:
```bash
make client
```

### Example Session

Here's what a typical session should look like, with comments (starting with `#`):

```bash
~$ ./warehouse-cli -H 127.0.0.1 -P 8765
Connected to a database of Warehouse 13 at 127.0.0.1:8765
Known nodes:
127.0.0.1:8765
127.0.0.1:9876
127.0.0.1:8697
> SET 0d5d3807-5fbf-4228-a657-5a091c4e497f '{"name": "Chapayev's Mustache comb"}'
Created (2 replicas)
> GET 0d5d3807-5fbf-4228-a657-5a091c4e497f
'{"name": "Chapayev's Mustache comb"}'
> DELETE 0d5d3807-5fbf-4228-a657-5a091c4e497f
Deleted (2 replicas)
> GET 0d5d3807-5fbf-4228-a657-5a091c4e497f
Not found
>
# if current instance is stopped in the background
Reconnected to a database of Warehouse 13 at 127.0.0.1:8697
Known nodes:
127.0.0.1:9876
127.0.0.1:8697
> 
# if another current instance is stopped in the background
Reconnected to a database of Warehouse 13 at 127.0.0.1:9876
Known nodes:
127.0.0.1:9876
WARNING: cluster size (1) is smaller than a replication factor (2)!
>
```

## Project Structure

The project structure is as follows:

```
src
├── api
├── bd
│   ├── api
│   ├── cmd
│   ├── config
│   └── internal
├── client
│   ├── cmd
│   ├── config
│   └── internal
├── go.mod
└── go.sum
```

Explanation of the Directories:

- **api**: This directory likely contains the API-related code, such as endpoints, request/response handling, and data models.
- **bd**: This directory contains the server-side implementation, including the command-line interface, configuration, and internal packages.
- **client**: This directory contains the client-side implementation, including the command-line interface, configuration, and internal packages.
- **cmd**: This directory may contain the main entry points for the server and client applications.
- **config**: This directory likely contains configuration-related files, such as settings, environment variables, and other configurations.
- **go.mod** and **go.sum**: Files managing the Go module dependencies in the project.
