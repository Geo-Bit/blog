---
layout: post
title: "How to Install Overseerr in a Docker Container"
date: 2023-10-12
categories: [Docker, Overseerr]
---

# Overview

This guide will walk you through the process of setting up Overseerr in a Docker container. Overseerr is a web-based application that allows you to manage your Plex Media Server and monitor Plex activity.

## Prerequisites

Before you begin, make sure you have the following:

- A server or computer with Docker and Docker Compose installed.
- Basic knowledge of Docker and command-line usage.

# Step 1: Create a Docker Compose File

First, create a directory to store your Overseerr Docker Compose configuration file. You can create this file using a text editor of your choice, such as `docker-compose.yml`.

```yaml
version: "3"
services:
  overseerr:
    image: sct/overseerr
    container_name: overseerr
    environment:
      - PUID=1000 # Set to your user's UID
      - PGID=1000 # Set to your user's GID
      - TZ=Your/Timezone # Set your timezone, e.g., "America/New_York"
    volumes:
      - /path/to/config:/config
      - /path/to/downloads:/downloads
    ports:
      - 5055:5055 # Change the host port if needed
    restart: unless-stopped
```

Replace the placeholders with your specific configuration:

    PUID and PGID should be set to your user's UID and GID.
    TZ should be set to your timezone.
    /path/to/config should be the directory for Overseerr's configuration files.
    /path/to/downloads should be the directory for downloaded content.
    You can change the host port (e.g., 5055) to any available port on your server.

# Step 2: Create Overseerr Configuration File

Create a directory named config (or another name you specified in the Compose file) and inside that directory, create a file named config.json. You can use the sample configuration provided in the Overseerr documentation and customize it as needed:

```bash
docker-compose up -d
```

The -d flag runs the container in detached mode, which means it runs in the background.

Step 4: Access Overseerr

Once the container is running, you can access Overseerr by opening a web browser and navigating to:

```arduino
http://your-server-ip:5055
```

Replace your-server-ip with the IP address or domain of your server, and 5055 with the port you specified in the Compose file.

# Step 5: Configure Overseerr

Follow the setup wizard in your web browser to configure Overseerr. You'll need to provide Plex authentication details and customize your preferences.

That's it! You now have Overseerr running in a Docker container, allowing you to manage your Plex Media Server more effectively.
