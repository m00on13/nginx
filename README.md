# NGINX Crash Course - Load Balancing Project

A learning project demonstrating NGINX reverse proxy and load balancing concepts through a practical implementation with Docker, Docker Compose, and Node.js.

## Project Overview

This project showcases how NGINX can be configured as a reverse proxy and load balancer to distribute incoming client requests across multiple upstream servers (Node.js application instances). It serves as a hands-on exploration of core NGINX concepts and their real-world applications.

### Key Technologies
- **NGINX**: Reverse proxy and load balancer
- **Node.js**: Backend application server (Express.js)
- **Docker & Docker Compose**: Containerization and orchestration
- **Express**: Lightweight web framework for Node.js

---

## Learning Objectives

This project covers the following NGINX concepts:

### 1. **Worker Processes**
   - NGINX spawns multiple worker processes to handle concurrent client requests
   - Each process operates independently to manage traffic efficiently
   - Configuration: `worker_processes auto;` auto-detects available CPU cores
   - **Best Practice**: Set worker processes equal to the number of CPU cores on your server

### 2. **Worker Connections**
   - Defines the maximum number of simultaneous connections each worker can handle
   - Higher values allow more concurrent requests but increase memory usage
   - Each connection consumes resources; must respect system's file descriptor limits
   - Configuration: `worker_connections 1024;` (default and common setting)

### 3. **Upstream & Downstream**
   - **Upstream**: Traffic flowing from client toward backend servers (higher-level infrastructure)
   - **Downstream**: Traffic flowing back from backend servers to the client
   - NGINX sits in the middle, managing request routing and response transmission

### 4. **Reverse Proxy**
   - NGINX intercepts client requests and forwards them to backend servers
   - Shields backend servers from direct client exposure
   - Can add/modify headers like `Host` and `X-Real-IP` for backend context
   - Improves security, performance, and load distribution

### 5. **Load Balancing**
   - Distributes incoming requests across multiple backend servers
   - **Algorithm used in this project**: `least_conn` (least connections)
   - Ensures no single server becomes a bottleneck
   - Improves application availability and fault tolerance

---

## Project Architecture

```
                        Client
                          |
                          |
                    ┌─────▼─────┐
                    │   NGINX    │
                    │   Reverse  │
                    │   Proxy    │
                    └─────┬─────┘
                          |
            ┌─────────────┼─────────────┐
            |             |             |
        ┌───▼──┐      ┌───▼──┐     ┌───▼──┐
        │ App1 │      │ App2 │     │ App3 │
        │ :3001│      │ :3002│     │ :3003│
        └──────┘      └──────┘     └──────┘
```

**Flow**:
1. Client sends HTTP request to NGINX on port 8080
2. NGINX evaluates the request against configured location blocks
3. Using the `least_conn` algorithm, NGINX forwards the request to the upstream server with the fewest active connections
4. Backend server (App1/App2/App3) processes the request and returns response
5. NGINX forwards response back to client with original headers intact

---

## Getting Started

### Prerequisites
- Docker
- Docker Compose
- (Optional) NGINX installed locally for configuration testing

### Installation & Setup

1. **Clone/Navigate to the project directory**
   ```bash
   cd c:\Mansi\nginx
   ```

2. **Start the application and NGINX**
   ```bash
   docker-compose up --build
   ```
   - This builds and starts 3 Node.js application instances (App1, App2, App3)
   - Each instance runs on a different port: 3001, 3002, 3003
   - All are accessible through NGINX on localhost:8080

3. **Test the setup**
   ```bash
   curl http://localhost:8080
   ```
   - Open browser to `http://localhost:8080`
   - Make multiple requests to see load distribution across different app instances
   - Check terminal logs to see which app instance handled each request

4. **Stop the application**
   ```bash
   docker-compose down
   ```

---

## Project Structure

```
nginx/
├── README.md                    # This file
├── server.js                    # Express.js backend server
├── package.json                 # Node.js dependencies
├── index.html                   # Frontend HTML served by Node.js
├── docker-compose.yaml          # Docker Compose configuration (3 app instances)
├── Dockerfile                   # Docker image definition for Node.js app
├── nginx-config.txt             # Detailed NGINX configuration with comments
├── certs/                       # SSL/TLS certificates (optional)
└── images/                      # Static images served by the application
    ├── banner.avif
    └── cast.avif
```

---

## NGINX Configuration Details

### Server Block Configuration

```nginx
server {
    listen 8080;
    server_name localhost;
    
    location / {
        proxy_pass http://nodejs_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Key Configuration Points

- **`listen 8080`**: NGINX listens on port 8080 for incoming connections
- **`server_name localhost`**: This config applies to requests to localhost
- **`proxy_pass http://nodejs_cluster`**: Routes requests to the upstream cluster
- **`proxy_set_header Host $host`**: Forwards the original host header to backend
- **`proxy_set_header X-Real-IP $remote_addr`**: Preserves client IP address for backend logging

### Load Balancing Algorithm: Least Connections

```nginx
upstream nodejs_cluster {
    least_conn;  # Sends requests to server with fewest active connections
    
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

**Why least_conn?**
- Better distribution than simple round-robin for variable request processing times
- Automatically routes new requests to the least busy server
- Ideal for applications with varying response times

---

## Testing Load Balancing

### Method 1: View Container Logs
```bash
docker-compose logs -f
```
Each request will show which app instance handled it.

### Method 2: Use a Load Test Tool
```bash
# Using Apache Bench (if installed)
ab -n 100 -c 10 http://localhost:8080/

# Or using curl in a loop
for i in {1..20}; do curl http://localhost:8080/; done
```

### Observation
- Watch the logs to see requests distributed across App1, App2, and App3
- The `least_conn` algorithm ensures balanced distribution based on active connections

---

## NGINX Concepts Deep Dive

### HTTP Module Components

1. **mime.types**: Defines file types and their MIME types for HTTP response headers
2. **upstream block**: Defines a pool of backend servers for load balancing
3. **server block**: Configures how NGINX handles requests for a specific domain/IP
4. **location block**: Defines request handling rules based on URL paths

### Performance Tuning

- **Worker Processes**: Match to CPU core count for optimal performance
- **Worker Connections**: Increase if handling high traffic (balance with system limits)
- **Keep-Alive**: Reuse connections for multiple requests
- **Caching**: Cache static assets to reduce backend load

---

## Security Considerations

- NGINX acts as a shield between clients and backend servers
- Backend servers are not directly exposed to the internet
- SSL/TLS certificates can be configured for encrypted communication
- Request headers can be sanitized before forwarding to backends

---

## Common NGINX Commands

```bash
# Test configuration syntax
nginx -t

# Reload NGINX with updated configuration
nginx -s reload

# Display NGINX version and compiled modules
nginx -v

# View running NGINX processes
ps aux | grep nginx
```

---

## Further Learning

- Experiment with different load balancing algorithms (round_robin, ip_hash, least_time)
- Add SSL/TLS configuration with HTTPS
- Implement caching strategies for static content
- Configure rate limiting to prevent abuse
- Set up health checks for backend servers
- Explore NGINX modules for additional functionality

