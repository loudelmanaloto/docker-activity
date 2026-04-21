# Docker Hands-On Activity: Build and Run Your First Container

## Objective
Apply `docker build` and `docker run` commands to create a containerized application from scratch.

---

## Scenario
You've been asked to containerize a simple Node.js web server so it can run consistently across different machines. Your task is to create a Dockerfile, build an image, and run it as a container.

---

## Part 1: Create the Application

First, create a simple Node.js web server:

**Step 1:** Create a file named `app.js`:
```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello from Docker! 🐳\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

**Step 2:** Create a file named `package.json`:
```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "description": "A simple Docker demo app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
```

---

## Part 2: Write the Dockerfile

**Your Task:** Complete the Dockerfile below. Fill in the blanks (marked with `???`).

Create a file named `Dockerfile`:
```dockerfile
# Use the official Node.js image as the base
FROM ???

# Set the working directory inside the container
WORKDIR ???

# Copy package.json to the container
COPY ??? ???

# Install dependencies
RUN npm install

# Copy the application code to the container
COPY ??? ???

# Expose port 3000
EXPOSE ???

# Define the command to run the app
CMD ["npm", "start"]
```

**Hints:**
- Node.js base image on Docker Hub: `node:18-alpine` (lightweight option)
- Working directory should be `/app`
- You need to copy both `package.json` and `app.js`
- The app listens on port 3000

---

## Part 3: Build the Image

**Your Task:** Build a Docker image named `my-docker-app` with version `1.0`.

Run this command in the directory containing your Dockerfile:
```bash
docker build -t my-docker-app:1.0 .
```

**Expected output:** You should see a series of steps being executed, ending with "Successfully built...".

---

## Part 4: Run the Container

**Your Task:** Run the container with the following requirements:
- Map port 3000 from the container to port 8080 on your machine
- Give the container a friendly name: `my-app-container`
- Keep it running in the foreground so you can see logs

Use this command structure:
```bash
docker run -p 8080:3000 --name my-app-container my-docker-app:1.0
```

**Expected output:** You should see `Server running at http://0.0.0.0:3000/`.

---

## Part 5: Test Your Container

**In another terminal**, test that your app is working:
```bash
curl http://localhost:8080
```

**Expected response:**
```
Hello from Docker! 🐳
```

---

## Verification Checklist

- [ ] I created `app.js`, `package.json`, and `Dockerfile`
- [ ] I filled in all the blanks in the Dockerfile correctly
- [ ] `docker build` completed successfully
- [ ] `docker run` started the container without errors
- [ ] `curl http://localhost:8080` returned the expected response

---

## Challenges (Try These Next!)

### Challenge 1: Run Multiple Instances
Can you run **two containers** from the same image, mapping them to different ports (e.g., 8080 and 8081)?

**Hint:** Use different `--name` values and different `-p` port mappings.

### Challenge 2: Clean Up
Remove the container and image you created. What commands did you use?

**Hint:** `docker stop`, `docker rm`, and `docker rmi`.

### Challenge 3: Add Environment Variables
Modify `app.js` to read a message from an environment variable (e.g., `process.env.MESSAGE`). Then use `docker run -e MESSAGE="Custom Message"` to pass it in.

### Challenge 4: Inspect the Image
Run `docker image inspect my-docker-app:1.0` and answer:
- What operating system is the image based on?
- How many layers does it have?
- What is the total size?

---

## Troubleshooting

**Problem:** `docker build` fails with "file not found"
- **Solution:** Make sure you're running the command in the directory containing `Dockerfile`, `app.js`, and `package.json`.

**Problem:** `docker run` says port 8080 is already in use
- **Solution:** Either stop the previous container or use a different port like `-p 8081:3000`.

**Problem:** `curl` returns "Connection refused"
- **Solution:** Make sure the container is still running (`docker ps`). If not, restart it or check the logs with `docker logs my-app-container`.

---

## Submission

-Do a video recording of your step-by-step proces.
-Make sure your face is also captured when doing the recording
-Upload said video recording on google drive and make sure that it is shared to me or anyone with link.
-Upload the pdf file containing the link of your recorded video.

---
