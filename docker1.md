# Tier 1: Container Lifecycle & Management

## Overview
Students will learn to inspect, monitor, and manage running containers. These skills are essential for understanding what's happening inside Docker and troubleshooting problems.

---

## Part 1: Understanding Container State

### Concept: Container Lifecycle
A container has several states:
- **Created** — Built but not running
- **Running** — Currently executing
- **Paused** — Running but suspended
- **Stopped** — Was running, now stopped (can be restarted)
- **Exited** — Ran to completion and stopped
- **Removed** — Deleted from the system

### The Container Lifecycle Diagram
```
docker run/create
     ↓
   CREATED
     ↓
docker start / docker run
     ↓
   RUNNING ←→ PAUSED (docker pause/unpause)
     ↓
docker stop / app exits
     ↓
   STOPPED / EXITED
     ↓
docker rm
     ↓
   REMOVED (from system)
```

---

## Part 2: The `docker ps` Command

### What It Does
Lists containers currently running on your system. Think of it like "process list" for containers.

### Basic Usage
```bash
docker ps
```

**Output columns:**
```
CONTAINER ID    IMAGE              COMMAND             CREATED         STATUS          PORTS               NAMES
a1b2c3d4e5f6    my-docker-app:1.0  "npm start"         5 seconds ago   Up 4 seconds    0.0.0.0:8080->3000/tcp   my-app-container
```

| Column | Meaning |
|--------|---------|
| `CONTAINER ID` | Unique identifier (first 12 chars shown) |
| `IMAGE` | Which image this container was created from |
| `COMMAND` | The command running inside (from CMD in Dockerfile) |
| `CREATED` | When the container was created |
| `STATUS` | Current state (Up X seconds, Exited X seconds ago, etc.) |
| `PORTS` | Port mappings (host:container) |
| `NAMES` | Friendly name assigned with `--name` |

### Viewing All Containers (Including Stopped)
```bash
docker ps -a
```

Shows both running AND stopped containers. Useful for seeing what you've created.

### Useful Flags
```bash
docker ps -q              # Only show container IDs (quiet mode)
docker ps -l              # Show only the latest container
docker ps --filter status=exited   # Show only stopped containers
docker ps --filter ancestor=my-docker-app:1.0   # Show containers from specific image
```

---

## Part 3: Understanding Logs with `docker logs`

### What It Does
Displays everything a container printed to standard output (stdout) and standard error (stderr). This is your window into what's happening inside.

### Basic Usage
```bash
docker logs <container-name-or-id>
```

Example:
```bash
docker logs my-app-container
```

**Output:**
```
Server running at http://0.0.0.0:3000/
```

### Why Logs Matter
- **Debugging:** See error messages when things break
- **Monitoring:** Watch what the app is doing
- **Troubleshooting:** Understand why a container crashed

### Useful Flags
```bash
docker logs -f <container>           # Follow logs (like tail -f), shows new output in real-time
docker logs --tail 10 <container>    # Show only last 10 lines
docker logs --since 5m <container>   # Show logs from the last 5 minutes
docker logs --timestamps <container> # Add timestamps to each line
```

### Real-World Example: Debugging a Failing Container

Scenario: Your container exited immediately after starting.

```bash
# Step 1: Check what happened
docker ps -a

# Output shows status: Exited (1) 2 seconds ago

# Step 2: Read the logs to see the error
docker logs <container-id>

# Output: Error: Cannot find module 'express'
# Now you know: Forgot to npm install or dependency missing from package.json
```

---

## Part 4: Stopping Containers with `docker stop`

### What It Does
Gracefully shuts down a running container. Sends a SIGTERM signal, giving the app time to clean up.

### Basic Usage
```bash
docker stop <container-name-or-id>
```

Example:
```bash
docker stop my-app-container
```

**Output:**
```
my-app-container
```

The container is now stopped but still exists on your system (can be restarted).

### Restarting a Stopped Container
```bash
docker start <container-name-or-id>

# Check it's running again
docker ps
```

### Difference: `docker stop` vs `docker kill`
| Command | Signal | Behavior |
|---------|--------|----------|
| `docker stop` | SIGTERM | Graceful shutdown (app can clean up) |
| `docker kill` | SIGKILL | Immediate termination (no cleanup) |

**Best practice:** Use `docker stop` for normal shutdown. Use `docker kill` only if `docker stop` hangs.

### Stop Multiple Containers
```bash
# Stop all running containers
docker stop $(docker ps -q)
```

---

## Part 5: Removing Containers with `docker rm`

### What It Does
Permanently deletes a container from your system. The container must be stopped first.

### Basic Usage
```bash
# Step 1: Stop the container
docker stop my-app-container

# Step 2: Remove it
docker rm my-app-container
```

### Remove a Running Container (Force)
```bash
docker rm -f <container-name>  # Force remove without stopping
```

**Warning:** This is destructive. Use with care.

### Remove Multiple Containers
```bash
# Remove all stopped containers
docker rm $(docker ps -aq --filter status=exited)

# Remove all containers (stopped and running)
docker rm -f $(docker ps -aq)
```

---

## Part 6: Managing Images with `docker images` and `docker rmi`

### List Images: `docker images`

Shows all images on your system.

```bash
docker images
```

**Output:**
```
REPOSITORY         TAG       IMAGE ID       CREATED        SIZE
my-docker-app      1.0       a1b2c3d4e5f6   2 minutes ago  180MB
node               18-alpine abc123def456   3 weeks ago    165MB
```

| Column | Meaning |
|--------|---------|
| `REPOSITORY` | Image name (e.g., my-docker-app) |
| `TAG` | Version label (e.g., 1.0, latest) |
| `IMAGE ID` | Unique identifier |
| `CREATED` | When the image was built |
| `SIZE` | How much disk space it uses |

### Useful Flags
```bash
docker images -q                    # Show only image IDs
docker images --filter dangling=true # Show unused/orphaned images
docker images my-docker-app         # Show images from specific repository
```

### Remove Images: `docker rmi`

Permanently deletes an image. Containers using the image must be removed first.

```bash
docker rmi <image-name>:<tag>
```

Example:
```bash
docker rmi my-docker-app:1.0
```

### Remove Multiple Images
```bash
# Remove all unused images
docker image prune

# Remove all images (dangerous!)
docker rmi -f $(docker images -q)
```

---

## Part 7: Cleanup & Disk Space with `docker system df`

### Check Disk Usage
```bash
docker system df
```

**Output:**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         2         850MB     650MB
Containers      8         2         120MB     120MB
Local Volumes   0         0         0B        0B
```

### Clean Up Everything Unused
```bash
docker system prune
```

**Removes:**
- Stopped containers
- Unused images
- Unused networks
- Dangling volumes

**Warning:** This is destructive but safe (only removes unused items).

---

## Activity 1: Container Lifecycle Exploration

### Objective
Students practice all Tier 1 commands in a real scenario.

### Tasks

**Setup:**
```bash
# Use the my-docker-app:1.0 image from the previous activity
# If you don't have it, rebuild:
docker build -t my-docker-app:1.0 .
```

**Task 1: Run Multiple Containers**
```bash
# Run container 1
docker run -p 8080:3000 --name app-1 my-docker-app:1.0

# In another terminal, run container 2
docker run -p 8081:3000 --name app-2 my-docker-app:1.0

# In a third terminal, list all running containers
docker ps
```

**Expected:** You should see both `app-1` and `app-2` in the output.

**Task 2: Check Logs**
```bash
docker logs app-1
docker logs app-2
```

**Expected:** Both should show "Server running at http://0.0.0.0:3000/".

**Task 3: Stop One Container**
```bash
docker stop app-1

# Check status
docker ps        # app-1 should not appear
docker ps -a     # app-1 should appear with status "Exited"
```

**Task 4: Restart the Stopped Container**
```bash
docker start app-1

# Verify it's running
docker ps
docker logs app-1
```

**Task 5: Remove All Containers**
```bash
docker stop $(docker ps -q)    # Stop all running
docker rm $(docker ps -aq)     # Remove all
docker ps -a                   # Should be empty
```

**Task 6: Check Image Size**
```bash
docker images my-docker-app:1.0
```

**Question:** How much disk space is the image using?

---

## Activity 2: Debugging a Broken Container

### Objective
Use logs to diagnose why a container failed.

### Scenario
You'll intentionally create a broken Dockerfile to practice debugging.

**Step 1:** Create a file named `broken-app.js`:
```javascript
const http = require('http');

// Intentional error: missing variable
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.end(missingVariable);  // This will crash!
});

server.listen(3000);
```

**Step 2:** Create `Dockerfile.broken`:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY broken-app.js app.js
EXPOSE 3000
CMD ["npm", "start"]
```

**Step 3:** Build the image
```bash
docker build -f Dockerfile.broken -t broken-app:1.0 .
```

**Step 4:** Try to run it
```bash
docker run --name broken-container broken-app:1.0
```

**Step 5:** Check why it failed
```bash
docker ps -a                      # See it exited
docker logs broken-container      # See the error message

# Output will show: ReferenceError: missingVariable is not defined
```

**Task:** Fix the error in `broken-app.js` and rebuild.

---

## Cheat Sheet: Tier 1 Commands

```bash
# View containers
docker ps                          # Running containers only
docker ps -a                       # All containers
docker ps -q                       # Only IDs

# View logs
docker logs <container>            # Show all logs
docker logs -f <container>         # Follow logs (live)
docker logs --tail 10 <container>  # Last 10 lines

# Manage containers
docker stop <container>            # Graceful shutdown
docker start <container>           # Restart a stopped container
docker rm <container>              # Delete container

# Manage images
docker images                      # List all images
docker rmi <image>                 # Delete image

# Cleanup
docker system df                   # Show disk usage
docker system prune                # Remove unused items
```

---

## Key Takeaways

1. **`docker ps`** is your status checker—always use it first when troubleshooting
2. **`docker logs`** is your detective tool—read it to understand errors
3. **Container lifecycle** goes: create → run → stop → remove
4. **Cleanup regularly** to avoid disk space issues
5. **Containers are temporary**—they can be created and destroyed easily, so experiment!

