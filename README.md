# Multithreaded HTTP Server

A multithreaded HTTP server developed as part of the Operating Systems course at the Technion – Israel Institute of Technology in 2025.

The project focuses on concurrent request handling, thread synchronization, request scheduling, and resource management using Linux POSIX APIs.

## Project Overview

The server handles incoming HTTP requests using a fixed-size pool of worker threads, alongside a dedicated thread for VIP requests.

Its main goals are to:
- Distribute incoming requests across multiple threads.
- Ensure thread-safe access to shared resources.
- Coordinate thread execution using synchronization mechanisms.
- Handle server overload according to configurable scheduling policies.
- Collect and report statistics about requests and worker threads.

## Architecture and Functionality

### Thread Management

- The server creates a fixed-size pool of worker threads.
- Each worker thread handles one request at a time.
- A separate, dedicated VIP thread handles VIP requests and remains idle until a VIP request arrives.
- The server coordinates request execution and manages shared resources to ensure correct concurrent behavior.

### Request Management

The implementation maintains three data structures:

- **Handled Requests List:** Contains requests currently being processed by worker threads.
- **Waiting Requests Queue:** Contains admitted requests waiting for an available worker thread.
- **VIP Requests List:** Stores VIP requests for the dedicated VIP thread.

The server has a maximum request capacity, defined by the combined number of requests currently being processed and admitted requests waiting in the queues, including VIP requests.

### Supported Request Types

The server supports GET requests and the following request types:

- **VIP Requests (REAL):** Special requests handled by the dedicated VIP thread.
- **`.skip` Requests:** Requests whose URL ends with `.skip`. After a `.skip` request is executed, the last admitted request in the waiting queue is selected for immediate handling. The VIP thread must wait until this selected request has finished before processing another VIP request.

The content of the requests can be one of the two:

- **Static Requests:** Requests for static files.
- **Dynamic Requests:** Requests that require executing a CGI process.


### Overload Handling Policies

The server supports five configurable policies for handling incoming requests when the server reaches its capacity:

| Policy | Description |
|---|---|
| `block` | Blocks an incoming request until a slot becomes available. |
| `dt` (Drop Tail) | Drops the newly arriving request. |
| `dh` (Drop Head) | Drops the first unhandled admitted request, then admits the new request. |
| `bf` (Block Flush) | Blocks an incoming request until there are no unhandled admitted requests. |
| `random` (Drop Random) | Drops 50% of the requests in the waiting queue randomly, then admits the new request. |

When the server is full of VIP requests, the blocking policy is applied.

## Request and Thread Statistics

The server collects statistics and includes relevant information in HTTP response headers.

**Per-request statistics:**
- Arrival time.
- Dispatch time.
- ID of the thread handling the request.

**Per-thread statistics:**
- Total number of requests handled.
- Number of static requests handled.
- Number of dynamic requests handled.

These statistics provide visibility into request processing and thread workload distribution.

## Technologies and Development Environment

- **Programming Language:** C
- **Operating System:** Linux
- **Development Environment:** Visual Studio Code
- **Platform and APIs:** POSIX
- **Multithreading:** Pthreads
- **Synchronization:** POSIX mutexes, condition variables, and semaphores
- **Networking:** Sockets
- **Process Management:** POSIX system calls and interfaces, including `fork()` and `wait()`
- **Build System:** Makefile

The project uses the relevant POSIX headers and interfaces for thread management, synchronization, networking, and process management.

The client functionality and the initial client-server connection mechanism were provided by the course instructors. The project focused on implementing the server-side logic.

## Technical Challenges

One of the main challenges was designing the synchronization logic between worker threads while preserving the intended execution flow.

This involved deciding:
- Where mutex locks and unlocks were required.
- Which shared variables needed protection.
- When condition variables should be signaled.
- How to coordinate waiting threads and prevent race conditions.

Another challenge was adding statistical information to the HTTP response headers. This required understanding the existing HTTP server implementation, studying the relevant protocol behavior, and modifying the response headers to include the collected statistics.

## Testing and Validation

Testing included both manual testing and automated tests provided by the course staff.

**Manual testing involved:**
- Running the server and clients locally.
- Establishing connections and sending different requests.
- Examining request and thread statistics.
- Comparing the reported statistics and timestamps against predefined request sequences.
- Checking that requests were handled according to the expected scheduling behavior.

The implementation was also tested against the automated tests provided by the course staff and passed the required tests.

## Building and Running

The project includes a Makefile for compiling the server and related files, as well as the client used for testing.

Run the server using:

```bash
./server [portnum] [threads] [queue_size] [schedalg]
```

**Arguments:**

| Argument | Description |
|---|---|
| `portnum` | Port on which the server listens. |
| `threads` | Number of worker threads, excluding the dedicated VIP thread. |
| `queue_size` | Maximum number of requests the server can accommodate. |
| `schedalg` | Overload-handling policy: `block`, `dt`, `dh`, `bf`, or `random`. |

## Team and Contributions

Developed collaboratively with **Sondos Taha** ([GitHub](https://github.com/SondosAhmedTaha)).

Both team members participated throughout the project, including the overall design, implementation, debugging, and testing.

## Skills Demonstrated

- Concurrent programming and multithreading.
- Thread synchronization using mutexes, condition variables, and semaphores.
- Race-condition prevention and thread-safe resource management.
- Request scheduling and overload handling.
- Linux and POSIX programming.
- HTTP and client-server networking.
- Testing and debugging concurrent systems.

---

*This project was developed as an academic assignment. Some supporting components, including the client functionality and initial connection mechanism, were provided by the course instructors.*
