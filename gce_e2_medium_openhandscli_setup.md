
# Setting up OpenHands CLI on Google Cloud Engine (GCE) E2 Medium Instance

This document outlines the steps to set up and run the OpenHands CLI on a Google Cloud Engine (GCE) E2 medium instance.

## System Requirements for GCE E2 Medium Instance
A GCE E2 medium instance (2 vCPUs, 4 GB memory) meets the recommended system requirements for running OpenHands (4GB RAM minimum).

## Prerequisites (on GCE E2 Medium Instance)

Before proceeding with OpenHands CLI, ensure the following prerequisites are met on your Linux-based GCE instance:

1.  **Operating System**: The OpenHands documentation states it's tested with Ubuntu 22.04. GCE instances can be provisioned with Ubuntu. I will focus on that, as the current environment already has ubuntu.
2.  **Docker Desktop**: OpenHands relies on Docker for its runtime. You need to install Docker Desktop on your Linux instance.

    *   **Install Docker Desktop on Linux**: Follow the official Docker documentation for installing Docker Desktop on your Linux distribution. For Ubuntu, this typically involves:
        1.  Setting up Docker's apt repository.
        2.  Installing `docker-ce`, `docker-ce-cli`, `containerd.io`, and `docker-buildx-plugin`.
        3.  Starting the Docker service.
        4.  Adding your user to the `docker` group to run Docker commands without `sudo`.

3.  **Git**: You will need Git to clone the OpenHands repository. Install it using:
    ```bash
    sudo apt update
    sudo apt install git
    ```

## Installation Steps

1.  **Connect to your GCE Instance**: Use SSH to connect to your GCE E2 medium instance.

2.  **Install Prerequisites**:
    *   **Docker Desktop**:
        ```bash
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
          $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
          sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        sudo systemctl start docker
        sudo usermod -aG docker $USER
        newgrp docker
        ```
        Verify Docker installation:
        ```bash
        docker run hello-world
        ```
    *   **Git**:
        ```bash
        sudo apt update
        sudo apt install git
        ```

3.  **Clone OpenHands Repository**:
    ```bash
    git clone https://github.com/All-Hands-AI/OpenHands.git
    cd OpenHands
    ```

4.  **Run OpenHands CLI with Docker**:
    OpenHands can be run directly using Docker commands. Navigate to the cloned `OpenHands` directory and execute:

    ```bash
    docker pull docker.all-hands.dev/all-hands-ai/runtime:0.49-nikolaik
    docker run -it --rm \
      -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.49-nikolaik \
      -e LOG_ALL_EVENTS=true \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v ~/.openhands:/.openhands \
      -p 3000:3000 \
      --add-host host.docker.internal:host-gateway \
      --name openhands-app \
      docker.all-hands.dev/all-hands-ai/openhands:0.49
    ```
    (Note: Replace `0.49` with the desired OpenHands version if different.)

5.  **Access OpenHands**:
    OpenHands will be accessible via `http://localhost:3000` on your GCE instance. If you need to access it from your local machine, ensure your GCE instance's firewall rules allow inbound traffic on port 3000.

    On your local machine, you can set up an SSH tunnel:
    ```bash
    gcloud compute ssh YOUR_INSTANCE_NAME --project=YOUR_PROJECT_ID --zone=YOUR_ZONE -- -L 3000:localhost:3000
    ```
    Then, navigate to `http://localhost:3000` in your local browser.

## Post-Installation Setup
After launching OpenHands, remember to configure your LLM provider and API key in the UI settings.
