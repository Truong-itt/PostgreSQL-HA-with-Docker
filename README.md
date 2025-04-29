# PostgreSQL High Availability Cluster with Patroni and etcd

This project sets up a high-availability PostgreSQL cluster using **Patroni**, **etcd**, **HAProxy**, and **pgweb**, orchestrated with **Docker** and **Docker Compose**. The cluster includes three PostgreSQL nodes managed by Patroni for automatic failover and replication, three etcd nodes for distributed configuration and leader election, HAProxy for load balancing, and pgweb for web-based database administration.

## Components
- **PostgreSQL**: The primary database, managed by Patroni for high availability.
- **Patroni**: Manages the PostgreSQL cluster, handling automatic failover and replication.
- **etcd**: A distributed key-value store that maintains cluster state and performs leader election.
- **HAProxy**: A load balancer that directs connections to the primary node (port 5000) or replica nodes (port 5001).
- **Docker**: Containerizes all components for easy deployment and management.
- **pgweb**: A web-based interface for interacting with the PostgreSQL database.

## Prerequisites
- **Docker**
- **Docker Compose**

## Project Structure
- `./etcd`: Dockerfile and configuration for etcd nodes.
- `./patroni`: Dockerfile and configuration for Patroni-managed PostgreSQL nodes.
- `./haproxy`: Dockerfile and configuration for HAProxy.
- `docker-compose.yml`: Defines services, networks, and volumes for the cluster.
- `./patroni/patroni_node{1,2,3}/patroni.yml`: Patroni configuration for each PostgreSQL node.

## Setup Instructions
1. **Clone the Repository**
   ```bash
   git clone https://github.com/Truong-itt/PostgreSQL-HA-with-Docker.git
   cd 
   ```

2. **Build and Start the Cluster**
   ```bash
   docker-compose up -d --build
   ```
   This command builds the Docker images and starts the services in detached mode.

3. **Verify Services**
   - **etcd Cluster Health**:
     ```bash
     docker exec etcd1 etcdctl --endpoints=http://127.0.0.1:2379 endpoint health
     docker exec -it etcd1 etcdctl member list
     docker exec -it etcd1 etcdctl cluster-health
     ```
     These commands check the health of etcd nodes and the cluster status.
   - **Patroni Cluster Status**:
     ```bash
     docker exec pg_node1 patronictl -c /etc/patroni.yml list
     docker exec -it pg_node1 curl http://localhost:8008/health
     ```
     These commands list the Patroni cluster members and verify node health.
   - **PostgreSQL Connectivity**:
     ```bash
     psql -h localhost -p 5000 -U postgres -c "SELECT 1;"
     psql -h localhost -p 5001 -U postgres -c "SELECT 1;"
     ```
     These commands test connections to the primary (port 5000) and replica (port 5001) nodes.
   - **pgweb**: Access the web interface at `http://localhost:8007`.
   - **HAProxy Stats**: View load balancer statistics at `http://localhost:7000/stats` (default credentials: admin/admin).

## Extend Commands
- **Build Patroni Image Without Cache**:
  ```bash
  docker build --no-cache -t patroni ./patroni
  ```
  Rebuilds the Patroni image without using cached layers.
- **Inspect Patroni Configuration**:
  ```bash
  docker run -it --rm -v $(pwd)/patroni/patroni_node2/patroni.yml:/etc/patroni.yml patroni cat /etc/patroni.yml
  ```
  Displays the Patroni configuration for node 2.
- **Access Patroni Container Shell**:
  ```bash
  docker run -it --rm patroni bash
  ```
  Opens a bash shell in a temporary Patroni container.
- **Check PostgreSQL Data Directory Permissions**:
  ```bash
  ls -ld /var/lib/postgresql/data
  ```
  Verifies permissions inside a PostgreSQL container (run within the container).

## Stopping the Cluster
To stop the services:
```bash
docker-compose down
```

To stop and remove persistent data (volumes):
```bash
docker-compose down -v
docker volume prune
```

## Notes
- The `postgres-ha` bridge network is automatically created by Docker Compose for communication between services.
- Patroni configurations are mounted from `./patroni/patroni_node{1,2,3}/patroni.yml`.
- PostgreSQL data is stored in Docker volumes (`patroni_data_node{1,2,3}`) for persistence.
- HAProxy configuration is located in `./haproxy/haproxy.cfg`.
- Port mappings:
  - etcd: 2379/2380 (etcd1), 2381/2382 (etcd2), 2383/2384 (etcd3).
  - PostgreSQL/Patroni: 5432/8008 (pg_node1), 5433/8009 (pg_node2), 5434/8010 (pg_node3).
  - HAProxy: 5000 (read/write), 5001 (read-only), 7000 (stats).
  - pgweb: 8007.

## Troubleshooting
- **View Container Logs**:
  ```bash
  docker logs <container_name>
  ```
  Check logs for errors in etcd, Patroni, HAProxy, or pgweb containers.
- **etcd Issues**: Ensure all etcd nodes are healthy and communicating (`etcdctl cluster-health`).
- **Patroni Issues**: Verify the cluster state with `patronictl list` and check Patroni logs.
- **PostgreSQL Connectivity**: Ensure HAProxy is routing traffic correctly to the primary/replica nodes.
- **HAProxy Stats**: Use `http://localhost:7000/stats` to monitor load balancer status and node availability.

## Additional Resources
- [Percona PostgreSQL High Availability Documentation](https://docs.percona.com/postgresql/15/solutions/high-availability.html)