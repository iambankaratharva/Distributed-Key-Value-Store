# In-Memory + Persistent Distributed Key-Value Store

## Overview

This project implements a distributed key-value store, which will be used as the storage system. The store is designed to be sharded across multiple worker nodes, using consistent hashing to determine key ranges. A single coordinator node manages the system membership and keeps track of active workers. The key-value store supports basic PUT and GET operations.

## Functionality

### Coordinator

- **Initialization**: The coordinator is initialized with a command-line argument specifying the port number for the web server.
- **Worker Management**:
  - **/ping?id=x&port=y**: Registers or updates a worker with ID `x` and port `y`. Returns "OK" on success, or a 400 error if `id` or `port` are missing.
  - **/workers**: Returns a list of active workers in the format `k\nid1,ip1:port1\nid2,ip2:port2\n...`, where `k` is the number of active workers.
  - **Home Page**: Returns an HTML page with a table listing each active worker, its ID, IP, and port, with hyperlinks to the worker's main page.
- **Worker Inactivity**: Workers that do not send a `/ping` request within 15 seconds are considered inactive and removed from the list.

### Worker

- **Initialization**: The worker is initialized with three command-line arguments: port number, storage directory, and the coordinator's IP and port.
- **ID Management**: Each worker has a unique ID stored in the `id` file within the storage directory. If the file does not exist, a new ID is generated and saved.
- **Heartbeat**: Sends a `/ping` request to the coordinator every five seconds to indicate it is active.
- **Data Management**:
  - **PUT /data/<T>/<R>/<C>**: Stores data in column `C` of row `R` in table `T`.
  - **GET /data/<T>/<R>/<C>**: Retrieves data from column `C` of row `R` in table `T`. Returns a 404 error if the table, row, or column does not exist.
- **Case Sensitivity**: Row and column keys are case-sensitive.

### Consistent Hashing

- **Key Distribution**: Workers are responsible for keys in the range between their ID and the next higher worker ID. The highest ID worker also handles keys higher than its ID and lower than the lowest ID.
