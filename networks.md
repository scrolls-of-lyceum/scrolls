# Docker Networks

Docker networks provide a way for Docker containers to communicate with each other, with the host, and with external networks. Docker offers several network drivers, each suited for different use cases. Understanding these networks and their scenarios helps in building efficient, secure, and scalable containerized applications.

### Types of Docker Networks

1. **Bridge Network**:
   - Default network driver for standalone containers.
   - Containers on the same bridge network can communicate with each other using IP addresses or container names.
   - Suitable for applications that run on a single host.

2. **Host Network**:
   - Removes network isolation between the container and the Docker host.
   - The container uses the host's networking directly.
   - Useful for performance-sensitive applications where network stack overhead is undesirable.

3. **Overlay Network**:
   - Enables communication between containers across multiple Docker hosts.
   - Requires Docker Swarm or Kubernetes.
   - Ideal for distributed applications and microservices that need to communicate across different nodes.

4. **Macvlan Network**:
   - Assigns a MAC address to each container, making it appear as a physical device on the network.
   - Containers can have direct access to the network.
   - Suitable for legacy applications that require direct L2 network access.

5. **None Network**:
   - Containers have no network access.
   - Useful for running containers that don’t require network connectivity or for security isolation.

### Creating and Managing Docker Networks

1. **Creating a Network**:
   - **Bridge Network**:
     ```sh
     docker network create my_bridge_network
     ```
   - **Overlay Network**:
     ```sh
     docker network create --driver overlay my_overlay_network
     ```
   - **Macvlan Network**:
     ```sh
     docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 my_macvlan_network
     ```

2. **Connecting a Container to a Network**:
   ```sh
   docker run -d --name my_container --network my_bridge_network my_image
   ```

3. **Connecting an Existing Container to a Network**:
   ```sh
   docker network connect my_bridge_network my_container
   ```

4. **Disconnecting a Container from a Network**:
   ```sh
   docker network disconnect my_bridge_network my_container
   ```

5. **Listing Networks**:
   ```sh
   docker network ls
   ```

6. **Inspecting a Network**:
   ```sh
   docker network inspect my_bridge_network
   ```

7. **Removing a Network**:
   ```sh
   docker network rm my_bridge_network
   ```

### Scenarios and Use Cases

1. **Single-Host Communication**:
   - **Bridge Network**: 
     - Use when you have multiple containers running on a single host that need to communicate with each other.
     - Example: A web server container and a database container on the same host.
     ```sh
     docker network create my_bridge_network
     docker run -d --name web --network my_bridge_network my_web_image
     docker run -d --name db --network my_bridge_network my_db_image
     ```

2. **Performance-Critical Applications**:
   - **Host Network**: 
     - Use for applications that require high performance and low latency.
     - Example: Running a high-performance web server.
     ```sh
     docker run -d --network host my_high_perf_web_image
     ```

3. **Multi-Host Communication**:
   - **Overlay Network**:
     - Use for applications spread across multiple hosts, such as in a Docker Swarm or Kubernetes cluster.
     - Example: A microservices architecture where services are distributed across multiple nodes.
     ```sh
     docker network create --driver overlay my_overlay_network
     docker service create --name my_service --network my_overlay_network my_service_image
     ```

4. **Direct Network Access**:
   - **Macvlan Network**:
     - Use when containers need direct access to the physical network.
     - Example: Legacy applications that require MAC addresses or are not compatible with NAT.
     ```sh
     docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 my_macvlan_network
     docker run -d --name legacy_app --network my_macvlan_network my_legacy_image
     ```

5. **Isolated Environments**:
   - **None Network**:
     - Use for containers that do not need network access.
     - Example: Running batch jobs or tasks that do not require network connectivity.
     ```sh
     docker run -d --network none my_batch_job_image
     ```

### Advanced Networking Concepts

1. **Network Aliases**:
   - Provide additional names for containers on a network.
   ```sh
   docker run -d --name my_container --network my_bridge_network --network-alias my_alias my_image
   ```

2. **Custom DNS Configuration**:
   - Configure custom DNS servers for a container.
   ```sh
   docker run -d --name my_container --dns 8.8.8.8 --dns 8.8.4.4 my_image
   ```

3. **Network Security**:
   - Use Docker network policies to restrict traffic between containers.
   - Implement firewall rules and security groups at the network level.

4. **Service Discovery**:
   - Docker Swarm and Kubernetes offer built-in service discovery for resolving container names to IP addresses.
   - Example: Using Docker Swarm:
     ```sh
     docker service create --name my_service --network my_overlay_network my_service_image
     ```

5. **Load Balancing**:
   - Use built-in load balancing in Docker Swarm or Kubernetes to distribute traffic across multiple containers.
   - Example: Deploying a replicated service in Docker Swarm:
     ```sh
     docker service create --name my_service --network my_overlay_network --replicas 3 my_service_image
     ```

### Best Practices

1. **Use Appropriate Network Drivers**:
   - Choose the network driver that best fits your application's architecture and requirements.

2. **Isolate Sensitive Containers**:
   - Use separate networks for containers that handle sensitive data to isolate them from other containers.

3. **Monitor Network Traffic**:
   - Use network monitoring tools to keep an eye on traffic between containers and detect any anomalies.

4. **Automate Network Management**:
   - Use orchestration tools like Docker Swarm or Kubernetes to manage network configurations automatically.

## Example (Connect a express.js back-end to a MongoDB database)


### Step 1: Create a Docker Network

First, create a Docker network that both the MongoDB and the Express.js containers will use:

```sh
docker network create my_network
```

### Step 2: Run MongoDB Container

Run the MongoDB container and connect it to the network:

```sh
docker run -d --name mongo_container --network my_network -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=example mongo
```

### Step 3: Run Express.js Container

Assume your Express.js application is set up with the necessary MongoDB connection code. We'll build and run the Express.js container, also connecting it to the same network:

1. **Create a Dockerfile for Express.js Application**:
   
   Create a `Dockerfile` in the root directory of your Express.js project:

   ```Dockerfile
   FROM node:21

   # Create app directory
   WORKDIR /usr/src/app

   # Install app dependencies
   COPY package*.json ./
   RUN npm install

   # Bundle app source
   COPY . .

   # Expose the port the app runs on
   EXPOSE 3000

   # Define the command to run the app
   CMD [ "node", "app.js" ]
   ```

2. **Build the Docker Image**:
   
   ```sh
   docker build -t my_express_app .
   ```

3. **Run the Express.js Container**:
   
   ```sh
   docker run -d --name express_container --network my_network -p 3000:3000 my_express_app
   ```

### Step 4: Update Express.js Code to Connect to MongoDB

In your Express.js application, typically in a file like `app.js` or a separate database configuration file, configure the MongoDB connection using the container name (`mongo_container`) as the hostname.

Install MongoDB driver if you haven't already:

```sh
npm install mongoose
```

Here’s an example of how to set up the connection using Mongoose:

```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const port = 3000;

// MongoDB connection
const dbUser = 'root';
const dbPassword = 'example';
const dbName = 'test'; // Change to your database name
const mongoHost = 'mongo_container';
const mongoPort = '27017'; // Default MongoDB port

const mongoURL = `mongodb://${dbUser}:${dbPassword}@${mongoHost}:${mongoPort}/${dbName}?authSource=admin`;

mongoose.connect(mongoURL, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => {
  console.log('Connected to MongoDB');
}).catch(err => {
  console.error('MongoDB connection error:', err);
});

// Define a simple route
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Express app listening at http://localhost:${port}`);
});
```

### Step 5: Verify Connection

1. **Check Running Containers**:
   
   Ensure both containers are running and attached to the same network:

   ```sh
   docker ps
   ```

2. **Check Network**:
   
   Verify both containers are on the `my_network` network:

   ```sh
   docker network inspect my_network
   ```

3. **Access Application**:
   
   Open a browser or use `curl` to check the Express.js application:

   ```sh
   curl http://localhost:3000
   ```

### Explanation

- The `docker network create my_network` command creates a new network named `my_network`.
- Both the MongoDB and Express.js containers are connected to this network using the `--network my_network` option in the `docker run` commands.
- In the Express.js code, the `mongoURL` is constructed using `mongo_container` as the hostname, allowing the Express.js container to resolve the MongoDB container by name within the Docker network.
