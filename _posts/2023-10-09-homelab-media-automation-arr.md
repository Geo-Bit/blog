---
layout: post
title: "Using Docker to Set Up the *Arr Media Automation Toolset in Your Homelab"
date: 2023-10-12
categories: [Docker, Homelab, Media Automation]
---

My journey into automated media management at home, and setting up the \*Arr stack of tools using Docker.

# Introduction

Setting up a media automation toolset like Radarr, Sonarr, and Lidarr (*Arr) in a homelab environment can significantly enhance your media management and streaming experience. Docker is a powerful tool that simplifies the deployment and management of these applications. In this guide, we'll walk you through the process of using Docker to set up the *Arr media automation toolset in your homelab.

## Prerequisites

Before you begin, ensure you have the following prerequisites:

1. A server or computer with Docker and Docker Compose installed.
2. Familiarity with basic Docker concepts and command-line usage.
3. A dedicated directory for your \*Arr Docker configuration files.

# Step 1: Create a Docker Compose Configuration

Start by creating a Docker Compose configuration file, typically named `docker-compose.yml`. This file defines how your \*Arr services will run in Docker containers.

```yaml
version: "3"
services:
  radarr:
    image: linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000 # Set to your user's UID
      - PGID=1000 # Set to your user's GID
      - TZ=Your/Timezone # Set your timezone, e.g., "America/New_York"
    volumes:
      - /path/to/config/radarr:/config
      - /path/to/media:/movies
    ports:
      - 7878:7878
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Your/Timezone
    volumes:
      - /path/to/config/sonarr:/config
      - /path/to/media:/tv
    ports:
      - 8989:8989
    restart: unless-stopped

  lidarr:
    image: linuxserver/lidarr
    container_name: lidarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Your/Timezone
    volumes:
      - /path/to/config/lidarr:/config
      - /path/to/media:/music
    ports:
      - 8686:8686
    restart: unless-stopped
```

Replace the placeholders with your specific configurations:

    - PUID and PGID should be set to your user's UID and GID.
    - TZ should be set to your timezone.
    - /path/to/config should be the directory for your *Arr configuration files.
    - /path/to/media should be the directory for your media libraries.
    - Adjust port numbers if needed.

# Step 2: Start the \*Arr Services

Navigate to the directory containing your docker-compose.yml file and run the following command to start the \*Arr services in Docker containers:

```bash
docker-compose up -d
```

The -d flag runs the containers in detached mode.

# Step 3: Access the \*Arr Web Interfaces

Once the containers are running, you can access the \*Arr web interfaces by opening a web browser and navigating to the following addresses:

    Radarr: http://your-server-ip:7878
    Sonarr: http://your-server-ip:8989
    Lidarr: http://your-server-ip:8686

Replace your-server-ip with the IP address or domain of your server.

# Step 4: Configure \*Arr Applications

Use the web interfaces to configure your \*Arr applications. Add your media libraries, set up download clients, and configure automation settings to your liking.

Using Docker to set up the \*Arr media automation toolset in your homelab environment offers a flexible and manageable solution for managing your media content.
