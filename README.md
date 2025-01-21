```bash

# Run the MongoDB containers first.

docker run -d \
  --name code-ide-mongodb \
  --network code-ide-network \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=example \
  -e MONGO_INITDB_DATABASE=code-ide \
  -v mongodb_data:/data/db \
  --health-cmd="mongosh --port 27017 --eval 'db.runCommand(\"ping\").ok' --quiet" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=40s \
  mongo:latest \
  --bind_ip_all
```

```bash

# Run the backend container second.

docker build -t full-stack-online-code-ide-backend:latest ./backend && \
docker run -d \
  --name code-ide-backend \
  --network code-ide-network \
  -p 3000:3000 \
  -e "MONGODB_URI=mongodb://root:example@code-ide-mongodb:27017/code-ide?authSource=admin" \
  -e "JWT_SECRET=your_jwt_secret_key_here" \
  -e "NODE_ENV=production" \
  --health-cmd="wget --no-verbose --tries=1 --spider http://localhost:3000/health" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=60s \
  full-stack-online-code-ide-backend:latest
```


```bash

# Run the frontend container last.

docker build -t full-stack-online-code-ide-frontend:latest ./frontend && \
docker run -d \
  --name code-ide-frontend \
  --network code-ide-network \
  -p 5173:5173 \
  -e "VITE_API_URL=http://localhost:3000" \
  -e "NODE_ENV=production" \
  --health-cmd="wget --no-verbose --tries=1 --spider http://localhost:5173" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=40s \
  full-stack-online-code-ide-frontend:latest
  ```


```bash

# Or just Up the compose file.

docker compose up -d

```