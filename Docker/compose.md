# Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. With Docker Compose, you can use a YAML file to configure your application’s services, networks, and volumes, making it easy to manage and scale your application.

### Docker Compose Overview

- **Configuration**: Use a `docker-compose.yml` file to configure your application's services.
- **Single Command**: Start your entire application with a single `docker-compose up` command.
- **Multi-Container Management**: Manage multiple containers, networks, and volumes in a single file.

### Example Docker Compose Setup

Here's an example `docker-compose.yml` file for a Next.js front-end, an Express.js back-end, and a MongoDB database.

1. **Project Structure**:

   ```
   my_project/
   ├── backend/
   │   ├── Dockerfile
   │   ├── package.json
   │   └── app.js
   ├── frontend/
   │   ├── Dockerfile
   │   ├── package.json
   │   └── pages/
   │       └── index.js
   └── docker-compose.yml
   ```

2. **Dockerfile for Next.js Front-End** (`frontend/Dockerfile`):

   ```Dockerfile
   # Use the official Node.js image
   FROM node:21

   # Set the working directory
   WORKDIR /app

   # Copy package.json and package-lock.json
   COPY package*.json ./

   # Install dependencies
   RUN npm install

   # Copy the rest of the application code
   COPY . .

   # Build the Next.js application
   RUN npm run build

   # Expose the port the app runs on
   EXPOSE 3000

   # Start the Next.js application
   CMD ["npm", "start"]
   ```

3. **Dockerfile for Express.js Back-End** (`backend/Dockerfile`):

   ```Dockerfile
   # Use the official Node.js image
   FROM node:14

   # Set the working directory
   WORKDIR /app

   # Copy package.json and package-lock.json
   COPY package*.json ./

   # Install dependencies
   RUN npm install

   # Copy the rest of the application code
   COPY . .

   # Expose the port the app runs on
   EXPOSE 5000

   # Start the Express.js application
   CMD ["node", "app.js"]
   ```

4. **Docker Compose File** (`docker-compose.yml`):

   ```yaml
   version: '3.8'

   services:
     frontend:
       build:
         context: ./frontend
       ports:
         - "3000:3000"
       depends_on:
         - backend
       environment:
         - NEXT_PUBLIC_API_URL=http://backend:5000

     backend:
       build:
         context: ./backend
       ports:
         - "5000:5000"
       depends_on:
         - mongodb
       environment:
         - DB_HOST=mongodb
         - DB_PORT=27017
         - DB_USER=root
         - DB_PASSWORD=example
         - DB_NAME=mydatabase

     mongodb:
       image: mongo
       ports:
         - "27017:27017"
       environment:
         - MONGO_INITDB_ROOT_USERNAME=root
         - MONGO_INITDB_ROOT_PASSWORD=example
       volumes:
         - mongo-data:/data/db

   volumes:
     mongo-data:
   ```

5. **Express.js Code (`backend/app.js`)**:

   ```javascript
   const express = require('express');
   const mongoose = require('mongoose');

   const app = express();
   const port = 5000;

   // MongoDB connection
   const dbUser = process.env.DB_USER;
   const dbPassword = process.env.DB_PASSWORD;
   const dbName = process.env.DB_NAME;
   const dbHost = process.env.DB_HOST;
   const dbPort = process.env.DB_PORT;

   const mongoURL = `mongodb://${dbUser}:${dbPassword}@${dbHost}:${dbPort}/${dbName}?authSource=admin`;

   mongoose.connect(mongoURL, {
     useNewUrlParser: true,
     useUnifiedTopology: true,
   }).then(() => {
     console.log('Connected to MongoDB');
   }).catch(err => {
     console.error('MongoDB connection error:', err);
   });

   app.get('/api', (req, res) => {
     res.send('Hello from Express!');
   });

   app.listen(port, () => {
     console.log(`Express app listening at http://localhost:${port}`);
   });
   ```

6. **Next.js Code (`frontend/pages/index.js`)**:

   ```javascript
   import { useEffect, useState } from 'react';

   export default function Home() {
     const [message, setMessage] = useState('');

     useEffect(() => {
       fetch(process.env.NEXT_PUBLIC_API_URL + '/api')
         .then(response => response.json())
         .then(data => setMessage(data.message));
     }, []);

     return (
       <div>
         <h1>{message}</h1>
       </div>
     );
   }
   ```

### Running the Application

1. **Build and Start the Application**:
   From the root directory (`my_project`), run:

   ```sh
   docker-compose up --build
   ```

2. **Access the Services**:
   - Next.js front-end: `http://localhost:3000`
   - Express.js back-end: `http://localhost:5000/api`
   - MongoDB: `mongodb://localhost:27017`

### Explanation

- **Services**:
  - `frontend`: Builds the Next.js front-end application and exposes it on port 3000.
  - `backend`: Builds the Express.js back-end application and exposes it on port 5000. It waits for the MongoDB service to be ready.
  - `mongodb`: Runs a MongoDB container using the official MongoDB image and initializes it with a root user.

- **Volumes**:
  - `mongo-data`: A named volume to persist MongoDB data.

- **Environment Variables**:
  - Passes necessary environment variables to the containers for configuration.