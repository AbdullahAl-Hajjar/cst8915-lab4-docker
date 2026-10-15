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

An image is the read-only template: the base OS files, the dependencies, my code and the start command, all baked in as layers from the Dockerfile. A container is a running instance of that image. Docker adds a writable layer on top, gives it its own process, network and file system view, and starts the command. You can run lots of containers from one image (I ran three from `my-python-app:v3`), and deleting a container doesn't touch the image. It's a bit like a class and its objects.

### 2. Explain how Docker's layered architecture improves efficiency.

Every Dockerfile instruction becomes a layer, and Docker caches layers and reuses them whenever nothing above them changed. When I edited `app.py` and rebuilt, the `WORKDIR`, `COPY requirements.txt` and `pip install` steps all came back as `CACHED`, and only the last `COPY app.py` layer was rebuilt, so the build took seconds instead of reinstalling everything. Layers are also shared: my three containers each showed only about 209 kB of their own data on top of the same 160 MB image, and pulling an image only downloads the layers you don't already have. That's also why you put dependencies before your code in a Dockerfile.

### 3. Why does each container get its own writable layer?

So containers started from the same image stay isolated and the image itself never changes. When I wrote `data1.txt`, `data2.txt` and `data3.txt` into three containers, each one only saw its own file. When I appended to `app.py` in `app1`, copy-on-write copied the file into app1's writable layer first, so `app2` still had the original. The trade-off is that the writable layer disappears when the container is removed, so anything that needs to survive, like the Redis data in Part 6, should go in a volume.

### 4. What are the benefits of using Docker Compose over running containers individually?

With Compose the whole app (the web service, Redis, the network, the volume, environment variables and start order) lives in one `docker-compose.yml` file instead of a pile of long `docker run` commands. One `docker compose up -d` builds and starts everything, and `docker compose down` cleans it all up. Services can find each other by name (the Flask app connects to `redis` as a hostname), `depends_on` handles the start order, logs from every service show up in one place, and scaling is a single flag (`--scale web=3`). It's also easy to version-control and share, so everyone runs the same setup.

---

## Setup Notes

- **Docker host:** Ubuntu 24.04 VM on Azure in North Central US, the region my Azure for Students subscription allows. Standard_B2s isn't offered to my subscription there, so I used **Standard_B2als_v2** (2 vCPU, 4 GB), the equivalent size. SSH and HTTP were only open to my own IP.
- **Docker install:** Docker Engine from Docker's official APT repository, following the lab. Versions: Docker **29.9.1**, Docker Compose **v5.6.0**. `hello-world` ran fine, and my user was added to the `docker` group so I didn't need `sudo`.
- I worked through Parts 2 to 5 on the same VM (nginx basics, the Flask Dockerfile, layers and caching, writable layers and copy-on-write), and the video covers Part 6 as required.

## Lessons Learned / Challenges

- Newer Docker Compose prints a warning that the `version:` line in `docker-compose.yml` is obsolete. It's ignored, so the file still works, but it can be removed.
- Scaling only works after removing the fixed `"80:5000"` host port, because three containers can't all bind host port 80. With `"5000"` Docker picks a random host port for each copy.
- On recent Docker versions `docker inspect --format '{{.NetworkSettings.IPAddress}}'` no longer works for containers on the default network; the IP is under `.NetworkSettings.Networks` instead.
