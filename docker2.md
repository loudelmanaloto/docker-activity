# Tier 2: Volumes, Environment Variables & Multi-Stage Builds

## Overview
Tier 2 dives deeper into making Docker containers practical for real applications. Students will learn to persist data, configure apps at runtime, and optimize image size—all essential for production-ready Dockerfiles.

---

# Part 1: Volumes — Persisting Data

## The Problem
When a container stops and is removed, all data inside is lost. This is a problem for databases, logs, and any app that needs persistent storage.

```
Container stops → Removed → All data deleted forever ❌
```

## The Solution: Volumes
Volumes are storage locations on the host machine that containers can access. They survive container removal.

```
Container stops → Removed → Volume data persists ✓
```

---

## How Volumes Work

### Concept: Bind Mounts
Map a folder on your computer to a folder inside the container.

```
Host Machine          Container
/Users/you/data  ---→  /app/data
(real folder)          (container sees it)
```

When the app writes to `/app/data`, it actually writes to `/Users/you/data` on your machine.

### Syntax
```bash
docker run -v <host-path>:<container-path> <image>
```

**Example:**
```bash
docker run -v /Users/you/mydata:/app/data my-app:1.0
```

- `<host-path>` — Real folder on your computer (absolute path)
- `<container-path>` — Where it appears inside the container
- `:` — Separator

---

## Use Case 1: Persist Database Data

### Scenario
You're running a database container. Data needs to survive restarts.

```bash
# Create a folder for database files
mkdir -p ~/my-database

# Run a database container (using PostgreSQL example)
docker run \
  -v ~/my-database:/var/lib/postgresql/data \
  --name my-db \
  postgres:15

# Even if you: docker stop my-db && docker rm my-db
# The data in ~/my-database still exists
# You can remount and the data comes back
```

---

## Use Case 2: Hot Reload / Live Development

### Scenario
You're developing an app. Instead of rebuilding the image every time you edit code, mount your source folder.

**Traditional workflow (slow):**
```
Edit code → docker build → docker run → Test
(repeat = slow!)
```

**With volumes (fast):**
```
Edit code → Changes appear immediately in container → Test
(no rebuild needed!)
```

### Example: Node.js App with Hot Reload

**Step 1:** Create an app directory with `app.js`:
```javascript
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
  const message = fs.readFileSync('/app/message.txt', 'utf-8');
  res.end(message);
});

server.listen(3000);
```

**Step 2:** Create `message.txt`:
```
Hello from development!
```

**Step 3:** Create a `Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

**Step 4:** Mount your source code:
```bash
docker build -t dev-app:1.0 .

docker run \
  -v $(pwd):/app \
  -p 3000:3000 \
  --name dev-container \
  dev-app:1.0
```

**Step 5:** Edit `message.txt` on your machine:
```
Hello from development! (Updated!)
```

**Step 6:** Refresh your browser at `http://localhost:3000`

**Result:** The change appears instantly—no container restart needed!

---

## Use Case 3: Shared Files Between Host and Container

### Example: Viewing Container-Generated Logs

```bash
# Create a logs folder on your machine
mkdir -p ~/app-logs

# Run container with logs folder mounted
docker run \
  -v ~/app-logs:/app/logs \
  my-app:1.0

# The app writes logs to /app/logs (inside container)
# You can read them from ~/app-logs on your machine
cat ~/app-logs/app.log
```

---

## Important: Read-Only Volumes

Sometimes you want to share files but prevent the container from modifying them.

```bash
docker run \
  -v ~/config:/app/config:ro \
  my-app:1.0
```

The `:ro` flag means "read-only". The container can read files but not write to them.

---

## Volume Troubleshooting

### Problem: Permission Denied
**Cause:** Container user doesn't have permissions to access host folder.

**Solution:** Make the folder writable:
```bash
chmod 777 ~/my-folder
```

### Problem: Volume Not Updating
**Cause:** Container is still using old code.

**Solution:** 
1. Stop and remove the container: `docker stop <name> && docker rm <name>`
2. Run again with the volume mount

### Problem: "Volume Mounts Must Use Absolute Paths"
**Cause:** You used a relative path.

**Solution:** Use absolute path or `$(pwd)`:
```bash
# Wrong:
docker run -v ./data:/app/data  ❌

# Right:
docker run -v $(pwd)/data:/app/data  ✓
```

---

# Part 2: Environment Variables — Runtime Configuration

## The Problem
Hard-coded values in code aren't flexible. You can't easily change settings without rebuilding the image.

```javascript
const API_URL = "http://localhost:8000";  // Hard-coded ❌
```

## The Solution: Environment Variables
Pass configuration at runtime without changing code.

```javascript
const API_URL = process.env.API_URL || "http://localhost:8000";  // Flexible ✓
```

---

## Three Ways to Pass Environment Variables

### Method 1: Command-Line Flag (`-e`)

Pass individual variables:

```bash
docker run \
  -e API_URL="http://api.example.com" \
  -e DEBUG="true" \
  my-app:1.0
```

The app can access these via `process.env.API_URL` and `process.env.DEBUG`.

### Method 2: Environment File (`--env-file`)

Create a file with multiple variables:

**Step 1:** Create `.env`:
```
API_URL=http://api.example.com
DEBUG=true
DATABASE_URL=postgres://user:pass@db:5432/mydb
LOG_LEVEL=info
```

**Step 2:** Pass it to docker run:
```bash
docker run \
  --env-file .env \
  my-app:1.0
```

### Method 3: Set Default Values in Dockerfile

Hard-code defaults (can be overridden at runtime):

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Set a default value
ENV API_URL="http://localhost:8000"
ENV DEBUG="false"

COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

At runtime, you can override these defaults:
```bash
docker run -e DEBUG="true" my-app:1.0
```

---

## Real-World Example: Configurable Web Server

### The App: `app.js`
```javascript
const http = require('http');

const API_URL = process.env.API_URL || "http://localhost:8000";
const PORT = process.env.PORT || 3000;
const DEBUG = process.env.DEBUG === "true";

const server = http.createServer((req, res) => {
  if (DEBUG) console.log(`Request to: ${req.url}`);
  
  res.end(`
    API URL: ${API_URL}
    Port: ${PORT}
    Debug Mode: ${DEBUG}
  `);
});

server.listen(PORT, () => {
  console.log(`Server listening on port ${PORT}`);
  console.log(`Connected to API: ${API_URL}`);
});
```

### The Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

### Running with Different Configurations

**Development (verbose):**
```bash
docker run \
  -e API_URL="http://localhost:8000" \
  -e DEBUG="true" \
  -p 3000:3000 \
  my-app:1.0
```

**Production (quiet):**
```bash
docker run \
  -e API_URL="http://api.production.com" \
  -e DEBUG="false" \
  -p 3000:3000 \
  my-app:1.0
```

**Same image, different behavior. No rebuild needed!**

---

## Sensitive Data: Secrets vs Environment Variables

### ⚠️ Never Put Secrets in Environment Variables
Database passwords, API keys, and tokens should NOT be passed as plain text environment variables—they're visible in `docker ps` output.

```bash
# DON'T DO THIS:
docker run -e DB_PASSWORD="super-secret-123" my-app  ❌
```

**Anyone can see the password:**
```bash
docker inspect <container>  # Shows all environment variables
```

### ✓ Use Docker Secrets (Advanced)
For production, use Docker Secrets or external secret management:
- Docker Swarm Secrets
- Kubernetes Secrets
- HashiCorp Vault
- AWS Secrets Manager

For now in development, use `.env` files and add them to `.gitignore`:

```bash
# .env (in .gitignore)
DB_PASSWORD=secret123

# Run with:
docker run --env-file .env my-app:1.0
```

---

# Part 3: Multi-Stage Builds — Optimizing Image Size

## The Problem
A single-stage Dockerfile can create bloated images with unnecessary build tools.

```dockerfile
FROM node:18  # Includes npm, build tools, etc.
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]
```

**Problem:** The final image includes:
- npm (not needed at runtime)
- Build tools
- Development dependencies
- Large file size (500MB+)

## The Solution: Multi-Stage Builds

Use multiple `FROM` statements. Build in one stage, copy only essentials to the final stage.

```dockerfile
# Stage 1: Builder (with npm and build tools)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .

# Stage 2: Runtime (lean, production-ready)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

**Result:**
- Stage 1 (builder): 900MB, gets discarded
- Stage 2 (final): 200MB, clean and efficient

**Size reduction: ~78%**

---

## Why Multi-Stage Builds Matter

### Traditional (Single-Stage)
```
Dockerfile: FROM node:18 → RUN npm install → COPY app → CMD
Result: 900MB image with all build tools included
```

### Multi-Stage
```
Stage 1: FROM node:18 → RUN npm install → (discarded)
Stage 2: FROM node:18-alpine → COPY node_modules → (only 200MB)
Result: 200MB image, production-ready
```

---

## Step-by-Step Example: Building a Lean Node.js App

### Initial Single-Stage Dockerfile (Bad)
```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

Image size: ~920MB

### Optimized Multi-Stage Dockerfile (Good)
```dockerfile
# Stage 1: Install dependencies
FROM node:18 AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build final image
FROM node:18-alpine
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

Image size: ~165MB (82% smaller!)

### Key Improvements
- `npm ci` instead of `npm install` (cleaner, faster)
- `--only=production` excludes dev dependencies
- `node:18-alpine` instead of `node:18` (smaller base)
- `COPY --from=dependencies` brings only what's needed

---

## Multi-Stage with Build Steps

### Example: Python App with Compilation

```dockerfile
# Stage 1: Build dependencies
FROM python:3.11 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY app.py .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

**Size reduction:** 1.2GB → 200MB

---

## Multi-Stage with Go (Super Efficient)

Go compiles to a single binary—perfect for multi-stage:

```dockerfile
# Stage 1: Build the binary
FROM golang:1.20 AS builder
WORKDIR /build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

# Stage 2: Run the binary (doesn't need Go installed)
FROM scratch
COPY --from=builder /build/app /app
ENTRYPOINT ["/app"]
```

**Result:** 8MB image (no OS, no runtime—just the binary!)

---

## Activity 1: Using Volumes for Development

### Objective
Set up a live-reloading development environment.

### Steps

**Step 1:** Create a development folder:
```bash
mkdir dev-project && cd dev-project
```

**Step 2:** Create `app.js`:
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.end('Hello! This is version 1.\n');
});

server.listen(3000);
```

**Step 3:** Create `package.json`:
```json
{
  "name": "dev-app",
  "version": "1.0.0",
  "main": "app.js"
}
```

**Step 4:** Create `Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

**Step 5:** Build the image:
```bash
docker build -t dev-app:1.0 .
```

**Step 6:** Run with volume mount (live reload):
```bash
docker run \
  -v $(pwd):/app \
  -p 3000:3000 \
  --name dev \
  dev-app:1.0
```

**Step 7:** In another terminal, edit `app.js`:
```javascript
res.end('Hello! This is version 2 (updated).\n');
```

**Step 8:** Restart the container (in same terminal):
```bash
docker stop dev && docker start dev
```

**Step 9:** Check the change:
```bash
curl http://localhost:3000
# Output: Hello! This is version 2 (updated).
```

### Extension Challenge
Can you use `nodemon` to auto-restart the app when files change (no manual restart)?

---

## Activity 2: Environment Variables in Action

### Objective
Create a configurable app that changes behavior based on environment variables.

### Steps

**Step 1:** Create `config-app.js`:
```javascript
const http = require('http');

const GREETING = process.env.GREETING || "Hello";
const NAME = process.env.NAME || "Friend";
const PORT = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  res.end(`${GREETING}, ${NAME}!\n`);
});

server.listen(PORT, () => {
  console.log(`Server listening on ${PORT}`);
});
```

**Step 2:** Create `Dockerfile`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY config-app.js app.js
EXPOSE 3000
CMD ["node", "app.js"]
```

**Step 3:** Build:
```bash
docker build -t config-app:1.0 .
```

**Step 4:** Run with default values:
```bash
docker run -p 3000:3000 config-app:1.0
curl http://localhost:3000
# Output: Hello, Friend!
```

**Step 5:** Run with custom environment variables:
```bash
docker run \
  -e GREETING="Bonjour" \
  -e NAME="Alice" \
  -p 3001:3000 \
  config-app:1.0

curl http://localhost:3001
# Output: Bonjour, Alice!
```

**Step 6:** Create `.env` file:
```
GREETING=Hola
NAME=Mundo
```

**Step 7:** Run with `.env` file:
```bash
docker run \
  --env-file .env \
  -p 3002:3000 \
  config-app:1.0

curl http://localhost:3002
# Output: Hola, Mundo!
```

### Key Takeaway
Same image, three different behaviors—no rebuild needed!

---

## Activity 3: Multi-Stage Build Optimization

### Objective
Build a bloated image, then optimize it with multi-stage builds.

### Steps

**Step 1:** Create `inefficient.Dockerfile` (single-stage):
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

**Step 2:** Build and check size:
```bash
docker build -f inefficient.Dockerfile -t bloated-app:1.0 .
docker images bloated-app:1.0
```

**Note the SIZE column—likely 900MB+**

**Step 3:** Create `efficient.Dockerfile` (multi-stage):
```dockerfile
# Stage 1: Dependencies
FROM node:18 AS dependencies
WORKDIR /app
COPY package.json .
RUN npm install

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
```

**Step 4:** Build and check size:
```bash
docker build -f efficient.Dockerfile -t lean-app:1.0 .
docker images lean-app:1.0
```

**Compare sizes:**
```bash
docker images | grep -E "bloated-app|lean-app"
```

**Result:** `lean-app` should be ~70-80% smaller!

### Challenge
What if you used `node:18-slim` in stage 1? Would the final image be smaller? Try it!

---

## Cheat Sheet: Tier 2 Commands

### Volumes
```bash
docker run -v <host-path>:<container-path> <image>          # Bind mount
docker run -v <host-path>:<container-path>:ro <image>       # Read-only
docker run -v ~/data:/app/data my-app                        # Example
```

### Environment Variables
```bash
docker run -e VAR_NAME=value <image>                  # Single variable
docker run -e VAR1=val1 -e VAR2=val2 <image>         # Multiple variables
docker run --env-file .env <image>                    # From file
docker inspect <container> | grep -A 10 Env          # View container's env vars
```

### Multi-Stage Dockerfile Structure
```dockerfile
FROM base-image AS stage-name
...build steps...

FROM final-image
COPY --from=stage-name /path /path
...final image...
```

---

## Key Takeaways

### Volumes
- Use for persistent data, databases, and development
- Bind host folders to container paths with `-v`
- Changes on host appear instantly in container
- Perfect for live development

### Environment Variables
- Configure apps at runtime without rebuilding
- Use `-e` for single variables, `--env-file` for multiple
- Set defaults in Dockerfile with `ENV`
- Never hardcode secrets—use external secret management in production

### Multi-Stage Builds
- Build with full toolchains, deploy with minimal images
- Use `COPY --from=stage-name` to move artifacts
- Huge size savings (often 70-80% smaller)
- Faster deployments, better for registries and CI/CD

