# CST8915 Lab 4: Introduction to Docker

**Student Name**: Abdullah Al-Hajjar
**Student ID**: 040656012
**Course**: CST8915 Full-stack Cloud-native Development
**Semester**: Fall 2026

---

## Demo Video (Part 6: Docker Compose)

🎥 [Watch Demo Video](https://youtu.be/7gL2B-CWe14)

---

## Reflection Questions

### 1. What are the main differences between a Docker image and a Docker container?

An image serves as a read-only template containing base OS files, dependencies, code, and a startup command built as Dockerfile layers. A container operates as a live instance of that same image. Docker places a writable layer on top, assigns a distinct process, network, and file system view, then launches the command. Multiple containers spawn from a single image, like running three from my-python-app:v3. Removing a container leaves the image untouched. Think of it like a programming class and its objects.

### 2. Explain how Docker's layered architecture improves efficiency.

Each instruction creates a layer, and Docker caches them when upper steps stay the same. Editing app.py and rebuilding made WORKDIR, COPY requirements.txt, and pip install return as CACHED. Only the last COPY app.py layer rebuilt, taking seconds instead of a full reinstall. Layers are also shared. Three containers displayed about 209 kB of distinct data over a 160 MB image, and image pulls only fetch missing layers. That is why dependencies go before code inside a Dockerfile.

### 3. Why does each container get its own writable layer?

This keeps instances isolated while the underlying image remains unaltered. Writing data1.txt, data2.txt, and data3.txt across three containers meant each instance only accessed its own file. Appending text to app.py inside app1 triggered copy-on-write, copying the file into the writable layer so app2 kept the original version. Removing a container deletes its writable layer. Any persistent data, such as the Redis information in Part 6, requires a volume.

### 4. What are the benefits of using Docker Compose over running containers individually?

Compose houses the entire application, including the web service, Redis, network, volume, environment variables, and startup sequence, in a single docker-compose.yml file instead of long docker run commands. Executing docker compose up -d builds and launches everything, while docker compose down handles cleanup. Services locate each other by name, allowing the Flask app to reach redis via hostname. The depends_on setting manages startup order, centralized logs gather in one location, and scaling requires just --scale web=3. Version control and sharing are simple, ensuring identical setups for everyone.

---

## Setup Notes

- **Docker host:** Ubuntu 24.04 VM on Azure in North Central US, the region my Azure for Students subscription allows. Standard_B2s isn't offered to my subscription there, so I used **Standard_B2als_v2** (2 vCPU, 4 GB), the equivalent size. SSH and HTTP were only open to my own IP.
- **Docker install:** Docker Engine from Docker's official APT repository, following the lab. Versions: Docker **29.9.1**, Docker Compose **v5.6.0**. `hello-world` ran fine, and my user was added to the `docker` group so I didn't need `sudo`.
- I worked through Parts 2 to 5 on the same VM (nginx basics, the Flask Dockerfile, layers and caching, writable layers and copy-on-write), and the video covers Part 6 as required.

## Lessons Learned / Challenges

- Newer Docker Compose prints a warning that the `version:` line in `docker-compose.yml` is obsolete. It's ignored, so the file still works, but it can be removed.
- Scaling only works after removing the fixed `"80:5000"` host port, because three containers can't all bind host port 80. With `"5000"` Docker picks a random host port for each copy.
- On recent Docker versions `docker inspect --format '{{.NetworkSettings.IPAddress}}'` no longer works for containers on the default network; the IP is under `.NetworkSettings.Networks` instead.
